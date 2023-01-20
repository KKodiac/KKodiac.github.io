---
layout: post
title: iOS에서 Clean Architecture 그리고 MVVM(1)
date: 2023-01-20 05:38:00 +0900
description: iOS 아키텍쳐에 대해서 알아보자. # Add post description (optional)
# img: redis.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Clean Architecture, MVVM, iOS]
---

# iOS에서 Clean Architecture 그리고 MVVM(1)

Created: November 28, 2022 2:40 PM
Tags: CleanArchitecture, iOS

우리가 소프트웨어를 개발할 땐, 디자인 패턴을 사용하는 것 만큼 중요한 것이 바로 아키텍쳐 패턴일 것이다. 소프트웨어 공학에는 많은 아키텍쳐 패턴이 존재한다. 여기서 모바일 소프트웨어 공학에서는 MVVM, Clean Architecture 그리고 Redux 패턴이 자주 응용된다. 

이 글에서는 [실제 응용된 예시 프로젝트](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master/ExampleMVVM)를 보여주며 iOS 앱에서 MVVM과 Clean Architecture라는 아키텍쳐 패턴들이 응용될 수 있는지 보여주겠다.

만약 Redux 패턴에 관심이 있다면 이 책을 확인해 보세요: **[Advanced iOS App Architecture](https://store.raywenderlich.com/products/advanced-ios-app-architecture)**.

더 많은 정보를 얻으려면 **[Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)** 여기를 참고하세요.

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/5.png)

위 Clean Architecture 그래프에서 볼 수 있듯, 어플리케이션 내에는 다양한 층이 존재합니다. 여기서 가장 중요한 규칙은 *내부 층과 외부 층 사이의 의존성이 존재해서는 안된다* 라는 것 입니다. 외부 → 내부로 향하는 화살표는 **[Dependency rule](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)(의존성 규칙)** 입니다. 외부에서 내부로 향하는 의존성만 존재할 수 있습니다. 

모든 층을 그룹으로 나누면: Presentation, Domain, 그리고 Data 층으로 나뉩니다. (위 사진의 빨간 부분)

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/4.png)

**Domain Layer (비지니스 로직)** 는 가장 내부에 있는 부분 입니다(다른 층에 의존성이 전혀 존재하지 않은 채로 완전히 고립되어 있습니다). *Entities(비지니스 모델), Use Cases, 그리고 Repository Interfaces* 로 이루어져 있습니다. 이 층은 의존성이 존재하지 않기에 독립적 이어서 여러 프로젝트 사이에 재사용 될 수 있습니다. 이러한 개체 사이에 분리는 Domain Use Cases 테스트 시 의존성이 필요 없기 때문에 몇 초 걸리지 않도록 합니다. **참고: Domain Layer 에는 다른 층에 개체가 존재해서는 안됩니다.***(예: Presentation - UIKit & SwiftUI / Data - Mappding Codable)*

좋은 아키텍쳐가 *Use Case*를 중심으로 하는 이유는 바로 설계 시 이러한 *Use Cases*를 지지하는 구조를 프레임워크, 도구, 환경(Framework, Tool, Environment)에  의존 없이 안전하게 정의할 수 있기 때문입니다. 이러한 정의는 **[Screaming Architecture](https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html)** 라고 합니다.

**Presentation Layer** 은 *****UI(UIViewControllers 또는 SwiftUI View)*를 포함합니다. View는 ViewModel(Presenters)로 인해 여러 Use Cases를 실행합니다. Presentation Layer는 **오직 Domain Layer 에만 의존합니다.**

**Data Layer** 는  저장소의 구현(Repository Implementations)과 한개 이상의 Data Sources를 포함합니다. 저장소(Repository)란 여러 Data Sources로 부터 오는 데이터를 조정하는 역할을 합니다. Data Source는 원격 또는 로컬(persistent database처럼)에 있을 수 있습니다. Data Layer는 **오직 Domain Layer에만 의존합니다.** 이 층에서는 네트워크의 JSON 데이터를 매핑(예: Decodable)하여 Domain 모델에 추가할 수 있습니다. 

아래에 있는 그래프 에서는 각 층의 모든 컴포넌트가 의존성 방향과 데이터 흐름에 따라 나타내져 있습니다. 여기서 우리는 저장소 인터페이스(Repository Interface, protocol)를 사용하는 **의존성 역위(Dependency Inversion)** 를 확인할 수 있습니다. 각 층의 설명은 [example project](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM) 를 기반으로 이루어 집니다. 

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/1.png)

**데이터의 흐름**

1. View(UI) 는 ViewModel(Presenter) 에 있는 메서드를 호출
2. ViewModel은 Use Case를 실행
3. Use Case는 User과 Repositories에 있는 데이터를 합침
4. 각 Repository는 원격 데이터(네트워크), Persistent DB 저장소, 또는 인-메모리 데이터(원격 또는 캐시)에 있는 데이터를 반환
5. 정보가 다시 View(UI)로 흘러 들어가고 반환된 값을 보여줌

**의존성 방향**

**Presentation Layer → Domain Layer ← Data Repositories Layer**

**Presentation Layer(MVVM)** = *ViewModels(Presenters) + Views(UI)*

**Domain Layer** = *Entities + Use Cases + Repositories Interfaces* 

**Data Repositories Layer** = *Repositories Implementations + API(Network) + Persistence DB*

### 예시 프로젝트: “Movies App”

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/2.png)

**Domain Layer**

예시 프로젝트 안에 [Domain Layer](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master/ExampleMVVM/Domain) 가 존재하는 것을 확인 할 수 있습니다. 이 층은 영화를 탐색하고 가장 최신 성공한 쿼리를 저장하는 [Entities](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master/ExampleMVVM/Domain/Entities), SearchMovies**************UseCase************** 를 포함하고 있습니다. 또한, 의존성 역위 (Dependency Inversion)을 위한 *Data Repositories Interfaces* 를 포함하고 있습니다. 

```swift
protocol SearchMoviesUseCase {
    func execute(requestValue: SearchMoviesUseCaseRequestValue,
                 completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable?
}

final class DefaultSearchMoviesUseCase: SearchMoviesUseCase {

    private let moviesRepository: MoviesRepository
    private let moviesQueriesRepository: MoviesQueriesRepository
    
    init(moviesRepository: MoviesRepository, moviesQueriesRepository: MoviesQueriesRepository) {
        self.moviesRepository = moviesRepository
        self.moviesQueriesRepository = moviesQueriesRepository
    }
    
    func execute(requestValue: SearchMoviesUseCaseRequestValue,
                 completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable? {
        return moviesRepository.fetchMoviesList(query: requestValue.query, page: requestValue.page) { result in
            
            if case .success = result {
                self.moviesQueriesRepository.saveRecentQuery(query: requestValue.query) { _ in }
            }

            completion(result)
        }
    }
}

// Repository Interfaces
protocol MoviesRepository {
    func fetchMoviesList(query: MovieQuery, page: Int, completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable?
}

protocol MoviesQueriesRepository {
    func fetchRecentsQueries(maxCount: Int, completion: @escaping (Result<[MovieQuery], Error>) -> Void)
    func saveRecentQuery(query: MovieQuery, completion: @escaping (Result<MovieQuery, Error>) -> Void)
}
```

**참고:** Use Cases 를 생성하는 다른 방식은 *start()* 함수를 사용한 *[UseCase](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Domain/UseCases/Protocol/UseCase.swift)* 프로토콜을 정의하고 모든 Use Cases 에 대해 해당 프로토콜을 구현하도록 할 수 있습니다. 예시 프로젝트에도 이러한 방식을 따른 예시가 있습니다: [FetchRecentMovieQueriesUseCase](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Domain/UseCases/FetchRecentMovieQueriesUseCase.swift). Use Cases 는 **Interactors** 라고 불리기도 합니다.

```swift
import Foundation

public protocol UseCase {
    @discardableResult
    func start() -> Cancellable?
}
```

```swift
final class FetchRecentMovieQueriesUseCase: UseCase {

    struct RequestValue {
        let maxCount: Int
    }
    typealias ResultValue = (Result<[MovieQuery], Error>)

    private let requestValue: RequestValue
    private let completion: (ResultValue) -> Void
    private let moviesQueriesRepository: MoviesQueriesRepository

    init(requestValue: RequestValue,
         completion: @escaping (ResultValue) -> Void,
         moviesQueriesRepository: MoviesQueriesRepository) {

        self.requestValue = requestValue
        self.completion = completion
        self.moviesQueriesRepository = moviesQueriesRepository
    }
    
    func start() -> Cancellable? {

        moviesQueriesRepository.fetchRecentsQueries(maxCount: requestValue.maxCount, completion: completion)
        return nil
    }
}
```

**참고:** *UseCase* 는 다른 *UseCase* 에 의존할 수 있습니다.

**Presentation Layer**

[Presentation Layer](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master/ExampleMVVM/Presentation) 는 MoviesList**View** 로부터 관측(observed) 될 수 있는 MoviesList**ViewModel** 내용을 포함합니다. MoviesList**ViewModel** 은 UIKit을 ****`import` 하지 않습니다. ViewModel을  UIKit, SwiftUI 또는 WatchKit 과 같은 UI 프레임워크로 부터 자유로워야 합니다. 재사용이 쉽고 리팩터하기 원활해지기 때문입니다. 예를 들어 ViewModel은 바뀔 필요가 없기 때문에 미래 UIKit에서 SwiftUI로 리팩터를 하는 것이 훨씬 쉬울 수 것입니다. 

```swift
// Note: We cannot have any UI frameworks(like UIKit or SwiftUI) imports here. 
protocol MoviesListViewModelInput {
    func didSearch(query: String)
    func didSelect(at indexPath: IndexPath)
}

protocol MoviesListViewModelOutput {
    var items: Observable<[MoviesListItemViewModel]> { get }
    var error: Observable<String> { get }
}

protocol MoviesListViewModel: MoviesListViewModelInput, MoviesListViewModelOutput { }

struct MoviesListViewModelActions {
    // Note: if you would need to edit movie inside Details screen and update this 
    // MoviesList screen with Updated movie then you would need this closure:
    //  showMovieDetails: (Movie, @escaping (_ updated: Movie) -> Void) -> Void
    let showMovieDetails: (Movie) -> Void
}

final class DefaultMoviesListViewModel: MoviesListViewModel {
    
    private let searchMoviesUseCase: SearchMoviesUseCase
    private let actions: MoviesListViewModelActions?
    
    private var movies: [Movie] = []
    
    // MARK: - OUTPUT
    let items: Observable<[MoviesListItemViewModel]> = Observable([])
    let error: Observable<String> = Observable("")
    
    init(searchMoviesUseCase: SearchMoviesUseCase,
         actions: MoviesListViewModelActions) {
        self.searchMoviesUseCase = searchMoviesUseCase
        self.actions = actions
    }
    
    private func load(movieQuery: MovieQuery) {
        
        searchMoviesUseCase.execute(movieQuery: movieQuery) { result in
            switch result {
            case .success(let moviesPage):
                // Note: We must map here from Domain Entities into Item View Models. Separation of Domain and View
                self.items.value += moviesPage.movies.map(MoviesListItemViewModel.init)
                self.movies += moviesPage.movies
            case .failure:
                self.error.value = NSLocalizedString("Failed loading movies", comment: "")
            }
        }
    }
}

// MARK: - INPUT. View event methods
extension MoviesListViewModel {
    
    func didSearch(query: String) {
        load(movieQuery: MovieQuery(query: query))
    }
    
    func didSelect(at indexPath: IndexPath) {
        actions?.showMovieDetails(movies[indexPath.row])
    }
}

// Note: This item view model is to display data and does not contain any domain model to prevent views accessing it
struct MoviesListItemViewModel: Equatable {
    let title: String
}

extension MoviesListItemViewModel {
    init(movie: Movie) {
        self.title = movie.title ?? ""
    }
}
```

**참고:** MoviesListView
Controller의 테스트성을 위해(*[example](https://github.com/kudoleh/iOS-Modular-Architecture/blob/master/DevPods/MoviesSearch/MoviesSearch/Tests/Presentation/MoviesScene/MoviesListViewTests.swift)*
)MoviesList**ViewModelInput** 그리고 MoviesList**ViewModelOutput** interface를 사용합니다. 또한 [MoviesSearch**FlowCoordinator](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Presentation/MoviesScene/Flows/MoviesSearchFlowCoordinator.swift)** 에게 언제 다른 view를 보여줘야 하는지 알려주는 MoviesListViewModel**Actions** 클로져도 존재합니다. 해당 클로져가 호출이되면 coordinator가 화면에 영화 세부정보를 보여주게 됩니다. `MoviesListViewModelActions` 에 `struct` 를 사용하는 이유는 추후 더 추가될 수 있는 action을 모아놓기 위해서 입니다. 

[Presentation Layer](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master/ExampleMVVM/Presentation)에는 MoviesList**ViewModel**에 있는 데이터에 바인딩된 MoviesListViewController 가 포함되어 있습니다.

UI는 비지니스 로직 또는 어플리케이션 로직(비지니스 모델 그리고 UsesCases)에 대한 권한이 부여되면 안됩니다. 오직 ViewModel이 갖고 있어야 하죠. 이것을 **Separation of Concerns** 라고 부릅니다. 우리는 직접 View(UI) 에 비지니스 모델을 보낼 수 없습니다. 이것이 우리가 비지니스 모델을 ViewModel 안에 매핑하고 해당 ViewModel을 View에 전달하는 이유 입니다. 

우리는 View에서 ViewModel 로 가는 검색 이벤트 호출을 추가했습니다.

```swift
import UIKit

final class MoviesListViewController: UIViewController, StoryboardInstantiable, UISearchBarDelegate {
    
    private var viewModel: MoviesListViewModel!
    
    final class func create(with viewModel: MoviesListViewModel) -> MoviesListViewController {
        let vc = MoviesListViewController.instantiateViewController()
        vc.viewModel = viewModel
        return vc
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        bind(to: viewModel)
    }
    
    private func bind(to viewModel: MoviesListViewModel) {
        viewModel.items.observe(on: self) { [weak self] items in
            self?.moviesTableViewController?.items = items
        }
        viewModel.error.observe(on: self) { [weak self] error in
            self?.showError(error)
        }
    }
    
    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        guard let searchText = searchBar.text, !searchText.isEmpty else { return }
        viewModel.didSearch(query: searchText)
    }
}
```

**참고:** 우리는 items 를 관찰(Observe) 하고 해당 item 이 바뀌면 view를 reload합니다. 여기선 간단한 *[Observable](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Presentation/Utils/Observable.swift)* 을 사용 했으며, 아래에 있는 MVVM 부분에서 설명 됩니다. 

우리는 또한 *howMovieDetails*(movie:) 함수를 MoviesListViewModel의 MoviesListViewModelActions 안에 할당 했습니다. 이것은 영화 세부정보에 대한 화면을 flow coordinator에서 부터 보여주기 위함입니다. 

```swift
protocol MoviesSearchFlowCoordinatorDependencies  {
    func makeMoviesListViewController() -> UIViewController
    func makeMoviesDetailsViewController(movie: Movie) -> UIViewController
}

final class MoviesSearchFlowCoordinator {
    
    private weak var navigationController: UINavigationController?
    private let dependencies: MoviesSearchFlowCoordinatorDependencies

    init(navigationController: UINavigationController,
         dependencies: MoviesSearchFlowCoordinatorDependencies) {
        self.navigationController = navigationController
        self.dependencies = dependencies
    }
    
    func start() {
        // Note: here we keep strong reference with actions closures, this way this flow do not need to be strong referenced
        let actions = MoviesListViewModelActions(showMovieDetails: showMovieDetails)
        let vc = dependencies.makeMoviesListViewController(actions: actions)
        
        navigationController?.pushViewController(vc, animated: false)
    }
    
    private func showMovieDetails(movie: Movie) {
        let vc = dependencies.makeMoviesDetailsViewController(movie: movie)
        navigationController?.pushViewController(vc, animated: true)
    }
}
```

 **참고:** 우리는 ViewController의 사이즈와 역할을 축소하기 위해 presentaion logic 으로 Flow Coordinator 를 사용했습니다. Flow Coordinator 에 대해 강한 참조(strong reference) 를 사용해(action closure, self 함수를 사용) 필요 시 비할당 되지 않도록 합니다. 

이러한 방식으로 우리는 같은 ViewModel 을 변형하지 않고 여러 View 에 적용해 사용할 수 있습니다. 우리는 iOS 13.0을 사용할 수 있는지를 확인하고 SwiftUI의 View를 생성해 UIKit 대신 사용할 수 있으며, 같은 ViewModel에 바인딩 해 응용할 수 있습니다. https://github.com/kudoleh/iOS-Clean-Architecture-MVVM 에서는 SwiftUI 예시를 추가해 MovidesQueriesSuggestionsList 를 만들었습니다. 최소 Xcode 11 베타 버전이 필요합니다. 

```swift
// MARK: - Movies Queries Suggestions List
func makeMoviesQueriesSuggestionsListViewController(didSelect: @escaping MoviesQueryListViewModelDidSelectAction) -> UIViewController {
   if #available(iOS 13.0, *) { // SwiftUI
       let view = MoviesQueryListView(viewModelWrapper: makeMoviesQueryListViewModelWrapper(didSelect: didSelect))
       return UIHostingController(rootView: view)
   } else { // UIKit
       return MoviesQueriesTableViewController.create(with: makeMoviesQueryListViewModel(didSelect: didSelect))
   }
}
```

**Data Layer**

[Data Layer](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master/ExampleMVVM/Data) 는 DefaultMoviesRepository 를 포함합니다. Domain Layer 안에 있는 interfaces 들을 따르게 합니다(의존성 역위, Dependency Inversion).  우리는 또한 JSON 데이터([Decodable conformance](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Data/Network/DataMapping/MoviesResponseDTO%2BMapping.swift)
) 그리고 [CoreData Entities](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Data/PersistentStorages/MoviesResponseStorage/EntityMapping/MoviesResponseEntity%2BMapping.swift) 매핑을 추가 했습니다.

```swift
final class DefaultMoviesRepository {
    
    private let dataTransferService: DataTransfer
    
    init(dataTransferService: DataTransfer) {
        self.dataTransferService = dataTransferService
    }
}

extension DefaultMoviesRepository: MoviesRepository {
    
    public func fetchMoviesList(query: MovieQuery, page: Int, completion: @escaping (Result<MoviesPage, Error>) -> Void) -> Cancellable? {
        
        let endpoint = APIEndpoints.getMovies(with: MoviesRequestDTO(query: query.query,
                                                                     page: page))
        return dataTransferService.request(with: endpoint) { (response: Result<MoviesResponseDTO, Error>) in
            switch response {
            case .success(let moviesResponseDTO):
                completion(.success(moviesResponseDTO.toDomain()))
            case .failure(let error):
                completion(.failure(error))
            }
        }
    }
}

// MARK: - Data Transfer Object (DTO)
// It is used as intermediate object to encode/decode JSON response into domain, inside DataTransferService
struct MoviesRequestDTO: Encodable {
    let query: String
    let page: Int
}

struct MoviesResponseDTO: Decodable {
    private enum CodingKeys: String, CodingKey {
        case page
        case totalPages = "total_pages"
        case movies = "results"
    }
    let page: Int
    let totalPages: Int
    let movies: [MovieDTO]
}
...
// MARK: - Mappings to Domain
extension MoviesResponseDTO {
    func toDomain() -> MoviesPage {
        return .init(page: page,
                     totalPages: totalPages,
                     movies: movies.map { $0.toDomain() })
    }
}
```

**참고:** Data Transfer Object(DTO) 는 JSON 응답을 Domain으로 매핑하기 위한 중간 객체로 사용됩니다. 그리고 만약 우리가 endpoint 응답을 캐싱하고 싶다면 DTO를 Persistent Object로 매핑해 persistent storage에 저장할 수 있습니다(예: DTO → NSManagedObject). 

일반적으로 API Data Service나 Persistent Data Storage를 Data Repositories에 주입할 수 있습니다. Data Repository는 이 두가지 의존(dependencies)를 통해 데이터를 반환 합니다. 규칙은 우선 persistent storage에게 캐싱된 데이터를 요청하고(DTO 객체를 통해 Domain에 NSManagedObject가 매핑되며, cached data closure에 저장됩니다.) 그후 API Data Service를 호출해 가장 최신 데이터를 가져옵니다. 다음으로 Persistent Storage를 해당 최신 데이터로 업데이트 합니다(DTO는 Persistent Objects로 매핑되며 저장됩니다.). 그리고 DTO는 Domain에 매핑 되어 updated data/completion closure 안에 반환 됩니다. 이러한 방식으로 사용자는 데이터를 즉시 확인할 수 있습니다. 인터넷 연결이 없더라도, 사용자는 가장 최신 데이터를 Persistent Storage로 부터 받을 수 있습니다. *[example](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/blob/master/ExampleMVVM/Data/Repositories/DefaultMoviesRepository.swift)*

![Untitled]({{site.baseurl}}/assets/img/CAMVVM/3.png)

Persistent Storage와 API Data Service는 아에 다른 구현으로도 변경될 수 있습니다(예: CoreData → Realm).

[**Infrastructure Layer (Network)**](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master/ExampleMVVM/Infrastructure/Network)

이것은 Alamofire과 같은 네트워크 프레임워크의 wrapper 역할을 합니다. Base URL과 같은 네트워크 매개변수로도 설정될 수 있습니다. 또한 endpoint를 정의하거나 Decodable을 사용한 데이터 매핑 메서드를 포함합니다.

```swift
struct APIEndpoints {
    
    static func getMovies(with moviesRequestDTO: MoviesRequestDTO) -> Endpoint<MoviesResponseDTO> {

        return Endpoint(path: "search/movie/",
                        method: .get,
                        queryParametersEncodable: moviesRequestDTO)
    }
}

let config = ApiDataNetworkConfig(baseURL: URL(string: appConfigurations.apiBaseURL)!,
                                  queryParameters: ["api_key": appConfigurations.apiKey])
let apiDataNetwork = DefaultNetworkService(session: URLSession.shared,
                                           config: config)

let endpoint = APIEndpoints.getMovies(with: MoviesRequestDTO(query: query.query,
                                                             page: page))
dataTransferService.request(with: endpoint) { (response: Result<MoviesResponseDTO, Error>) in
    let moviesPage = try? response.get()
}
```

**참고:** You can read more here: [https://github.com/kudoleh/SENetworking](https://github.com/kudoleh/SENetworking)

## References

- [https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3](https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3)

[Clean Coder Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
