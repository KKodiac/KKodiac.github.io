---
layout: post
title: Data Types in Swift (1)
date: 2022-03-25 05:38:00 +0900
description: Swift에 있는 다양한 데이터 타입을 알아보자 # Add post description (optional)
img: swift-logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Swift, Struct, Class, Generics, Functions, Closures, DataType]
---
# Varieties of Types
Swift언어에서 제공되는 다양한 타입에 대해서 알아보자
2021년도 스탠포드 CS

## Table of Contents
 - [struct and class](#struct_&_class)
 - [generics](#generics)
 - [functions](#functions)
 

## struct & class
Swift 언어에서는 struct 와 class 를 둘 다 지원한다.

### 유사한 점
우선 Struct 와 Class 는 매우 비슷한 문법 구조를 갖는다.

둘 다

 - 저장된 vars
 - 계산된 vars
 - 상수인 lets
 - functions 
    - 내부 method 및 함수를 정의할 수 있다.
    ```swift
    func multiply(_ operand: Int, by otherOperand: Int) -> {
        return operand * otherOperand
    } 
    ```
 - initializers 
    - 초기 생성자를 정의할 수 있다. 즉, 다형성을 지원한다.
    ```swift
    struct AutoMobile {
        init(vehicleName: String) {
            // 차량의 이름을 initialize
        }
        init(numberOfWheels: Int) {
            // 차량의 바퀴 수를 initialize
        }
    }
    ```

를 포함하고 있다.

### 다른 점

둘 다 매우 유사한 구조를 띄고 있지만 결국 다른 상황에서 사용되기 때문에 각각 언어에서 사용될 것이다. struct 와 class 를 비교해보자

Data Type | *Struct* | *Class*
----------|----------|---------
호출 방식 | *Value Type | *Reference Type 
새로 변수 지정 시 동작 | 변경 시에 복사됨 | 자동적으로 레퍼런스 갯수를 셈
프로그래밍 패러다임 | 함수형 프로그래밍 | 객체지향형 프로그래밍
상속(Inheritance) 가능 여부 | 상속 없음 | 단일 상속 가능
"공짜" `init` 동작 | 모든 `vars` 를 초기화함 | `vars` 초기화 없음
Mutability 여부 | 명시해야만 가능 | 항상 Mutable
우선적으로 사용할 것? | ✅ | 특이 케이스에 활용
SwiftUI MVVM 에서의 역할 | `protocol`인 View를 제외한 모든 것 | ViewModel

위 내용 중 상세 설명이 필요한 부분을 알아보도록 하자

------------------------------------------------------------------

#### 호출 방식 및 동작
`struct` 는 **Value** 타입이다. 즉, 전달이 되거나 변수에 지정되었을 때 같은 복사본이 생겨나게 된다. 여기서 참고 해야 될 것은 복사본이 호출이 되었다고 무지성으로 생겨나는 것이 아니라 *Swift* 에서는 해당 `struct` 가 다른 내용이 추가되어 쓰여지게 되면 그때 새로운 복사본을 정의하게 된다. 이것을 강의에서는  `Copy on Write` 이라고 불렀다. 

`class` 는 **Reference** 타입으로 호출이 될 때 자동적으로 힙에 해당 객체를 지정하는 포인터를 생성하여 저장한다. 특이한 점은 *Swift* 에서는 이러한 포인터들이 생성이 될 때마다, 그 갯수를 트래킹 하는데, 포인터의 갯수가 0이 될 때 해당 `class` 에 할당된 메모리를 free 한다. 이것을 강의에서는 `Automatic Reference Counting` 이라고 불렀다. 


------------------------------------------------------------------

#### 프로그래밍 패러다임 및 상속 여부
`struct` 는 함수형 프로그래밍을 기반으로 한다. 결국 이 의미는 `Swift` 프로그램에서 불안정함을 제거해야하는 컴포넌트에 적용할 때 사용된다는 의미이다. 그래서 `SwiftUI` 에서도 불변성을 보장해야되는 부분에서 `struct` 를 사용한다. 상속 또한 그렇기 때문에 존재하지 않는다.

`class` 는 객체지향 프로그래밍 인데, 공유되는 자원(함수)에 대해 정의 할 때 사용하면 좋다. 포인터로 자원이 공유되기 때문에 `SwiftUI` 에서는 ViewModel 부분을 `class` 로 짜야하는 이유도 그것이다. 객체지향에 상속은 빠질 수 없으니 지원된다. 

------------------------------------------------------------------
 

### Generics
*Swift* 는 `generic` 을 지원한다. `Generic` 에 대해서 강의에서는 "don't care" 타입이라고 설명한다. *Swift* 는 strongly-typed 언어이기 때문에 타입이 지정되지 않은 변수를 사용할 수 없다. 따라서 `generic`을 사용해 자료 구조 안에 어떠한 타입이 들어갈지 무관함을 보여주고 호출할 때 넣을 타입을 정해준다. 

대표적인 `generic` 타입의 예시로 `Array` 가 있다. `Array` 는 아무런 타입이 내부 변수로 들어가 저장될 수 있기 때문에 `generic` 으로 정의해 준 후 호출할 때 들어갈 타입을 명시해주는 형식으로 정의된다. 
```swift
struct Array<Element> {
    ...
    func append(_ element: Element) { ... }
}
```
위 처럼 `append` 에 argument 로 들어가는 값의 타입은 `generic` 타입인 `Element` 이다. 
```swift
var a = Array<Int>()
a.append(52)
a.append("Hello") // 오류!
```
이렇게 `Array` 가 호출이 될 때 들어갈 타입을 명시해 준다.
또한 다수의 `generic` 을 지정해 줄 수 있다. 
```swift
struct Array<Element, Foo, Bar> { ... } 
```

`Generic` 은 `Type Parameter` 라고도 불리는데, *Swift* 에서는 `generic` 을 `protocol` 과 묶어 사용할 수도 있다. 자세한 것은 다음 글에...


### Functions

#### Functions as Types
함수형 프로그래밍의 일환?으로 함수들도 타입으로 정의 될 수 있다. 따라서 변수, 매개변수, 함수를 정의할 때 `function` 으로 타입을 설정할 수 있다. *Swift* 의 특징이라고 말씀하시는데 처음 듣고 으잉? 싶었다. 

예를 들어...
```swift
(Int, Int) -> Bool  // 두 Int 를 매개변수로 갖고 Bool 을 반환하는 함수 타입
() -> Array<String> // 아무런 인자를 갖지 않고 Array<String> 을 반환하는 함수 타입
() -> Void          // 아무런 인자를 갖지 않고 void 를 반환하는 함수 타입
```
는 모두 타입으로 정의된다!

그래서 
```swift
var operation: (Double) -> Double // Double 인자에 Double 반환 함수.
func square(operand: Double) -> {
    return operand * operand
}

operation = square
let result = operation(4)       // 인자 label을 명시하지 않는다. 값 16
operation = sqrt
let result2 = operation(4)      // 값 2
```
처럼 정의하여 사용할 수 있다. 

#### Closures
`"Inline" function` 으로 함수들을 전달하는 방식이 있는데, 이를 `closure` 라고 부른다. 즉, 함수명 없이 함수를 정의해주는 경우를 말한다. *Python* 의 `lambda` 와 비슷한 것 같아서 차이점을 잠깐 알아보고 넘어갈텐데, 스택 오버플로에 좋은 [답글](https://stackoverflow.com/a/36878651/8090273)이 있어서 참고하면 좋을 것 같다. 

두 개념 모두 Lambda Calculus(람다 대수학) 이라는 수학적 개념에서 부터 시작 되었다. `Lambda` 는 그저 익명의 함수(이름이 없는 함수) 를 말한다. 익명이기 때문에 선언 이후에 호출이 되는 것은 불가능하다(다른 변수명으로 정의되지 않는 이상). 하지만 선언 후 바로 호출이 되기 때문에 많은 프로그래밍 언어에서 활용이 되고 있다.

`Closure` 에 대한 설명을 나아가기 전에 람다 연산에서 두가지 상태를 우선적으로 알아볼 필요가 있다. 람다식은 
- Closed 된 식 : 람다식 안에 모든 값(-,+ 같은 symbol 포함)들의 정의가 식 내부에서 해결된다.
- Open 된 식 : 람다식 안에 있는 값들 중 정의가 되지 않은 값이 포함되어 있어서 외부에서 공급되는 값의 정의를 불러와야 한다. 

`Closure` 은 여기서 한층 더 나가는 부분이 있다. 익명의 함수 내 연산에 필요한 모든 값들이 정의가 되어 있지 않는다면, 해당 값들을 익명 함수에 공급해야 해당 함수가 풀어지게 된다. 이렇게 값이 정의되어 있지 않는 값들을 "자유"인 상태라고 본다. 여기서 공급 되는 "환경(environment)" 은 이러한 자유로운 값들에 대해 정의를 공급한다. 

다시 말해  `closure` 은 기존에 Open 되어 있던 람다식일 수 있던 것을 외부에서 "자유"인 상태의 값의 정의가 공급하면서 Closed 상태가 된 람다식을 말한다.

```swift
// 람다 형식의 클로저
{ 
    print("Hello, World!")
}

// 클로저 정의 후 변수 지정
var findSquare = { (num: Int) -> (Int) in // 외부환경에서 공급되는 값 정의
    return num * num
}

// 클로저(함수) 호출
print(findSquare(5)) // 25
```


## Reference 
[Stanford CS193P](https://youtu.be/--qKOhdgJAs)
[What is the difference between a 'closure' and a 'lambda'?](https://stackoverflow.com/questions/220658/what-is-the-difference-between-a-closure-and-a-lambda)
[Closure Examples](https://www.programiz.com/swift-programming/closures)