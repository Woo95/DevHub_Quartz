# **타워 디펜더: 끝없는 침공**
---
## **개요**
이 프로젝트는 2023년 수업 과제의 일환으로 한 달간 개발된 **모바일 타워 디펜스 게임**입니다. 디펜스 게임 장르의 **핵심 플레이 요소**를 포함하고 있으며, 디펜스 장르의 핵심 메커니즘을 체계적으로 설계한 프로젝트입니다.

> ⚠️ 참고: 별도의 게임 기획서나 제공된 에셋 없이, 기획부터 시스템 구현, 외부 에셋 선별까지 모든 개발 과정을 스스로 진행했습니다.

[**View Repository**](https://github.com/Woo95/Unity_Mobile_Game_Woo)<br/>[**Watch Demo**](https://youtu.be/sOtYVJ6maYU)

---
## **게임 소개**
유저는 자원을 모으고, 타워를 설치하며, 자율 행동 병사를 배치하며 10번의 웨이브로부터 중앙 타워를 방어하고, 몬스터의 등장 패턴과 자원의 등장 타이밍이 달라지는 만큼 전략적으로 대응하는 것이 핵심입니다.
![[Tower Defender - Waves of War (Media)#^735f14]]

---
## **핵심 기능**
> ⚠️ 참고: 아래의 코드 예제들은 **간소화된 버전**입니다. 전체 구현은 [**repository**](https://github.com/Woo95/Unity_Mobile_Game_Woo)에서 확인 가능합니다.
### `1. 자원 및 몬스터 랜덤 생성 시스템`
자원과 몬스터는 서로 다른 랜덤 생성 시스템을 사용하여 독립적인 게임플레이 역할을 수행합니다.
#### `랜덤한 자원 생성기`
- **자원**은 **게임 시작 시 무작위로 배치**됩니다. 일부 자원은 **여러 번 터치해야 수집**할 수 있으며, **제거되기 전에 파편을 생성**하기도 합니다.
![[Tower Defender - Waves of War (Media)#^7b62fe]]
##### `FieldManager 클래스`
![[Tower Defender - Waves of War (Code)#^895f15]]
주어진 수 만큼의 자원 오브젝트를 **필드 범위 내 무작위 위치**에 생성합니다. 각 오브젝트는 **생성 영역이 겹치지 않을 경우에만 배치**되며, 콜라이더 크기를 기준으로 정확한 간격을 계산합니다. 오브젝트 1개당 최대 500번까지 생성을 시도하며, **무한 루프 없이 안전한 배치**를 보장합니다.

---
#### `랜덤한 몬스터 생성기`
- **몬스터는 각 웨이브마다 일정한 간격으로 게임 플레이 도중 동적으로 생성**됩니다. 맵의 랜덤한 가장자리에서 등장하며, 게임이 진행될수록 생성 속도가 점점 빨라집니다.
![[Tower Defender - Waves of War (Media)#^698c01]]
##### `EnemyManager 클래스`
![[Tower Defender - Waves of War (Code)#^d238ba]]
[[Tower Defender - Waves of War (Code)#^d238ba|SpawnEnemy()]]는 매 `m_NextSpawnTime`마다 호출되어 **맵의 랜덤한 가장자리**에 무작위 적 프리팹을 생성합니다. 생성 위치는 미리 정의된 코너 지점 사이에서 보간되며, 이를 통해 적들이 모든 방향에서 진입합니다. 생성된 적은 초기화된 후 `enemySpawnedList`에 추가됩니다.

---
### `2. 병사 및 몬스터를 위한 간단한 AI 패턴 (FSM 구조)`
모든 병사와 몬스터는 이동, 타겟 지정, 공격 행동을 실시간으로 제어하기 위해 간단한 **Finite State Machine(FSM)** 구조를 사용합니다. (유사한 FSM 구현은 [[Portfolios_Kor/02_Game Projects/2D Platformer Game|2D Platformer Game]]에서 확인)
![[Tower Defender - Waves of War (Media)#^fe06c5]]
`OnDrawGizmos()`를 통해 그려진 **노란색**과 **빨간색** 원은 각각 `searchRadius`와 `attackRadius`를 나타내며, 병사와 몬스터의 탐지 및 공격 범위를 시각적으로 보여줍니다.
- **병사들**
    - **상태**:
        - `IDLE → CHASE → ATTACK`
        - 적과의 거리에 따라 상태가 전환됩니다.
    - **타겟 지정**:
        - [[Tower Defender - Waves of War (Code)#^e563a5|SearchEnemy()]]는 `Physics2D.OverlapCircleAll()`을 사용해 주변을 탐색하고 가장 가까운 `몬스터`를 찾습니다.
    - **전투 로직**:
        - 타겟이 `searchRadius`내에 있으면 추격하고, 공격 사거리 내에 도달하면 정지 후 공격합니다. 유효한 타겟이 없다면 `IDLE` 상태로 돌아옵니다.
- **몬스터들**
    - **State**:
        - `MOVE → ATTACK`
        - 지속적으로 대상을 탐색하며, 사거리 내에 들어오면 공격을 시작합니다.
    - **다중 타겟 우선순위**:
        - 다음 순서로 우선순위를 두고 공격합니다: 
	        - `Guardian → GuardianTower → CentralTower`
    - **대상 없음 시 동작**:
        - `searchRadius`내에 `병사`가 없을 경우, 다시 최종 목표인 중앙 타워를 향해 계속 이동합니다.
---
### `3. 모바일 친화적 카메라 조작`
**드래그 및 핀치 동작**을 통해 카메라 이동과 줌을 지원하며, 카메라 뷰는 항상 **맵 경계 내**에 벗어나지 않습니다.
![[Tower Defender - Waves of War (Media)#^feed51]]
#### `CameraManager 클래스`

![[Tower Defender - Waves of War (Code)#^2f3221]]
모바일 게임 플레이를 위한 **터치 기반 카메라 조작**을 처리합니다.
- [[Tower Defender - Waves of War (Code)#^2f3221| void MobileInput()]]:
    - **한 손가락 드래그**로 카메라를 이동시킵니다.
    - **두 손가락 핀치**로 줌 인/아웃을 수행합니다. 
- [[Tower Defender - Waves of War (Code)#^089f5d|void ClampCameraPosition()]]:
    - 카메라 뷰가 **게임 월드 범위 내**에 항상 머무르도록 보정합니다.
- [[Tower Defender - Waves of War (Code)#^34e87e|void Zoom()]]:
	- **Orthographic size**를 조절하고, 현재 줌 레벨에 맞춰 카메라 범위를 재계산합니다.

---
## **맺는 말**
> 이 프로젝트는 **동적 생성 시스템**, **자율 행동 병사**, **모바일 친화적 조작 방식**을 갖춘 **타워 디펜스 시스템 구축**에 중점을 두었습니다. 비록 규모는 작지만, **확장 가능한 게임 로직 설계**와 **실시간으로 여러 시스템을 안정적으로 관리하는 구조**를 구성하는 데 있어 탄탄한 기반이 되었습니다.