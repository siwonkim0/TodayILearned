# Traits

Observable을 감싸고 있는 구조체임
`.asObservable()` 사용시 다시 기본 Observable로 돌아갈 수 있음 
- Single
- Completable
- Maybe

# Single

`단 하나의 요소`나 `error`만 방출하는 Observable의 변형인 Traits중 하나이다.

- 언제 사용하냐?
보통 HTTP 요청의 결과로 response와 error를 딱 리턴하므로 많이 쓰임

- 특징
    - 딱 하나의 요소나 error만 방출
    - side effects를 공유하지 않음(?)
    - Observable에 `.asSingle()` 사용시 Single로 변환 가능
### 1. Single.create

```swift
func fetch() -> Single<[Project]> {
    var projects = [Project]()
    return Single.create { single in
        self.realm.objects(ProjectRealm.self).forEach { projectRealm in
            let project = Project(projectRealm: projectRealm)
            projects.append(project)
        }
        single(.success(projects))
        return Disposables.create()
    }
}
```

```swift
func getRepo(_ repo: String) -> Single<[String: Any]> {
    return Single<[String: Any]>.create { single in
        let task = URLSession.shared.dataTask(with: URL(string: "https://api.github.com/repos/\(repo)")!) { data, _, error in
            if let error = error {
                single(.error(error))
                return
            }

            guard let data = data,
                  let json = try? JSONSerialization.jsonObject(with: data, options: .mutableLeaves),
                  let result = json as? [String: Any] else {
                single(.error(DataError.cantParseJSON))
                return
            }

            single(.success(result))
        }

        task.resume()

        return Disposables.create { task.cancel() }
    }
}

getRepo("ReactiveX/RxSwift")
    .subscribe { event in
        switch event {
            case .success(let json):
                print("JSON: ", json)
            case .error(let error):
                print("Error: ", error)
        }
    }
    .disposed(by: disposeBag)
```

# Completable

`completed` 나 `error`만 방출하는 Observable의 변형인 Traits중 하나이다.

- 언제 사용하냐?
특정 동작이 끝났는지 여부만 관심이 있고, 그 동작으로 인해 일어난 결과물에는 관심이 없을 때 사용한다.

ex) Firebase에 데이터를 업데이트하고 업데이트 완료했다고 completable로 알려주면 
구독하고 있는 곳에서 데이터 업데이트가 되고 나서 수행해야할 동작을 수행할 수 있음

- 특징
    - 요소를 방출하지 않음
    - side effect를 공유하지 않음(무슨말이지?)

### 1. Completable.create

Observable.create하는거랑 같은 방법이다.
똑같이 create의 리턴 타입이 Disposable이라서 `return` Disposables.create() 해야함 

### 프로젝트 적용 예시
```swift
//Completable.create
func delete(_ project: Project) -> Completable {
    return Completable.create { completable in
        self.dataBase
            .collection("users")
            .document(project.id.description)
            .delete()
        completable(.completed)
        return Disposables.create()
    }
}

//Completable 사용할 때는 subscribe해서 사용
Completable.zip(
    remoteDataSource.delete(project),
    localDataSource.delete(project)
).subscribe(onCompleted: { [self] in
    var currentProjects = projects.value
    if let row = currentProjects.firstIndex(where: { $0.id == project.id }) {
        currentProjects.remove(at: row)
    }
    projects.accept(currentProjects)
}).disposed(by: disposeBag)
```
local, remote datasource의 delete 메서드를 실행하는 곳(repo)에서 Completable.zip으로 둘을 묶어서 구독하여, 두 작업 모두가 끝난 후에 바로 모델 업데이트할 수 있게 구현한 예시이다.

### 2. Completable.zip

Completable들이 배열에 담긴: [Completable]을 하나의 Completable 형태로 바꿔주는 메서드 zip
![](https://velog.velcdn.com/images/dev_jane/post/febe0599-0119-417f-9b16-d875cc98b107/image.png)

### 프로젝트 적용 예시
**1st. [Project] → [Completable]**

- intersectingRemoteProjects: [Project]
- intersectingLocalProjects: [Project]

배열의 뎁스는 유지하면서 filter의 조건으로 localProject, remoteProject 둘다를 사용하고 싶어서

이렇게 flatMap과 filter을 겹쳐서 사용함 

```swift
let sameIDCompletable: [Project] = intersectingRemoteProjects.flatMap { remoteProject in
        intersectingLocalProjects.filter { localProject in
            localProject.updatedAt > remoteProject.updatedAt
        }
}
```

여기서 .map 를 해서 배열 안의 Project 타입을 Completable로 바꾼 것임 
self.remoteDataSource.update($0)의 리턴값이 Completable

```swift
let sameIDCompletable: [Completable] =
      intersectingRemoteProjects.flatMap { remoteProject in
          intersectingLocalProjects.filter { localProject in
              localProject.updatedAt > remoteProject.updatedAt
          }.map {
              return self.remoteDataSource.update($0)
          }
      }
```

- 만약 flatMap이 아니라 map을 썼다면 [[Completable]] 타입이었을 것...
    
    ```swift
    let sameIDCompletable: [[Completable]] =
        intersectingRemoteProjects.map { remoteProject in
            intersectingLocalProjects.filter { localProject in
                localProject.updatedAt > remoteProject.updatedAt
            }.map {
                return self.remoteDataSource.update($0)
            }
        }
    ```
    

**2nd. [Completable] → Completable**
Completable.zip을 사용하여 최종적으로 [Completable] 타입에서 Completable 타입으로 변경해줌
```swift
let sameIDCompletable = Completable.zip(
    intersectingRemoteProjects.flatMap { remoteProject in
        intersectingLocalProjects.filter { localProject in
            localProject.updatedAt > remoteProject.updatedAt
        }.map {
            return self.remoteDataSource.update($0)
        }
    }
)
```
# zip과 combineLatest의 차이와 사용법
디폴트로 combineLatest를 사용하고 특정한 경우에만 zip을 사용하기
둘다 두 Observable을 합칠때 사용하지만, 합치는 방식이 다르다.

1. Single의 경우 zip만 사용가능
2. combineLatest는 두 Observable이 다른 타입인 경우에도 사용가능하지만 zip은 둘의 타입이 같아야함
## zip
![](https://velog.velcdn.com/images/dev_jane/post/0ce8c901-167a-4326-bdbc-e9818d529263/image.png)
- 두 Observable 중 방출하는 요소가 작은 Observable의 요소 수만큼 합쳐진 결과가 방출된다
    - 예를 들어 a Observable의 요소 개수가 3이고, b Observable의 요소 개수가 4일때 합쳐진 결과는 3개가 나온다는 뜻이다.
- 방출된 순서가 같은 요소끼리만 합쳐진다.
    - 예를 들어  a Observable에서 `1, 2, 3`이 방출되었고, b Observable에서 `1, 2`가 방출되었다면 합친 결과는 `1-1, 2-2` 가 된다. a Observable의 `3`은 짝이 없어서 합쳐지지 않는다.

### 프로젝트 적용 예시

Observable의 변형인 Traits의 한 종류인 Completable을 zip으로 묶어서 나오는 이벤트를 구독하여, 두 Completable에서 onCompleted 이벤트가 둘다 내려오면 하나의 Completable 이벤트를 발생시킨다. 
```swift
Completable.zip(
    remoteDataSource.delete(project),
    localDataSource.delete(project)
).subscribe(onCompleted: { [self] in
    var currentProjects = projects.value
    if let row = currentProjects.firstIndex(where: { $0.id == project.id }) {
        currentProjects.remove(at: row)
    }
    projects.accept(currentProjects)
}).disposed(by: disposeBag)
```
## combineLatest
![](https://velog.velcdn.com/images/dev_jane/post/8a5e1e2e-8227-48cf-8862-a290fea68efe/image.png)

- 두 Observable의 요소가 최소한 하나씩 방출되어야 합쳐진 시퀀스에서 이벤트가 발생한다.
    - 예를 들어  a Observable에서 `1` 만 나온 상태일때는 아무런 이벤트도 발생하지 않는다.
- 한 Observable에서 요소가 방출되면 다른 Observable에서 제일 최근에 방출된 요소랑 합쳐진다.

따라서 두 Observable의 요소가 하나씩 방출되고 나서는 한 Observable에서 요소가 방출될때마다 합쳐진 시퀀스에서 이벤트가 발생한다. 

- 예를 들어 a Observable과  b Observable에서 각각 `1, 2` 두개씩 나온 상태일때 b Observable에서 `3`이 나오면 a Observable의 가장 최신 요소인 `2`와 합쳐진다.

### 비교 실험
```swift
import RxSwift
import Foundation

func exampleZip(a: Observable<Int>) -> Observable<(Int, String)> {
    let b = a.map { "\($0)" }
    return Observable.zip(a, b)
}

func exampleCombineLatest(a: Observable<Int>) -> Observable<(Int, String)> {
    let b = a.map { "\($0)" }
    return Observable.combineLatest(a, b)
}

exampleZip(a: Observable.from([1, 2, 3]))
    .subscribe(onNext: { print("zip", $0) })

exampleCombineLatest(a: Observable.from([1, 2, 3]))
    .subscribe(onNext: { print("combineLatest", $0) })
```

```swift
zip (1, "1")
zip (2, "2")
zip (3, "3")
combineLatest (1, "1")
combineLatest (2, "1")
combineLatest (2, "2")
combineLatest (3, "2")
combineLatest (3, "3")
```

# Reference
[https://reactivex.io/documentation/operators/combinelatest.html](https://reactivex.io/documentation/operators/combinelatest.html)
[https://reactivex.io/documentation/operators/zip.html](https://reactivex.io/documentation/operators/zip.html)


# flatMap

Swift에서 원래 flatMap은 요소들을 평평하게 펴주는 역할을 한다
map과 동일하게 안에있는 요소들을 변형하는 기능도 하지만, 결과적으로 중첩된 배열을 단일 배열로 바꿔서 리턴한다. 

```swift
let intArray = [1,2]
let map = intArray.map { [$0*2, $0*3] }
let flatMap = intArray.flatMap { [$0*2, $0*3] }
    
print(map) //[[2, 3], [4, 6]]
print(flatMap) //[2, 3, 4, 6]
```

이와 비슷하게, rxSwift에서 flatMap은 Observable을 평평하게 펴준다고 간단하게 생각할 수 있겠다. 

## rxSwift에서 map과 flatMap의 차이는

클로저 리턴값이 다르다는 것이다.
만약 연산 결과 Observable을 얻고 싶다면, 

- 클로저에서 map은 Observable 안의 요소(Int, String, Float 등등)를 리턴값으로 지정해줘야하고
- 클로저에서 flatMap은 Observable 자체(Observable<String>)를 리턴값으로 지정해주는 것이다.

결과는 둘다 `Observable<String>` 이 된다. 

```swift
let numberObservable = Observable<Int>.just(10)

//축약형
let observableAfterMap = numberObservable.map { String($0) }
let observableAfterFlatMap = numberObservable.flatMap { Observable<String>.just(String($0)) }

//풀어서 썼을때
let observableAfterMap = numberObservable.map { number -> String in
    return String(number)
}
let observableAfterFlatMap = numberObservable.flatMap { number -> Observable<String> in 
        return Observable<String>.just(String(number))
}
```

<정의 부분>

- map
    
    ```swift
    public func map<Result>(_ transform: @escaping (Element) throws -> Result)
        -> Observable<Result> {
        Map(source: self.asObservable(), transform: transform)
    }
    ```
    
- flatMap
    
    ```swift
    public func flatMap<Source: ObservableConvertibleType>(_ selector: @escaping (Element) throws -> Source)
        -> Observable<Source.Element> {
            return FlatMap(source: self.asObservable(), selector: selector)
    }
    ```
    

rxSwift에서 보통 map은 Observable에서 방출되는 item을 변형하고자 할 때 사용된다. 
예를 들어 Observable<Int> → Observable<String> 으로 변형할 때 사용할 수 있다. 
rxSwift에서의 flatMap은 Observable에서 방출되는 item을 변형하고자 할 때 사용되는데, 결과적인 형태가 Observable이 되도록 한다.

몇가지 예시들을 보자... 

### 예시1

API 호출 결과 Observable을 리턴하는 함수를 리턴할때 flatMap의 결과값은 Observable<Data>이다.

만약 map을 사용해서 변형했다면 결과값은 Observable로 한겹 더 감싸진 Observable<Observable<Data>>가 됐을 것이다. 

```swift
let url = URL(string: ....)

Observable.just(url)
    .flatMap { url -> Observable<Data> in
        let request = URLRequest(url: url)
        return URLSession.shared.rx.data(request: request)
    }.map { data -> [SomeModel]? in
        return try? JSONDecoder().decode(SomeModelList.self, from: data).models
    }.subscribe(onNext: { [weak self] models in
        if let models = models {
            self?.models = models
            DispatchQueue.main.async {
                self?.tableView.reloadData()
            }
        }
    }).disposed(by: disposeBag)
```

### 예시2

Observable<Bool>을 리턴하는 shouldListenToUpdated() 함수에 
  map을 하면, `Observable<Observable<String>>` 이 나오고
flatMap을 하면, `Observable<String>` 이 나오게 된다. 

```swift
func updates() -> Observable<String> {
    return Observable.just(String(1))
}

func shouldListenToUpdated() -> Observable<Bool> {
    return Observable.just(true)
}

let aa = shouldListenToUpdated()
    .map { (listenToUpdated) -> Observable<String> in
        return listenToUpdated ? updates() : Observable.empty()
    }

let bb = shouldListenToUpdated()
    .flatMap { (listenToUpdated) -> Observable<String> in
        return listenToUpdated ? updates() : Observable.empty()
    }
```


![](https://velog.velcdn.com/images/dev_jane/post/a320c2c5-00e6-453b-9e75-71451fd42324/image.png)


### 예시3

마지막으로 곰튀김님의 영상에 나온 map과 flatMap을 동시에 사용한 예시를 보자 

- map으로 Observable<String> → Observable<URL>로 변경하고
- flatMap으로  Observable<URL> → Observable<UIImage?>로 변경한다

catchedImageLoding의 리턴 타입이 Observable<UIImage?>라서 flatMap을 안쓴다면 `Observable<Observable<UIImage?>>` 이런식으로 됐을텐데 최종적으로 Observable을 리턴하게 해서 한겹 껍질을 벗기게 되는 것이다. 
![](https://velog.velcdn.com/images/dev_jane/post/01562a12-d48d-416a-bb09-3d3abaf6da36/image.png)

# Reference
  [https://eunjin3786.tistory.com/41](https://eunjin3786.tistory.com/41)
  [https://reactivex.io/documentation/operators/flatmap.html](https://reactivex.io/documentation/operators/flatmap.html)
