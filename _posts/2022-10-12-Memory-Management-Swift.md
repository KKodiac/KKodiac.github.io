# 홍성민 - Swift의 Memory 관리란?

Date: October 18, 2022
Tags: Memory Allocation, Swift

프로그래밍 언어를 배우면서, 그리고 기술 면접에서도 나오는 영역이 언어의 메모리 영역에서 일어나는 일에 관한 것이죠! 저는 iOS 에서 주력으로 쓰이는 `Swift` 라는 언어 내부에서 일어나는 일들에 대해서 공유해보고자 합니다.

우선 이 이야기를 하기 위해서는 Stack 과 Heap 이라는 메모리 영역에 대해서 알아야 합니다. 자료구조에서 나오는 개념과는 아에 다른 것이니 헷갈리지 마세요!

# 🤓기초적인 설명부터 하겠습니다!

컴공 지식이 아에 없으신 분들을 위해서 기본 배경부터 간단히 설명 드리겠습니다!

## 코드의 실행

컴퓨터의 프로그램(또는 코드)는 대표적으로 두가지 방법으로 실행 시킬 수 있습니다. 

- 컴파일(Compile): 흔히 코드 뭉치를 한번에 묶어서 실행하는 방식 | C, Java, Swift 등 언어
    - Compiler: 코드를 여러 오브젝트 파일(.obj) 로 번역
    - Linker: 여러 오브젝트 파일(+ 라이브러리 코드)을 하나의 뭉치로 묶음. 해당 묶음은 실행 파일이 됨
    - Loader: 하나의 실행 파일을 **메모리에 적재**함(실행시킴)
- 인터프리드: 코드의 한줄 한줄을 파일 실행 시 실행 시키는 방식 | Python, Javascript
    - 여기서는 다루지 않겠습니다.

Swift 같은 경우도 컴파일러가 있고 코드가 Compile - Link - Load 방식으로 준비가 되며 메모리에 적재되어 실행 됩니다. 여기서 소스 코드의 내용이 메모리의 어느 부분에 적재가 되는지의 여부가 달라지게 되는데, **크게 저희는 Stack 과 Heap 부분으로 나누어 보려고 합니다.**

## Call by Value - Call by Reference - ~~Call by Address~~

![Screen Shot 2022-10-12 at 2.19.46 AM.png]({{site.baseurl}}/assets/img/exstruct.png)

![Screen Shot 2022-10-12 at 2.29.41 AM.png]({{site.baseurl}}/assets/img/exclass.png)

Swift 언어에서는 총 3가지로 타입 선언이 이루어집니다. `class` `struct` 그리고`enum` 타입입니다. 위 사진과 같이 Int 타입 또한 `struct` 타입이라고 나와 있으며, 흔히 View Model 에서 사용되는 ObservableObject 는 `class` 로 의 일종인 `protocol` 타입 입니다. 여기서 `class` `struct` 그리고`enum` 이 갖는 성질에 따라서 두 가지로 분류하는데, `**Value`** 타입(`struct`, `enum`) 과 `**Reference**` 타입(`class`) 로 나뉩니다. 

- Value Type: 새로운 변수에 값을 할당할 때, 해당 변수에 값을 직접 저장합니다. 즉, 메모리에 새로운 변수에 할당된 값의 자리를 새로 만들어줘야 합니다.
- Reference Type: 새로운 변수에 값을 할당할 때, 해당 변수의 값은 메모리 한 구석에 있고, 변수는 해당 메모리를 가리키고 있을 뿐입니다. 여러 변수가 같은 메모리 위치의 값을 가리킬 수(참조할 수) 있습니다.

그래서 프로그램이 실행되면서 코드에 적힌 변수의 타입에 따라서 메모리에서 해당 값이 호출이 될 때 Call by Value(Value 타입으로 지정된 변수의 메모리 호출) 또는 Call by Reference(Reference 타입의 포인터(참조) 호출)가 되는 것입니다.

자, 이제 두가지 타입을 알았으니 해당 타입의 변수들이 어느 메모리 부분에 저장되는 지 알아야겠습니다.

# Stack vs Heap

![image.png]({{site.baseurl}}/assets/img/stackheap.png)

## Stack이란?

Value Type의 데이터를 저장하는 장소 입니다. Swift 언어에서는 번외로 Reference Type 이 저장이되는 경우가 있긴 합니다. “Stack Promotion” 이라고 불리며 이것에 대해서는 다른 글로 정리해보겠습니다. 

Stack 의 특징을 살펴보자면,

- Memory allocation: 값들이 Compile 될 때 Stack에 할당이 됩니다.
- Temporary memory allocation: Compile 시 해당 메서드(struct)가 호출되어 실행될 때에만 값이 접근 가능합니다. 즉, Stack 메모리 할당은 Compiler 몫 입니다.
- Thread Safety: 오직 해당 메서드(struct)가 실행 중인 쓰레드에서만 접근 가능합니다. 쉽게 말해서 해당 작업을 하고 있는 일꾼만 그 일을 제어할 수 있는 겁니다.
- Speed and Storage: 할당과 비할당 속도가 빠름니다. 저장 공간이 비교적 적습니다.
- Automatic: 자동으로 메모리를 할당하고 비할당 시킵니다.

## Heap이란?

Reference Type의 데이터를 저장하는 장소 입니다. Swift 언어에서는 번외로 Value Type 이 저장이되는 경우가 있긴 합니다. “Boxing” 이라고 불리며 위 Stack Promotion 과 함께 다루겠습니다. 

Heap의 특징으로는,

- Memory allocation: Run-time에 Heap에 할당 됩니다. Run-time은 loader 에 의해 프로그램이 실행되고 있을 때를 말합니다.
- Dynamic memory allocation: Heap 메모리 할당은 프로그래머의 몫 입니다. 따라서 할당된 메모리를 비할당 상태로 만드는 것도 프로그래머가 해야합니다. (Memory leak 의 가능성)
- Thread Safety: Heap 영역의 메모리는 모든 쓰레드에서 접근 가능합니다. 쉽게 말해 모든 일꾼이 작업에 관여 할 수 있다는 뜻 입니다.
- Speed and Storage: 할당과 비할당 속도가 느립니다. 저장 공간이 비교적 넓습니다. (Virtual Memory까지 먹기 때문입니다.)
- Automatic: 프로그래머가 알아서 사용되지 않는 값을 메모리에서 비할당 시켜야합니다. (i.e. C에서 malloc(), free())

이렇게 각 메모리 영역의 특징을 알아보았습니다. 이제 iOS 에서 해당 메모리를 어떻게 관리하는 지 알아볼까요?

# ARC(Automatic Reference Counting)이란?

위 Stack 과 Heap 의 차이에서 Heap은 메모리 관리를 프로그래머로 전가 시키기 때문에 메모리 누수(Memory Leak)라고 불리는 문제가 일어날 수 있습니다. 결국 할당된 메모리를 비할당 시키지 못하고 계속 메모리를 잡아 먹고 있기에 생기는 문제인데, 이것을 해결하기 위해 Java는 JVM이 관리하는 Garbage Collector가 있죠? Swift 에서는 **ARC(Automatic Reference Counting: 자동으로 클라스의 참조 개수를 세어주는 기능)**가 있습니다.  

기존 iOS 개발에 사용되었던 Objective-C 라는 언어에서 제공한 MRC 방식과 유사하지만 자동(Automatic RC) 와 수동(Manual RC)이란 방식의 차이를 갖습니다. 그렇다면 ARC 와 MRC 의 자동/수동의 차이는 어떤건지 살짝만 알아보도록 하죠. 

**MRC** 는 `retain` 과 `release` 라는 키워드를 사용해서 메모리 할당과 비할당을 직접 작성해 줘야합니다. C 에서 `malloc()` 과 `free()` 를 사용하는 것과 유사하죠. 이렇게 수동으로 참조하는 개수를 세어야 합니다.

**ARC** 는 메모리의 할당과 비할당을 Compiler 가 알아서 해주는 기능입니다. 객체의 참조 개수를 세어가면서 해당 객체에 대한 참조가 0이 되면 `deinit` 과 deallocation 을 진행하죠. 

이렇게 보면 ARC 의 사용은 훨씬 간편해 보이지만 여기서 캐치가 있습니다. 만약 객체 A 와 객체 B 가 서로를 참조하고 있다면 어떤 일이 벌어질까요?

![1*KAsfeBmtyqO7-heoG92IBg.png]({{site.baseurl}}/assets/img/memleak.png)

서로의 참조 개수는 1로 유지되면서 두 객체가 할당 받은 메모리는 계속 할당되어 있을 것이므로 메모리 누수가 일어납니다. 이것을 retain cycle 또는 reference cycle 이라고 합니다. 그렇다면 retain cycle 을 없애기 위해서는 어떻게 코드를 짜야할까요?

### 객체 생명주기

우선 그것을 알기 전에 객체의 생명주기를 파악해 봅시다. 

1. Allocation: Stack 또는 Heap 의 메모리를 할당 받습니다.
2. Initialization: `init` 함수로 인해 객체가 초기화 됩니다.
3. Use: 객체가 사용(실행) 됩니다.
4. De-initialization: `deinit` 함수로 인해 객체가 소멸 됩니다.
5. Deallocation: Stack 또는 Heap 에 할당된 메모리를 반환 합니다.

### 메모리 관리 문제

- 객체가 사용중일 때 해당 객체를 비할당 시키거나 데이터를 덮어 쓸때
- 객체가 사용 중이지 않을때 객체 할당을 비워주지 않을때
    - → 이것을 ARC에서 자동으로 해주는데, 여기서 일어나는 retain cycle 로 인한 memory leak 을 해결하는 방법을 아래서 알아보겠습니다.

### `weak` 그리고 `unowned`

위 strong reference 로 인해 생기는 retain cycle 의 문제는 객체 둘 중 하나의 참조를 `weak` 또는 `unowned` 참조로 지정해 주면서 해결할 수 있습니다. 그럼 여기서 `strong` `weak` 그리고 `unowned` 참조는 무엇이냐: Swift 에서 정의된 참조 종류 입니다. 아래 테이블을 참고 하자면,

| Reference Type | var | let | optional | non-optional |
| --- | --- | --- | --- | --- |
| strong | ✅ | ✅ | ✅ | ✅ |
| weak | ✅ | ❌ | ✅ | ❌ |
| unowned | ✅ | ✅ | ❌ | ✅ |

각 `weak` 그리고 `unowned` 참조 타입에 대해서 각각 자세히 알아보죱. 

- Weak Reference: 객체 정의 시 앞에 붙혀주는 키워드 `weak` 을 통해 객체를 약한 참조를 갖는다고 지정할 수 있습니다. 항상 `optional` 형으로 선언이 되며 이렇게 선언된 객체의 reference count 를 증가 시키지 않습니다. 결국 참조가 없어지게 되면 `nil` 로 남게 됩니다.
- Unowned Reference: Weak reference와 유사하게 약한 참조가 가능하며 대신 `optional` 지원을 하지 않습니다. 이 뜻은 객체에 대한 참조가 끝나게 되면 해당 메모리를 가리키는 포인터(댕글 포인터)로 남게 되는데, 해당 포인터를 참조하게 되면 crash가 나게 됩니다. 결국 객체가 사라지지 않을 것이라고 보장될 때만 `unowned` 는 설저하는 것이 좋습니다.

결국 둘 다 retain cycle 을 해결하는데 사용이 되며, `unowned` 는 사라지지 않는 것이 보장되는 객체에, `weak` 은 그 외에 케이스에 사용하면 되겠습니다. 

# 마치며…

다음에는 ARC 에 대해서 더 자세하게 알아 보겠습니다.

### 참고문헌

weak and unowned → [https://shark-sea.kr/entry/iOS-ARC-strong-weak-unowned](https://shark-sea.kr/entry/iOS-ARC-strong-weak-unowned)

Compiler → [https://stackoverflow.com/questions/3996651/what-is-compiler-linker-loader](https://stackoverflow.com/questions/3996651/what-is-compiler-linker-loader)

Swift Mem Man → [https://medium.com/elements/memory-management-in-swift-31e20f942bbc](https://medium.com/elements/memory-management-in-swift-31e20f942bbc)

보면 좋은 글 → [https://shark-sea.kr/entry/iOS-Swift-메모리의-Stack과-Heap-영역-톺아보기](https://shark-sea.kr/entry/iOS-Swift-%EB%A9%94%EB%AA%A8%EB%A6%AC%EC%9D%98-Stack%EA%B3%BC-Heap-%EC%98%81%EC%97%AD-%ED%86%BA%EC%95%84%EB%B3%B4%EA%B8%B0)

Swift Types → [https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/](https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/)

Virtual Memory →[https://www.techtarget.com/searchstorage/definition/virtual-memory](https://www.techtarget.com/searchstorage/definition/virtual-memory)

SIL → [https://techblog.woowahan.com/2563/](https://techblog.woowahan.com/2563/)

Stack vs Heap →[https://www.geeksforgeeks.org/stack-vs-heap-memory-allocation/](https://www.geeksforgeeks.org/stack-vs-heap-memory-allocation/)

Stack Heap Types → [https://stackoverflow.com/questions/27441456/swift-stack-and-heap-understanding](https://stackoverflow.com/questions/27441456/swift-stack-and-heap-understanding)

Manual Memory Management → [https://stackoverflow.com/questions/15285831/manual-memory-management-vs-arc](https://stackoverflow.com/questions/15285831/manual-memory-management-vs-arc)