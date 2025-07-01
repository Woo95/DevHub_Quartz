# **오브젝트 풀링**
---
## **개요**
이 기술은 **성능 최적화** 기법으로, **생성된 오브젝트를 재사용**하여 빈번한 메모리 할당과 해제를 피합니다. [[Portfolios_Kor/03_Technical Notes/MemoryPool Library|메모리 풀]]과 유사하게, 완전히 초기화된 오브젝트 객체를 재사용하는데 중점을 둡니다. 지속적으로 생성과 소멸하는 오브젝트에 이상적입니다.

> ⚠️ 참고: 이 기법은 재사용으로 **CPU 오버헤드를 줄여** 성능을 개선하지만, 사전 할당으로 **메모리 사용이 늘어**날 수 있습니다.

---
## **구조 설계도**
![[object pooling structure diagram_Kor.png]]
 - **흰색 테두리**: 처리 흐름  
 - **노란색 테두리**: 오브젝트 집합체
---
## **실제 적용 사례들**

### `1. 프로젝트: Infinite Runners`
> *Infinite Runners*는 예전에 만든 프로젝트로, 오브젝트 풀링 설명하기 위해 사용했습니다.

*Infinite Runners*의 `MapGenerator` 클래스는 **배열 기반 풀 구조**를 사용해 오브젝트 풀링으로 맵을 무작위로 재사용하며 다양한 맵 시퀀스를 제공합니다.
![[Object Pooling (Media)#^a71532]]
#### `MapGenerator 클래스 - 실행 흐름`
- 이 함수들은 게임이 시작될 때 한번만 호출되어 맵 풀을 생성 및 구성합니다:
	- [[Object Pooling (Code)#^b06806|Start()]] → [[Object Pooling (Code)#^27dd05|CreateMapPool()]] → [[Object Pooling (Code)#^31cd3b|InitializeFreeIndices()]] → [[Object Pooling (Code)#^302dda|SpawnMapsAtStart()]]
- 이 함수들은 게임 플레이 중 호출됩니다:
	- [[Object Pooling (Code)#^612f56|OnTriggerEnter(Collider other)]] → [[Object Pooling (Code)#^5232ac|ReplaceMap(int oldIdx)]]

#### `MapGenerator 클래스 - 코드 분석`
![[Object Pooling (Code)#^27dd05]]
모든 맵 오브젝트는 한번만 생성되어 배열에 저장되며, 기본적으로 비활성화된 상태로 설정됩니다. 각 맵은 `MapBehaviour` 컴포넌트를 통해 고유한 풀 인덱스를 할당받아 추적됩니다.

---
![[Object Pooling (Code)#^31cd3b]]
이 함수는 미리 생성된 맵 오브젝트의 모든 인덱스를 리스트에 담아, 현재 재사용 가능한 맵을 추적하는 역할을 합니다.

---
![[Object Pooling (Code)#^302dda]]
풀에서 `m_SpawnAmount`만큼의 맵을 무작위로 선택해 순차적으로 배치하고 활성화하여 시작 환경을 만듭니다.

---
![[Object Pooling (Code)#^612f56]]
이 함수는 충돌체가 트리거 존에 들어올 때 호출됩니다. 충돌체가 `"Map"` 태그와 `MapBehaviour` 컴포넌트를 갖고 있으면 해당 풀 인덱스를 `ReplaceMap()`에 전달하고, 그렇지 않으면 오브젝트를 제거합니다.

---
![[Object Pooling (Code)#^5232ac]]
이 함수는 사용 가능한 인덱스를 선택해 풀에서 새 맵을 재배치하고 활성화하며, 기존 맵을 비활성하고 해당 인덱스를 나중에 재사용할 수 있도록 풀에 되돌려 넣습니다. 이를 통해 잦은 인스턴스화를 피하면서 풀에 재사용 가능한 맵을 유지합니다.

---
### `2. 프로젝트: Spaceship Battle`
> *Spaceship Battle*는 예전에 만든 프로젝트로, 오브젝트 풀링 설명하기 위해 사용했습니다.
> [**View Repository**](https://github.com/Woo95/Unity_2D_SpaceShipBattle_Automatic_CameraSetup_With_Object_Pooling)

*SpaceShip Battle*의 `BulletManager` 클래스는 **큐 기반 풀 구조**를 사용해 오브젝트 풀링으로 총알을 재사용합니다. 이는 위의 프로젝트 *Infinite Runners*와 달리 무작위 선택이 필요하지 않기 때문입니다.
![[Object Pooling (Media)#^b6ae16]]
#### `풀 미적용 vs. 풀 적용 - 성능 비교`
![[Object Pooling (Media)#^65f1dd]]
위 이미지에서 볼 수 있듯이, **FPS 차이**는 왼쪽의 **풀 미적용 상태가 성능이 훨신 낮고**, 오른쪽의 **풀 적용 상태가 훨씬 더 높은 성능**을 보여줍니다.

---
## **맺는 말**
> **오브젝트 풀링은 강력한 기법**으로, **런타임에 자주 생성되고 소멸되는 오브젝트를 효율적으로 관리**합니다. 그러나 **너무 많은 오브젝트를 사전에 생성하면 상당한 메모리 낭비로 이어질 수 있습니다**. 따라서 런타임 중에 필요한 오브젝트 수를 신중히 예측하고, 그에 맞춰 **적절한 풀 크기를 설정하여 과한 메모리 할당을 방지하는 것이 중요**합니다. 마지막으로, 제 두 프로젝트의 오브젝트 풀링 적용 사례에서 볼 수 있듯이, **상황에 따라  풀에 맞는 적합한 자료 구조를 선택하는 것도 매우 중요**합니다.