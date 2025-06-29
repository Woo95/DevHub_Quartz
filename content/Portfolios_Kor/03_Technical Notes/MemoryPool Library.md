# **메모리 풀 라이브러리**
---
## **개요**
이 라이브러리는 C++로 작성된 **정적 타입**, **동적 크기** 메모리 풀 시스템을 구현합니다. 메모리 재사용을 통해 할당과 해제를 최적화하여 성능을 향상시키며, 효율적인 메모리 관리를 제공합니다. 모든 메모리 풀의 `생성(create)`, `할당(alloc)`, `해제(dealloc)`은 `CMemoryPoolManager`를 통해서 수행할 수 있습니다.

할당(alloc)의 경우, 이 프로젝트는 **수동 메모리 관리 방식**과 직접 구현한 [[Portfolios_Kor/03_Technical Notes/SharedPtr Library|CSharedPtr]]를 이용한 **자동 메모리 관리 방식** 두 가지를 제공합니다.
> ⚠️ 참고: 메모리 풀 시스템에서 사용하는 [[Portfolios_Kor/03_Technical Notes/SharedPtr Library|CSharedPtr]]는 약간 수정된 버전입니다.
- **수동 메모리 관리 방식**:
	- 특정 메모리 풀을 명시적으로 제어하고 즉시 해제할 수 있어, 성능 최적화와 유연한 메모리 관리가 가능합니다.  
- **자동 메모리 관리 방식**:
	- [[Portfolios_Kor/03_Technical Notes/SharedPtr Library|CSharedPtr]]를 통해 여러 곳에서 특정 메모리를 공유하다가 더 이상 사용하지 않으면 메모리를 자동으로 해제하여 안전한 메모리 관리가 가능합니다.

이 두 방식은 각각의 특징에 따라 선택적으로 사용할 수 있도록 구현되어 있으며, 필요에 따라 메모리 제어의 유연성을 제공합니다.

[**View Repository**](https://github.com/Woo95/MemoryPool)

---
## **클래스 목록 및 설명**
### `IMemoryPool 클래스`
- **역할**:
	- 모든 메모리 풀의 기본 인터페이스이자 **추상 클래스**입니다. `CMemoryPoolManager`는 `void*`를 통해 `CStaticMemoryPool<T>`를 관리할 수 없으므로, `IMemoryPool`을 사용해 `T 타입`의 메모리 풀을 안전하게 관리합니다.
- **핵심 함수**:
	- [[MemoryPool Library (Code)#^da8d6e|virtual ~IMemoryPool()]]:
		- `CStaticMemoryPool<T>`와 같은 파생 클래스가 올바르게 소멸되도록 가상으로 선언되었습니다. 이를 통해서 부모 클래스 포인터로도 소멸자를 정확히 호출할 수 있습니다.
### `CStaticMemoryPool<T> 클래스`
- **역할**:
	- `T 타입` 객체를 위한 **정적 타입** 메모리 풀을 관리하는 클래스입니다. 타입마다 별도의 풀이 생성됩니다.
- **핵심 함수들**:
	- [[MemoryPool Library (Code)#^a512da|void ExpandPool()]]:
		- 메모리 풀의 크기는 **동적으로 조정**됩니다. 초기 크기는 `CMemoryPoolManager`를 통해 `CStaticMemoryPool<T>`의 생성자에서 설정할 수 있습니다. 풀은 필요할 때마다 초기 크기만큼 새로운 메모리 블록을 할당하고, 사용 가능한 블록의 인덱스를 `mFreeIdx` 스택에 추가합니다.
	- [[MemoryPool Library (Code)#^aade06|T* Allocate()]]:
		- `mFreeIdx` 스택에서는 사용 가능한 블록의 인덱스를 관리하여, 가장 최근에 해제되었거나 가장 앞에 있는 블록이 우선적으로 할당됩니다. 포인터가 가리키는 대상은 `placement new`를 사용해 할당된 메모리 블록에서 생성자를 호출해 객체를 생성함으로써 올바르게 초기화하고 가상 함수도 지원합니다. 사용 가능한 블록이 없으면 [[MemoryPool Library (Code)#^a512da|ExpandPool()]]을 호출합니다.
	- [[MemoryPool Library (Code)#^467684|void Deallocate(T* deallocPtr)]]:
		- 포인터가 가리키는 대상의 소멸자를 호출한 뒤, 해당 메모리 블록을 재사용할 수 있도록 `mFreeIdx` 스택에 다시 추가하여 메모리 재사용과 성능을 최적화합니다.
### `CMemoryPoolManager 클래스`
- **역할**:
	-  모든 메모리 풀의 `생성`, `할당`, `해제`를 관리하는 `싱글톤` 클래스입니다. 다양한 타입의 메모리 풀을 생성하고 관리하도록 설계되었습니다.
- **핵심 함수들**:
	- [[MemoryPool Library (Code)#^1ecaa6|bool CreatePool\<T\>(int initCapacity)]]:
		- `T 타입`에 대한 메모리 풀을 생성합니다. 초기 용량은 `initCapacity`를 통해 설정할 수 있습니다. 해당 타입의 풀이 이미 존재하면 새로 생성되지 않습니다.
	- [[MemoryPool Library (Code)#^34624f|bool DeletePool\<T\>()]]:
		- `T 타입`의 메모리 풀을 삭제하고 관련된 자원을 해제합니다.
	- [[MemoryPool Library (Code)#^8be1c4|T* Allocate\<T\>()]]:
		- 동일한 타입의 메모리 풀에서 사용 가능한 `T 타입` 포인터를 반환합니다. 사용 가능한 블록이 없으면 풀이 확장됩니다.
	- [[MemoryPool Library (Code)#^820f8e|void Deallocate\<T\>(T* deallocPtr)]]:
		- 이 함수는 메모리를 해제하고 포인터를 풀에 반환하여 재사용할 수 있도록 합니다. 해제 후, 풀이 더 이상 사용되지 않으면 해당 풀을 삭제합니다.
	- [[MemoryPool Library (Code)#^afa457|void DeallocateButKeepPool\<T\>(T* deallocPtr)]]:
		- 이 함수는 메모리를 해제하고 포인터를 풀에 반환하여 재사용할 수 있도록 합니다. 해제 후, 풀이 더 이상 사용되지 않더라도 해당 풀은 유지됩니다.

---
## **코드 분석**
### `사용 예제 #1 - (수동 메모리 관리 방식)`

![[MemoryPool Library (Code)#^2ce538]]
`Player` 타입의 메모리 풀에서 `CMemoryPoolManager`를 사용한 **수동 메모리 관리** 사용 예제입니다.

---
### `사용 예제 #2 - (자동 메모리 관리 방식)` 

![[MemoryPool Library (Code)#^03d993]]
`Player` 타입의 메모리 풀에서 참조 카운트를 사용하는 [[Portfolios_Kor/03_Technical Notes/SharedPtr Library|CSharedPtr]]를 활용한 **자동 메모리 관리** 사용 예제입니다.