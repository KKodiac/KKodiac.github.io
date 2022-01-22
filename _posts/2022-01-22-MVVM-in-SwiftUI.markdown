---
layout: post
title: MVVM in SwiftUI
date: 2022-01-22 05:38:00 +0900
description: SwiftUI 에서 적용되는 MVVM 아키텍쳐 알아보기 # Add post description (optional)
img: swift-logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Swift, SwiftUI, iOS, MVVM]
---
# SwiftUI 에서의 MVVM

![MVVM-Simple]({{site.baseurl}}/assets/img/mvvm-simple-flow.jpeg)

## Model-View-ViewModel

- Architectural design paradigm
- SwiftUI를 활용하기 위해선 지켜야 한다.
- UIKit이 사용했던 MVC(Model View Controller)과는 다른 디자인 패턴이다.

> **View 는 필요한 Model 의 데이터에 대해서 ViewModel 을 통해 받는다**
> 

## MVVM 정의

### Model

- UI 독립적이다
- Data + Logic
- 데이터를 저장하는 유일한 개체

### View

- Model 에 있는 데이터 등을 상태를 반영함

- View에는 세가지 특징이 있다.
    - Stateless(상태 없는)
        - Model 에서 항상 모든 데이터 및 State 을 Fetch 만 하기 때문이다
    - Declared(선언된)
        - View 에 무엇과 언제 반영이 되는 지는 이미 **선언**된 `var body` 에 구현된 부분이기 때문이다
        - View 는 Immutable 하다
        - View와 Model 사이의 오류 발생 시 오직 declared 된 부분, 즉 `var body` 부분만 확인하면 된다.
        - 따라서 위 이유로 인해 (**Locality of Code로 인해 →**) declarative coding 이 SwiftUI에서 사용된다
    - Reactive(반응적인)
        - Model 의 변화에 영향이 있는 View 부분만 rebuild 하게 된다.
        - 위는 ViewModel에 의해 가능하게 되는데, 이는 [아래](#architecture-flow)에서 설명한다
        

### ViewModel

- View와 Model 을 엮어주는 역할을 한다
- Gatekeeper 로써 데이터의 접근을 제어한다
- Interpreter
    - Model 을 View 에서 처리하기 쉬운 데이터 타입으로 치환해줌
        - ex) SQL Model → Array Model 등
    - 따라서 View 가 필요한 Model 의 데이터 및 데이터 타입을 제공해야 한다
- 또한 사용자의 의도(intent)를 처리함에 있어서 Intent 함수를 제공한다
    
    

## Architecture Flow

**Model → ViewModel → View** 

- 저장된 데이터의 변환 시
    1. ViewModel 은 지속적으로  Model 의 변화를 트래킹
    2. Model 변화 감지 시 **모든 View** 에게 변화를 알림 (Publish)
    3. View 에서 자동적으로 publication 감지 후 Model 변화에 해당하는 View 만 rebuild

**View → ViewModel → Model**

- 사용자가 데이터 변환 시
    1. View 에서 사용자가 의도한 데이터 변환 발생 시 ViewModel의 Intent 함수 호출
        - User Intent 가 그저 간단한 데이터 저장 시 Store(direct access to model) 라는 개념을 사용한다. 즉 Intent 함수를 사용하지 않는다
        - 따라서 Store 을 활용하여 View 작성 시 사용자에 의한 직접적인 Model 수정이 일어나지 않도록 조심한다
    2. ViewModel 에서 의도된 Intent 에 따라서 Model 수정
    3. Model 이 수정 됨에 따라 다시 위의 Model → ViewModel → View 이벤트 발생

![MVVM-Detail]({{site.baseurl}}/assets/img/mvvm-detail-flow.jpeg)

> **Simplicity and Declarability 를 지키기 위해 유용하다.**


## Reference 
[Stanford CS193P](https://youtu.be/--qKOhdgJAs)