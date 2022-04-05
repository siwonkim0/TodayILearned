## Observable

Observable는 Event를 전달한다.

Observer(구독자)는 Observable을 감시(구독, Subscribe)하고 있다가 전달되는 이벤트를 처리한다.

# Single

Observable의 변형으로,
~~Observable처럼 여러 값을 방출하지 않고 : next, error, completed~~
하나의 값이나 에러를 방출하는 애
그래서 subscribe할때도 
Observable처럼 onNext, onError, onCompleted가 아니라
onSuccess, onError로 받는다

### 프로젝트 코드

firebase에서 데이터를 fetch해올때 데이터를 가져오고 나서 single로 감싸서 리턴해준다

그럼 레포에서 single을 구독하고 있고, 전달된 데이터를 빼서 사용한다

```swift
class RemoteDataSource {
    let dataBase = Firestore.firestore()
    
    func fetch() -> Single<[Project]> {
        **return Single.create** **{** single in
            self.dataBase
                .collection("users")
                .getDocuments { snapshot, error in
                if let snapshot = snapshot, error == nil {
                    let projects = snapshot.documents.compactMap { document in
                        Project(document: document.data())
                    }
										//실제 이벤트를 내보내는 부분
                    single(.success(projects))
                }
            }
						//Disposable을 만들어 내보냅니다.
            **return Disposables.create()**
        **}**
    }
}
```

```swift
final class ProjectRepository: ProjectRepositoryProtocol {
    let disposeBag = DisposeBag()
    let remoteDataSource = RemoteDataSource()
    private lazy var projects = BehaviorRelay<[Project]>(value: [])
    
    func bindProjects() -> BehaviorRelay<[Project]> {
        remoteDataSource.fetch()
						.**subscribe(onSuccess:** { fetchedProjects in
            self.projects.accept(fetchedProjects)
        }).disposed(by: disposeBag)
        return projects
    }
}
```

# <Operator> Create
![](https://i.imgur.com/9nEyOTq.png)

[https://reactivex.io/documentation/operators/create.html](https://reactivex.io/documentation/operators/create.html)

뭐야 생각해보니 create 완전 기본 메서드잖아 .. ㅋ
옵저버블을 첨에 만들때 쓰는..

### 공식문서 샘플 코드

```swift
let source : Observable = Observable.**create** { observer in
    for i in 1...5 {
        observer.on(.next(i))
    }
    observer.on(.completed)

    // Note that this is optional. If you require no cleanup you can return
    // `Disposables.create()` (which returns the `NopDisposable` singleton)
    return Disposables.create {
        print("disposed")
    }
}

source.subscribe {
    print($0)
}
```

```swift
next(1)
next(2)
next(3)
next(4)
next(5)
completed
disposed
```

## Observable<T>.create안의 Disposables.create() 왜 하지?

### 일단 Observable<T>.create 먼저 보자.

사용 예시
```swift
let one = Observable<Int>.create { observer in
 	observer.onNext(10)
 	observer.onNext(20)
 	**return Disposables.create()**
 }
```

RxSwift 레포에 가서 코드를 까봤더니 
create는 ObservableType의 메서드이고, 
Observable은 ObservableType 프로토콜을 채택하고 있었음 😱

그냥 Observable의 메서드인줄 알었더니 사실은 ObservableType 꺼였군 ... 
![](https://i.imgur.com/OLBW9DR.png)


Observable의 create 메서드의 파라미터로 있는 
subscribe라는 escaping closure의 반환타입이 Disposable이라서 리턴을 해주는 것 같은데
의미 없는 Disposable 리턴은 대체 왜하는거지🧐 라는 생각이 들었다
리턴한 값을 대체 어디서 써먹는거야
AnonymousObservable이라는 타입의 이니셜라이저에 담아서 반환하네.. 일단 패스

### Disposables.create() 보자

Disposables 구조체를 확장해서 create 메서드를 다양하게 정의해놓은 것을 볼 수 있었다

파라미터가 있는것도 있고 없는것도 있네
![](https://i.imgur.com/PaVAFcT.png)

파라미터가 없는 create 메서드는 NopDisposable 타입을 리턴하고
있는 메서드는 AnonymousDisposable 을 리턴한다

![](https://i.imgur.com/Kje8kVR.png)

참고 링크
[https://jeonyeohun.tistory.com/365](https://jeonyeohun.tistory.com/365)
