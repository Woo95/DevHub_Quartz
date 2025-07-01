# **RealTimeMovement**
---
## **개요**
이 프로젝트는 수업 과제의 일환으로, 제공된 **네트워크 프레임워크를 확장하여 클라이언트-서버 간 움직임 동기화**, **입력 처리**, 그리고 **서버와 클라이언트 양쪽의 명령 송수신 시스템**을 직접 구현했습니다. 실시간 멀티플레이 상호작용에 적합한 **반응성 높고 모듈화된 설계**를 구축하는데 중점을 두었습니다.

![[RealTimeMovement (Media)#^5a7686]]

[**View Repository**](https://github.com/Woo95/RealTimeMovement)

---
## **클라이언트-서버 명령 처리 시스템 설계도**
![[Client-Server Command Processing System Diagram_Kor.png]]
- **흰색 테두리**: 처리 흐름
- **노란색 테두리**: 네트워크 관련 클래스
---
## **🖧 서버 핵심 구성 요소**

### `1. GameLogic 클래스`
`NetworkServerProcessing`에서 호출되는 **핵심 편의 함수를 제공하며 서버의 플레이어 상태를 관리**합니다. 메시지를 직접 처리하지 않고, 연결된 플레이어 목록을 유지하며, 새로 접속한 클라이언트에 고유 `seed`를 할당하고, 연결, 연결 해제, 이동 같은 런타임 이벤트 갱신을 위한 데이터 조회를 지원합니다.

플레이어 등록, 제거, 조회를 위한 **서버 측 함수를 제공**합니다.
![[RealTimeMovement (Code)#^201a55]]

- **핵심 함수들**
	- [[RealTimeMovement (Code)#^fc9c2b|PlayerData Add(int clientConnectionID)]]:
		- 새 클라이언트가 연결될 때 호출됩니다. 고유한 `seed`를 할당하고 접속 세션 목록에 플레이어를 등록합니다. 초기화 과정에서 `NetworkServerProcessing`에 의해 호출됩니다.
	- [[RealTimeMovement (Code)#^e4f3d1|PlayerData Remove(int clientConnectionID)]]:
		- 접속한 클라이언트의 연결 해제 시 호출됩니다. 접속 세션 목록에서 플레이어를 찾아 제거하며, `NetworkServerProcessing`이 게임 상태를 갱신할 때 사용됩니다.
	- [[RealTimeMovement (Code)#^52ae28|PlayerData Search(int playerSeed)]]:
		- 고유한 `seed`를 기준으로 플레이어를 찾아 반환합니다. 서버가 게임 플레이 중, 이동이나 입력을 갱신 및 적용할 때 사용됩니다.

---
### `2. NetworkServerProcessing 클래스`
서버 측의 핵심 **명령어 처리기** 역할을 합니다. 이 정적 클래스는 **연결된 클라이언트로부터 메시지를 받아 처리**하고, **프로토콜 식별자를 분석하여 적절한 로직을 실행**합니다. 연결, 연결 해제, 이동 같은 상태를 갱신하도록 `GameLogic`과 연계합니다.

프로토콜 메시지를 분석하고 경로를 지정하며 전달하는 **중앙 관리 함수**를 제공합니다.
![[RealTimeMovement (Code)#^d082db]]

- **핵심 함수** 
	![[RealTimeMovement (Code)#^95f0c7]]
**클라이언트로부터 수신된 메시지를 처리**하고, 식별자를 분석해 실시간 서버 측 명령을 실행합니다. 세션 및 이동을 갱신 및 적용하기 위해 `GameLogic`과 연계합니다.

---
## **🎮 클라이언트 핵심 구성 요소**
### `1. GameLogic 클래스`
`NetworkClientProcessing`에서 호출되는 핵심 게임플레이 처리 함수를 제공하여 **클라이언트에서 플레이어 객체 상태를 관리**합니다. 메시지를 직접 분석하기보다는, 서버 지시에 따라 생성된 플레이어를 관리하고 위치를 갱신하며 연결 해제 시 제거하는 역할을 합니다.

플레이어의 생성, 동기화, 제거를 위한 **클라이언트 측 함수를 제공**합니다.
![[RealTimeMovement (Code)#^ae6818]]

- **핵심 함수들**
	- [[RealTimeMovement (Code)#^d888d2|void SpawnMySelf(int mySeed, Vector3 position)]]:
		- 지정된 위치에 로컬 플레이어를 생성하고 고유한 `seed`를 할당합니다.
	- [[RealTimeMovement (Code)#^1f211d|void SpawnOthers(int otherSeed, Vector3 position)]]:
		- 다른 클라이언트를 위한 원격 플레이어를 생성하고 지정된 `seed`를 할당합니다.
	- [[RealTimeMovement (Code)#^3cb25e|void MovePlayer(int movedPlayerSeed, Vector3 targetPos)]]:
		- 서버에서 동기화된 좌표로 원격 플레이어의 위치를 갱신합니다.
		- **Type A**: 입력 상태와 상관없이 **정해진 고정 주기**로 이동 정보를 전송합니다.
	- [[RealTimeMovement (Code)#^a84f81|void MovePlayer(int movedPlayerSeed, Vector3 targetPos, Vector2 inputKeys)]]:
		- 원격 플레이어의 위치 및 방향 입력을 모두 갱신합니다.
		- **Type B**: 입력 변화가 있을 때만 상태를 갱신해 **네트워크 트래픽을 감소**시킵니다.
	- [[RealTimeMovement (Code)#^068ad0|void OtherPlayerLeft(int leftPlayerSeed)]]:
	    - `seed` 식별자를 이용해 연결 해제된 플레이어의 오브젝트를 씬에서 제거합니다.
---
### `2. NetworkClientProcessing Class`
클라이언트 측의 **핵심 명령 처리기 역할**을 합니다. 이 정적 클래스는 서버로부터 메시지를 받아 **프로토콜 식별자를 분석하여 적절한 게임 로직을 실행**합니다. 서버 기반 상태 변경에 맞춰 플레이어를 생성, 동기화, 제거하기 위해 `GameLogic`과 연계하여 동작합니다.

프로토콜 메시지를 분석하고 경로를 지정하며 전달하는 **중앙 관리 함수**를 제공합니다.
![[RealTimeMovement (Code)#^ac80a7]]
- **핵심 함수**
  ![[RealTimeMovement (Code)#^d87cc9]]
**서버로부터 수신된 메시지를 처리**하고, 식별자를 분석해 클라이언트 측에 적용합니다. 생성, 이동, 플레이어 제거를 위해 `GameLogic`과 연계합니다.
> ⚠️ 참고: 로직은 생략되어 있습니다. 전체 구현은 [**repository**](https://github.com/Woo95/RealTimeMovement)에서 확인할 수 있습니다.

---
## **맺는 말**
> 이 프로젝트는 **모듈화된 프로토콜 기반**의 **실시간 멀티플레이어 동기화 방식**을 보여줍니다. **서버-클라이언트 간 로직을 일치**시키고 **명확한 메시지 처리 방식**을 통해, **확장 가능하고 반응성이 뛰어난 플레이어 이동 시스템의 탄탄한 기반을 제공**합니다. 또한 **클라이언트-서버 구성**과 **실시간 게임 설계의 기본 개념**에 대한 제 이해를 반영했습니다.