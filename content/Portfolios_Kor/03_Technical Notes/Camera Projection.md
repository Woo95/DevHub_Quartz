# **카메라 투영**
---
## **개요**
이 문서는 2D 및 3D 렌더링 환경에서의 **카메라 투영** 개념을 설명합니다. 원근 투영과 직교 투영의 차이를 소개하고, 실제 프로젝트 사례를 통해 직교 투영의 활용 방식을 간단한 코드와 함께 다룹니다.

---
## **투영의 종류**
> 대부분의 게임 엔진에서는 두 가지 주요 투영 방식을 사용합니다.
### `1. 원근 투영 (Perspective Projection)`
**원근 투영**에서는 카메라로부터 멀어질수록 물체가 작아 보입니다. 이는 시각적인 깊이감을 주며, 착시를 일으켜 장면을 더욱 자연스럽고 실감나게 만듭니다.
![[Camera Projection (Media)#^ef5534]]
#### 특징:
- **원근감**: 카메라에 가까운 물체는 더 크게, 멀리 있는 물체는 더 작게 보입니다.
- **소실점**: 평행한 선들은 멀리 있는 하나의 지점으로 수렴하며, 그로 인해 원근감이 만들어집니다.
- **시야각 (FOV)**: 시야각은 카메라가 볼 수 있는 장면의 각도를 정의합니다. 시야각이 커지면 깊이감이 증가하며, 물체들이 더 작게 보이고 평행선들이 더 날카롭게 수렴하게 됩니다.

---
### `2. 직교 투영 (Orthographic Projection)`
**직교 투영**에서는 물체가 카메라로부터 얼마나 떨어져 있든 크기가 변하지 않습니다. 이 투영법은 2D 게임이나 일부 3D 게임에서 원근감 없는 평면 장면을 구현할 때 자주 사용됩니다.
![[Camera Projection (Media)#^a0ca3a]]
#### 특징:
- **원근감 부재**: 물체가 카메라에 가까이 있든 멀리 있든 크기가 동일하게 보입니다.
- **평행선**: 평행한 선들은 수렴하지 않고 평행 상태를 유지합니다.
- **직교 크기**: 카메라가 볼 수 있는 영역의 높이를 정의하며, 가로 크기는 화면 비율에 따라 자동으로 계산되니다.

---
## **실제 적용 사례**
### `프로젝트: Spaceship Battle`
> *Spaceship Battle*은 예전에 직접 만든 프로젝트로, **직교 카메라 기반의 오브젝트 설정 방식**을 설명하기 위해 사용했습니다.
> 
> [**View Repository**](https://github.com/Woo95/Unity_2D_SpaceShipBattle_Automatic_CameraSetup_With_Object_Pooling)

*Spaceship Battle*에서는 직교 크기가 변경될 때마다 **오브젝트의 위치와 화면 경계**를 실시간으로 조정합니다.![[Camera Projection (Media)#^ed2b15]]
#### `PlayerBehaviour 클래스 - 실행 흐름`
- 게임 시작 시 1회만 호출되어 플레이어를 설정하는 함수들:
	- [[Camera Projection (Code)#^1f0edb|Start()]] → [[Camera Projection (Code)#^1db78b|SetPlayerBoundaryCameraPerspective()]] → [[Camera Projection (Code)#^58ca00|SetPlayerPositionCameraPerspective()]]
- 게임 플레이 중 반복적으로 호출되는 함수:
	- [[Camera Projection (Code)#^f510cd|ClampPlayerYPosition()]]
#### `PlayerBehaviour 클래스 - 코드 분석`
![[Camera Projection (Code)#^1db78b|SetPlayerBoundaryCameraPerspective()]]
이 함수는 카메라의 위치와 직교 크기를 기준으로 플레이어의 Y축 경계를 계산하여, 플레이어가 화면 안에 머물도록 보장합니다.

---
![[Camera Projection (Code)#^58ca00|SetPlayerPositionCameraPerspective()]]
이 함수는 플레이어의 초기 생성 위치를 설정하며, X축과 Y축 모두에서 카메라 시야 내에 정확히 위치하도록 합니다.

---
![[Camera Projection (Code)#^f510cd|ClampPlayerYPosition()]]
이 함수는 플레이어의 Y 위치가 이전에 계산된 경계 내에서만 움직이도록 제한합니다. X위치는 고정됩니다.

---
## **맺는 말**
> **카메라 투영에 대한 이해는 게임 개발의 다양한 측면에서 매우 중요**합니다. 위의 예시는 이러한 지식이 **실제로 어떻게 활용될 수 있는지를 보여주는 하나의 사례**이며, 이와 같은 경우 2D 게임에서는 투영 방식에 대한 이해를 바탕으로 **기기 간 일관된 오브젝트 배치와 화면 경계를 유지**할 수 있습니다. 그 결과 **해상도나 화면 비율이 달라지더라도 자연스럽게 전환**되며, **안정적이고 반응성이 뛰어난 게임 환경**을 구축할 수 있습니다.