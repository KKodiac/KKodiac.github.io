---
layout: post
title: Clean Architecture for SwiftUI(1)
date: 2022-03-16 05:38:00 +0900
description: SwiftUI 를 위한 클린 아키텍쳐  # Add post description (optional)
img: swift-logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Swift, SwiftUI, iOS, MVVM, CleanArchitecture]
---

# Clean Architecture for SwiftUI
IOS 개발에 대해서 전반적으로 알아보던 중 토이 프로젝트를 진행하기로 마음 먹었다. 일단 stacklounge 의 백엔드를 활용해 보고자 해서 기존 Android 로 개발 했었던 기능들을 그대로 IOS 의 SwiftUI 로 구현해보자는 생각으로 시작했다. 

하지만 맨땅의 헤딩 식으로 진행 하기보다는 조금 더 Swift 라는 언어와 애플에서 최근 UIKit을 뒤로 하고 새로 밀고 있는 SwiftUI 라는 IOS Framework 에 대해서 조금 더 알아보기로 했다. 저번 글에서 SwiftUI 가 사용하는 MVVM 구조에 대해서 조금 알아보았지만, 그것을 적용한 SwiftUI 가 추구하는 클린 아키텍쳐에 대해서 조금 더 알아보다 찾은 [블로그](https://nalexn.github.io/clean-architecture-swiftui/) 를 참고해 공부한 내용을 적어보도록 하겠다. 

한글 번역을 진행한 [블로그](https://gon125.github.io/posts/SwiftUI를-위한-클린-아키텍처/)도 있다. 


## SwiftUI == MVVM?
SwiftUI 는 MVVM 구조를 기반으로 설계 되었다. MVVM 구조는 프레젠테이션 계층을 담당하는 View, 데이터 액세스를 담당하는 Model 과 그 사이 비지니스 로직을 담당하는 ModelView 간의 연결을 의미한다.

소프트웨어공학을 다룬 수업 시간에 `결합도는 낮게, 응집도는 높게` 프로그램의 요소들을 구성해야 유지보수성이 높은 소프트웨어를 만들 수 있다고 배운 것이 생각이 났다. MVVM 처럼 구성하면
[`Presentation`] - [`Business Logic`] - [`Data Access`] 이 세개의 층을 결합도가 낮게 설계 할 수 있다. 

SwiftUI에서는 간단한 상황에서는 `@State`을, 더 복잡한 상황에서는 `ObservableObject`를 View가 참조하여 ViewModel의 개념으로 사용하게 된다. 해당 ModelView 값에 변화가 일어날 시 이것을 View에서 `@ObservedObject`로 계속 주시하고 있다가 ModelView에서 `@Published` 호출이 왔을 때 View를 해당 데이터로 업데이트 하게 된다. 

참조하고 있는 블로그에 있는 코드를 예시로 들어보겠다. 

**Model**: a data container

```swift
struct Country {
    let name: String
}
```

**View**: a SwiftUI view

```swift
struct CountriesList: View {
    
    @ObservedObject var viewModel: ViewModel
    
    var body: some View {
        List(viewModel.countries) { country in
            Text(country.name)
        }
        .onAppear {
            self.viewModel.loadCountries()
        }
    }
}
```

**ViewModel**: `ObservableObject`로 비지니스 로직을 캡슐화 시키고 View가 해당 변수의 상태를 관찰할 수 있도록 돕는다

```swift
extension CountriesList {
    class ViewModel: ObservableObject {
        @Published private(set) var countries: [Country] = []
        
        private let service: WebService
        
        func loadCountries() {
            service.getCountries { [weak self] result in
                self?.countries = result.value ?? []
            }
        }
    }
}
```

위 코드의 동작을 다이어그램으로 표시해 보았다. 데이터 액세스는 서비스인 WebService에서 호출되는 외부 API 저장소겠지만 Model 코드로 대체했다.

![MVVM Example]({{site.baseurl}}/assets/img/mvvm-example1.jpg)

 - [1] `ViewModel`에 있는 `loadCountries()` 함수를 호출한다. 
 - [2]~[3] service.getCountries 로 외부 API 호출 및 데이터를 저장한다. 여기서 `self?.countries`를 호출 결과로 가져오는 값 또는 널 값으로 지정해주게 되는데, 이렇게 데이터의 변화가 일어난다.
 - [4] `View`에서는 지속적으로 `@ObsesrvedObject`로 지정된 객체를 관찰하고 있는데, [3]으로 인해 `countries` 변수의 값이 업데이트 되고 `ViewModel`은 `@Published countries`라는 변수를 통해 업데이트를 `View`에 알리게 된다. 
 - [5] 변경된 내용이 해당되는 `View`만 업데이트 하여 보여주게 된다.

 ## 마치며 
 아직 커버할 내용이 많은 것은 알고 있다. 아직도 너무 많은 것들을 모르고 있어서 조바심이 많이나지만 이번 글의 내용을 정리하면서  기본적으로 SwiftUI가 MVVM 구조를 통해 작동하는 법은 이해한 것 같다. 다음 글에는 참조글에서 얘기하는 `AppState`, `View`, `Interactor`, `Repository`의 정의와 내용을 알아보면서 더 자세한 SwiftUI 내에서의 클린 아키텍쳐를 정리해보겠다.

 ## Reference Article
 [Clean Architecture for SwiftUI](https://nalexn.github.io/clean-architecture-swiftui/) By Alexey Naumov