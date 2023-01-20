---
layout: post
title: iOS에서 Clean Architecture 그리고 MVVM(2)
date: 2023-01-20 05:38:00 +0900
description: iOS 아키텍쳐에 대해서 알아보자. # Add post description (optional)
# img: redis.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Clean Architecture, MVVM, iOS]
---
# iOS에서 Clean Architecture 그리고 MVVM(2)

Created: January 19, 2023 12:05 AM
Tags: CleanArchitecture, iOS

## **MVVM**

**Model-View-ViewModel** 패턴 (MVVM)은 UI와 Domain 사이에 깔끔한 Separation of Concerns를 제공합니다. 

Clean Architecture와 함께 사용되면 Presentation 과 UI Layer 사이에 Separation of Concern을 도울 수 있습니다. 

같은 ViewModel은 여러 view 구현에 사용할 수 있습니다. 예를 들어 CarsAround**ViewModel**을 CarsAroundList**View**와 CarsAroundMap**View**에 사용 될 수 있습니다. 또한 당신은 View를 UIKit으로 하나, SwiftUI으로 하나 구현할 수 있습니다. 여기서 꼭 기억할 점은 ViewModel 안에 UIKit, SwiftUI, WatchKit을 `import` 하면 안된다는 점입니다. 다른 플랫폼으로 구현할 때에 재사용성을 높이기 위함입니다. 

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/6.png)

**View**와 **ViewModel** 사이에 **Data Binding**은 클로저, 위임자(delegate), 또는 관찰자(observable)를 사용할 수 있습니다. Combine과 SwiftUI는 iOS 버전 13이상을 지원할 수 있다면 사용할 수 있습니다. **View**는 **ViewModel**과 직접적인 관련이 있기 때문에 View안에서 이벤트가 일어날 시 알립니다. ViewModel에서는 View로 향하는 직접적인 참조는 일어나지 않습니다.(오직 바인딩만 있습니다.)

이 예시에서는 간단한 클로저와 `didSet`을 이용한 구현을 볼 수 있습니다. 

```swift
public final class Observable<Value> {
    
    private var closure: ((Value) -> ())?

    public var value: Value {
        didSet { closure?(value) }
    }

    public init(_ value: Value) {
        self.value = value
    }

    public func observe(_ closure: @escaping (Value) -> Void) {
        self.closure = closure
        closure(value)
    }
}
```

**참고:** Observable에 대한 매우 간단한 예시 입니다. 모든 구현 내용을 확인하려면: **[Observable](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Presentation/Utils/Observable.swift)**을 확인하세요. 관찰자의 호출은 UI를 포함한 Presentation Layer에서 사용되기 때문에 **main thread**에서 작동합니다.

ViewController에서의 데이터 바인딩 예시:

```swift
final class ExampleViewController: UIViewController {
    
    private var viewModel: MoviesListViewModel!
    
    private func bind(to viewModel: ViewModel) {
        self.viewModel = viewModel
        viewModel.items.observe(on: self) { [weak self] items in
            self?.tableViewController?.items = items
            // Important: You cannot use viewModel inside this closure, it will cause retain cycle memory leak (viewModel.items.value not allowed)
            // self?.tableViewController?.items = viewModel.items.value // This would be retain cycle. You can access viewModel only with self?.viewModel
        }
        // Or in one line
        viewModel.items.observe(on: self) { [weak self] in self?.tableViewController?.items = $0 }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        bind(to: viewModel)
        viewModel.viewDidLoad()
    }
}

protocol ViewModelInput {
    func viewDidLoad()
}

protocol ViewModelOutput {
    var items: Observable<[ItemViewModel]> { get }
}

protocol ViewModel: ViewModelInput, ViewModelOutput {}
```

**참고:** 관찰자 클로저 안에서 ViewModel을 엑세스 하면 retain cycle이 생기기 때문에 메모리 누수가 발생합니다. 오직 `self?.viewModel` 의 방식으로 ViewModel을 다뤄야 합니다. 

## **MVVM 사이의 통신**

### Delegation

MVVM 내에서 ViewModel 사이의 통신은 위임자 패턴(Delegation Pattern)을 사용하면 가능합니다. 

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/7.png)

ItemsList**ViewModel와** ItemEdit**ViewModel**, 이 두개의 ViewModel을 예로 들어보겠습니다. 우리는 ItemEdit**ViewModelDelegate**이라는 프로토콜을 정의하고 안에ItemEditViewModelDidEditItem(item) 메서드를 정의할 수 있도록 합니다. 그리고 해당 프로토콜에 conform하게 만듭니다: `extension ListItemsViewModel: ItemEditViewModel**Delegate`** 

```swift
// Step 1: Define delegate and add it to first ViewModel as weak property
protocol MoviesQueryListViewModelDelegate: class {
    func moviesQueriesListDidSelect(movieQuery: MovieQuery)
}
...
final class DefaultMoviesQueryListViewModel: MoviesListViewModel {
    private weak var delegate: MoviesQueryListViewModelDelegate?
    
    func didSelect(item: MoviesQueryListViewItemModel) { 
        // Note: We have to map here from View Item Model to Domain Enity
        delegate?.moviesQueriesListDidSelect(movieQuery: MovieQuery(query: item.query))
    }
}

// Step 2:  Make second ViewModel to conform to this delegate
extension MoviesListViewModel: MoviesQueryListViewModelDelegate {
    func moviesQueriesListDidSelect(movieQuery: MovieQuery) {
        update(movieQuery: movieQuery)
    }
}

```

**참고:** 우리는 이러한 경우 Delegate을 Responder로 명명할 수도 있습니다: ItemEditViewModel**Responder**

### Closures

또 다른 방식은 FlowCoordinator로부터 주입받은 클로저를 사용하는 경우 입니다. 예시 프로젝트에서는 [MoviesListViewModel](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Presentation/MoviesScene/MoviesList/ViewModel/MoviesListViewModel.swift)가 MoviesQueriesSuggestions**View 보여주기 위해** *showMovieQueriesSuggestions *****action 클로저를 사용합니다. 또한 매개변수 *(**_**didSelect: MovieQuery) -> Void* 를 넘겨 View 자체로 부터 다시 호출 받을 수 있도록 합니다. 해당 예시는  

[MoviesSearch**FlowCoordinator**](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Presentation/MoviesScene/Flows/MoviesSearchFlowCoordinator.swift): 에서 확인할 수 있습니다.

```swift
// MoviesQueryList.swift
// Step 1: Define action closure to communicate to another ViewModel, e.g. here we notify MovieList when query is selected
typealias MoviesQueryListViewModelDidSelectAction = (MovieQuery) -> Void

// Step 2: Call action closure when needed
class MoviesQueryListViewModel {
    init(didSelect: MoviesQueryListViewModelDidSelectAction? = nil) {
        self.didSelect = didSelect
    }
    func didSelect(item: MoviesQueryListItemViewModel) {
        didSelect?(MovieQuery(query: item.query))
    }
}

// MoviesQueryList.swift
// Step 3: When presenting MoviesQueryListView we need to pass this action closure as paramter (_ didSelect: MovieQuery) -> Void
struct MoviesListViewModelActions {
    let showMovieQueriesSuggestions: (@escaping (_ didSelect: MovieQuery) -> Void) -> Void
}

class MoviesListViewModel { 
    var actions: MoviesListViewModelActions?

    func showQueriesSuggestions() {
        actions?.showMovieQueriesSuggestions { self.update(movieQuery: $0) } 
        //or simpler actions?.showMovieQueriesSuggestions(update)
    }
}

// FlowCoordinator.swift
// Step 4: Inside FlowCoordinator we connect communication of two viewModels, by injecting actions closures as self function
class MoviesSearchFlowCoordinator {
    func start() {
        let actions = MoviesListViewModelActions(showMovieQueriesSuggestions: self.showMovieQueriesSuggestions)
        let vc = dependencies.makeMoviesListViewController(actions: actions)  
        present(vc)
    }

    private func showMovieQueriesSuggestions(didSelect: @escaping (MovieQuery) -> Void) {
        let vc = dependencies.makeMoviesQueriesSuggestionsListViewController(didSelect: didSelect)
        present(vc)
    }
}
```

## ****Layer Separation into frameworks (Modules)****

이제 각 층(Domain, Presentation, UI, Data, Infrastructure Network)은 각자의 프레임워크로 쉽게 나뉠 수 있습니다.

```swift
New Project -> Create Project… -> Cocoa Touch Framework
```

그후 각각의 프레임워크를 CocoaPods 같은 툴로 main app에 추가할 수 있습니다. [working example here](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/MVVM%20Modular%20Layers%20Pods.zip)을 참고해 주세요. 

**참고:** 권한 문제로 **ExampleMVVM.xcworkspace**을 삭제한 후 `pod install` 을 실행해 새로운 놈을 생성해야 합니다.

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/8.png)

## ****Dependency Injection Container(DIC)****

Dependency Injection(의존성 주입)은 어느 한 객체가 다른 객체의 의존성을 공급해 주는 역할을 하는 것을 말합니다. [DIContainer](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Application/DIContainer/AppDIContainer.swift) 는 해당 어플리케이션에서 모든 의존성 주입의 중심 역할을 합니다. 

### Factory 프로토콜 의존성 사용하기

하나의 방법은 의존성 프로토콜(Dependency Protocol)을 정의해서 DIContainer로 가는 의존성의 생성을 위임하는 것입니다. 이렇게 하기 위해서는 MoviesSearchFlowCoordinator**Dependencies**
프로토콜을 정의하고 MoviesScene**DIContainer**가 해당 프로토콜에 conform하게 만들고, MoviesList**View**Controller를 만들고 보여주는 MoviesSearch**FlowCoordinator**에 주입하는 것입니다. 

```swift
// Define Dependencies protocol for class or struct that needs it
protocol MoviesSearchFlowCoordinatorDependencies  {
    func makeMoviesListViewController() -> MoviesListViewController
}

class MoviesSearchFlowCoordinator {
    
    private let dependencies: MoviesSearchFlowCoordinatorDependencies

    init(dependencies: MoviesSearchFlowCoordinatorDependencies) {
        self.dependencies = dependencies
    }
...
}

// Make the DIContainer to conform to this protocol
extension MoviesSceneDIContainer: MoviesSearchFlowCoordinatorDependencies {}

// And inject MoviesSceneDIContainer `self` into class that needs it
final class MoviesSceneDIContainer {
    ...
    // MARK: - Flow Coordinators
    func makeMoviesSearchFlowCoordinator(navigationController: UINavigationController) -> MoviesSearchFlowCoordinator {
        return MoviesSearchFlowCoordinator(navigationController: navigationController,
                                           dependencies: self)
    }
}
```

### 클로저 사용하기

다른 방법은 클로저를 사용하는 법입니다. 하는 방법은 의존성 주입이 필요한 class에 클로저를 정의하고 클로저를 주입합니다. 예를 들어:

```swift
// Define makeMoviesListViewController closure that returns MoviesListViewController
class MoviesSearchFlowCoordinator {
   
    private var makeMoviesListViewController: () -> MoviesListViewController

    init(navigationController: UINavigationController,
         makeMoviesListViewController: @escaping () -> MoviesListViewController) {
        ...
        self.makeMoviesListViewController = makeMoviesListViewController
    }
    ...
}

// And inject MoviesSceneDIContainer's `self`.makeMoviesListViewController function into class that needs it
final class MoviesSceneDIContainer {
    ...
    // MARK: - Flow Coordinators
    func makeMoviesSearchFlowCoordinator(navigationController: UINavigationController) -> MoviesSearchFlowCoordinator {
        return MoviesSearchFlowCoordinator(navigationController: navigationController,
                                           makeMoviesListViewController: self.makeMoviesListViewController)
    }
    
    // MARK: - Movies List
    func makeMoviesListViewController() -> MoviesListViewController {
        ...
    }
}
```

## 소스 코드

[https://github.com/kudoleh/iOS-Clean-Architecture-MVVM](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM)

## 참고 문헌

*[Advanced iOS App Architecture](https://store.raywenderlich.com/products/advanced-ios-app-architecture)*

*[The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)*

*[The Clean Code](https://www.amazon.de/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)마*

# 마무리

대부분의 모바일 개발에서 사용되는 아키텍쳐 패턴은 Clean Architecture(Layered), MVVM, 그리고Redux 입니다. 당연히 MVVM과 Clean Architecture은 따로 사용될 수 있지만, MVVM은 오직 Presentation Layer 사이에만 Separation of Concern을 제공하고, Clean Architecture은 여러 모듈 층으로 프로젝트를 나누어 테스트성, 재사용성과 전체 설계에 이해가 쉽습니다. 

Use Case가 오직 Repository를 호출하는 역할로 밖에 쓰이지 않지만, Use Case의 생성을 무시하지 않는 것이 중요합니다. 다른 개발자가 Use Case를 봤을 때 무엇을 하는 지 알게하기 위해선 이것이 필요합니다. 

이 글이 시작점으로 참고하는 것엔 도움이 될 수 있으나, 이게 무조건 맞다는 것이 아닙니다. 각 프로젝트의 필요한 아키텍쳐를 적용하는 것을 목표로 하는 것이 중요합니다. 

Clean Architecture는 TDD와 용이하게 작동합니다. 이 아키텍쳐는 프로젝트의 테스트성을 높이며 각 층이 분리되어 UI와 데이터를 각각 테스트할 수 있도록 돕습니다. 

CA는 [Domain-Driven Design](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) (DDD)와도 잘 어울립니다.

소프트웨어 개발에서 더 다양한 아키텍쳐를 알고 싶으면: **[The 5 Patterns You Need to Know](https://dzone.com/articles/software-architecture-the-5-patterns-you-need-to-k)**

*좋은 개발 습관:*

*테스트 없는 코드는 작성하지 말라(TDD하자!)*

*연속적인 리팩터링*

*오버하지 마라, 적당한 설계*

*최대한 서드파티 프레임워크 의존성을 줄여라*
