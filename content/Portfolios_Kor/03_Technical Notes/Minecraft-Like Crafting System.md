# **마인크래프트 스타일 제작 시스템**

---
## **개요**
이 프로젝트는 **기본적인 아이템 인벤토리**와 **마인크래프트 스타일 제작 시스템**을 구현을 다루고 있습니다. 유저는 아이템을 제작대에 아이템을 조합하여 새로운 아이템을 만들 수 있습니다. 이 시스템은 **쉬운 확장성을 고려하여 설계**되었기 때문에, 개발자가 **새로운 레시피를 손쉽게 추가**할 수 있습니다.

[**View Repository**](https://github.com/Woo95/Unity_Minecraft_Like_Crafting_System)<br/>[**Watch Demo**](https://www.youtube.com/watch?v=wVEw_mPlB5I)

---
## **아이템 제작 시연**
아래 영상에서는 시스템에 미리 정의된 8가지 레시피를 사용하여 아이템을 제작하는 과정을 보여줍니다.
![[Minecraft-Like Crafting System (Media)#^737682]]

---
## **클래스 목록 및 설명**

#### `Item 관련 클래스`

- [[Minecraft-Like Crafting System (Code)#^c616c6|ItemInfo class]]:
	- 아이템의 아이콘, 설명, 그리고 `eItemType`을 저장하는 `ScriptableObject`로, 아이템의 모든 객체가 공유하는 정보를 담고 있습니다.
	- [[Minecraft-Like Crafting System (Code)#^7e13a8|eItemType enum]]:
		- 각 아이템을 식별하기 위한 고유 ID를 정의하며, **제작 알고리즘**에서 **아이템 구분**을 하는데 사용됩니다.
- [[Minecraft-Like Crafting System (Code)#^c6f0e7|ItemInstance class]]:
	- 인벤토리 UI의 `ItemSlot`을 나타냅니다. `ItemInfo`와 수량을 추적하며, 아이콘과 수량 표시를 갱신하고 마우스를 올리면 아이템 상세정보를 보여줍니다. 
#### `ItemSlot 관련 클래스`
- [[Minecraft-Like Crafting System (Code)#^6bfaef|ItemSlot class]]:
	- 아이템의 **들기, 놓기, 바꾸기, 쌓기**와 같은 상호작용을 관리합니다.
	- **인벤토리**와 **제작 시스템**에서 공통으로 사용하는 **`ItemSlot`의 기반 클래스**입니다.
- [[Minecraft-Like Crafting System (Code)#^0d06b1|CraftingInputSlot class]]:
	- `ItemSlot`을 **상속**하며 **제작 재료 칸**의 아이템 배치를 처리합니다.
	- 재료 아이템 상호작용 이후, 제작 시스템을 갱신합니다.
- [[Minecraft-Like Crafting System (Code)#^9be179|CraftingOutputSlot class]]:
	- `ItemSlot`을 **상속**하며 **제작 생성 칸**의 아이템 배치를 처리합니다.
	- 제작 생성 칸에 아이템을 생성하고, 가져가면 제작 재료 칸의 재료를 갱신합니다.
#### `Crafting 관련 클래스`
- [[Minecraft-Like Crafting System (Code)#^24120e|Recipe class]]:
	- 필요한 **제작 재료**와 **생성되는 아이템**의 **제작 공식**을 저장하는 `ScriptableObject`입니다.
- [[Minecraft-Like Crafting System (Code)#^b3c61a|Crafting class]]:
	- **전반적인 제작 시스템의 동작**을 관리합니다.
	- 제작 재료 칸의 아이템이 **레시피와 일치하는지 확인**하고, 일치할 경우 **제작 아이템을 생성**합니다.

---
## **프로젝트 주요 특징**

### `1. 재정의 가능한 아이템 칸(ItemSlot) 상호작용`
- 재사용 가능한 이벤트 및 조건 로직.
- `ItemSlot` 상속을 통한 손쉬운 확장성.
#### **작동 원리**
`ItemSlot` 클래스는 아이템 칸 상호작용을 위한 유연하고 모듈화된 기반을 제공합니다.
![[Minecraft-Like Crafting System (Code)#^cc1460]]
- `virtual void OnPointerClick()` 함수는 아이템 칸이 클릭에 어떻게 반응할지를 결정합니다.
- 이 함수 내부에서는 **미리 만들어둔 이벤트**와 **조건 함수들**을 **블록처럼 조합**하여 원하는 동작을 만들 수 있습니다.
- 이 방식 덕분에 핵심 로직을 다시 작성하지 않고도 **복잡한 상호작용**을 손쉽게 정의 할 수 있습니다.

---
`CraftingOutputSlot` 같은 자식 클래스는 아래에 정의된 가상 함수를 간단히 오버라이드하여 원하는 동작을 재정의 할 수 있습니다.
![[Minecraft-Like Crafting System (Code)#^8a723c]]
**결과:** 아이템 칸 유형별로 **쉽게 확장, 디버그, 맞춤 설정**할 수 있는 깔끔하고 재사용 가능한 동작 구조를 얻을 수 있습니다.

---
### `2. 쉬운 레시피(Recipe) 설정`
- `ScriptableObject`를 통한 새로운 레시피 추가.
- 직관적인 시각화 설정을 위한 `RecipeEditor`.
#### **작동 원리**
[[Minecraft-Like Crafting System (Code)#^24120e|Recipe]]는 `ScriptableObject`로, [[Minecraft-Like Crafting System (Code)#^f0a932|RecipeEditor]]의 드래그-앤-드롭 인터페이스를 통해 손쉽게 편집할 수 있습니다.
![[Minecraft-Like Crafting System (Media)#^4c0250]]
**결과:** 위의 `RecipeEditor` 사용 전후 예시에서 보듯이, `Recipe`를 **손쉽게 설정**할 수 있어, 제작 레시피의 **생성과 관리를 빠르고 실수 없이** 할 수 있고, **개발자와 디자이너 모두가 간편하고 쉽게 활용**할 수 있습니다.

---
### `3. 유연한 제작(Crafting) 알고리즘`
- 어떤 사이즈의 n x n 제작 격자도 대응.
- 모양 기반 레시피 판별 방식.
#### **작동 원리**
![[Minecraft-Like Crafting System (Media)#^32d465]]
**제작 알고리즘은 아주 직관적입니다:**
1. 제작 재료 칸에서 [[Minecraft-Like Crafting System (Code)#^7e13a8|eItemType]] **고유 ID**를 **행 단위**로 하나의 문자열로 결합시킵니다.
	- `<Sample 3>`과 같이 아이템이 **여러 행에 연속적으로 배치**된 경우, 행 사이에 **추가 문자를 삽입**하여 구조를 유지합니다.
2. 생성된 문자열에서 **외각의 "빈 슬롯 아이템 타입" 문자**를 **제거**합니다.
3. **모양만 동일하면** 아이템 배치가 **어느 위치에 있어도 정확히 일치**하는 것을 확인할 수 있습니다.
4. 마지막으로, 만들어진 문자열을 정의된 모든 레시피 문자열과 비교하여 일치하는지를 확인합니다.

---
## **맺는 말**
> 이 제작 시스템은 **유연성**, **확장성**, 그리고 **간편한 레시피 구현 기능**을 제공합니다. 직관적인 설계 덕분에 **통합과 사용자 정의**가 쉽고, 개발자뿐만 아니라 **디자인 팀도 복잡한 코드 없이 레시피를 효율적으로 관리**할 수 있습니다.