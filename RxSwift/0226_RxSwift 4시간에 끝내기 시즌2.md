## RxSwift 4시간에 끝내기 시즌2

- 옵셔널 클로져는 디폴트가 escaping이라서 @.escaping을 써주지 않아도 된다.
   
- Rx 적용 전에는 /....
    - 데이터 통신 결과를 받아오는 함수인 downloadJson 안에 DispatchQueue도 넣어서 메인 함수인 onLoad()를 간단하게 만들고 싶다
    
    ```swift
    func downloadJson(_url: String) -> String? {
        let url = URL(string: MEMBER_LIST_URL)!
        let data = try! Data(contentsOf: url)
        let json = String(data: data, encoding: .utf8)
        return json
    }
    
    @IBAction func onLoad() {
        editView.text = ""
        self.setVisibleWithAnimation(self.activityIndicator, true)
        DispatchQueue.global().async {
            **let json = self.downloadJson(_url: MEMBER_LIST_URL)**
            DispatchQueue.main.async {
                self.editView.text = json
                self.setVisibleWithAnimation(self.activityIndicator, false)
            }
        }
        
    }
    ```
    
    따라서 디스패치큐를 downloadJson으로 옮겨왔더니 리턴값을 못쓰고 
    
    completionHandler을 사용해야함 
    
    ```swift
    func downloadJson(_url: String, completion: ((String?) -> Void)?) {
        DispatchQueue.global().async {
            let url = URL(string: MEMBER_LIST_URL)!
            let data = try! Data(contentsOf: url)
            let json = String(data: data, encoding: .utf8)
            DispatchQueue.main.async {
                completion?(json)
            }
        }
    }
    
    @IBAction func onLoad() {
        editView.text = ""
        self.setVisibleWithAnimation(self.activityIndicator, true)
    
        **downloadJson(_url: MEMBER_LIST_URL)** { json in
            self.editView.text = json
            self.setVisibleWithAnimation(self.activityIndicator, false)
        }
        
    }
    ```
    
    completionHandler 사용시 네트워크 통신을 여러번 하는 경우 복잡한 코드가 된다.
    
    ![이렇게](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99d964cd-e780-44ee-b5ff-fc4017285a1d/스크린샷_2022-02-17_오후_7.39.28.png)
    
    이렇게
    

### 고민) 비동기로 생긴 데이터를 completion이 아니라 어떻게 리턴값으로 만들지?

→ 리턴값을 나중에생기는데이터<>로 감싸는 것 

```swift
func downloadJson(_ url: String) -> **나중에생기는데이터<String?>** {
    return 나중에생기는데이터() { f in
        DispatchQueue.global().async {
            let url = URL(string: url)!
            let data = try! Data(contentsOf: url)
            let json = String(data: data, encoding: .utf8)
            
            DispatchQueue.main.async {
                f(json)
            }
        }
    }
}

// MARK: SYNC

@IBOutlet var activityIndicator: UIActivityIndicatorView!

@IBAction func onLoad() {
    editView.text = ""
    self.setVisibleWithAnimation(self.activityIndicator, true)
    let json: 나중에생기는데이터<String?> = downloadJson(MEMBER_LIST_URL)
    json.나중에오면 { json in
        self.editView.text = json
        self.setVisibleWithAnimation(self.activityIndicator, false)
    }
}
```

여러 유틸리티가 생겨남 

- PromiseKit
- Bolt
- RxSwift
    - 나중에생기는 데이터 **observable**
    - .**subscribe**

```swift
class ViewController: UIViewController {
    @IBOutlet var timerLabel: UILabel!
    @IBOutlet var editView: UITextView!

    override func viewDidLoad() {
        super.viewDidLoad()
        Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) { [weak self] _ in
            self?.timerLabel.text = "\(Date().timeIntervalSince1970)"
        }
    }

    private func setVisibleWithAnimation(_ v: UIView?, _ s: Bool) {
        guard let v = v else { return }
        UIView.animate(withDuration: 0.3, animations: { [weak v] in
            v?.isHidden = !s
        }, completion: { [weak self] _ in
            self?.view.layoutIfNeeded()
        })
    }
    
    func downloadJson(_ url: String) -> Observable<String?> {
        return Observable.create() { f in
            DispatchQueue.global().async {
                let url = URL(string: url)!
                let data = try! Data(contentsOf: url)
                let json = String(data: data, encoding: .utf8)
                
                DispatchQueue.main.async {
                    f.onNext(json)
                }
            }
            
            return Disposables.create()
        }
    }

    // MARK: SYNC

    @IBOutlet var activityIndicator: UIActivityIndicatorView!
  
    @IBAction func onLoad() {
        editView.text = ""
        self.setVisibleWithAnimation(self.activityIndicator, true)
        downloadJson(MEMBER_LIST_URL) //얘의 리턴값이 나중에생기는 데이터
            .subscribe { event in
                switch event {
                case .next(let json):
                    self.editView.text = json
                    self.setVisibleWithAnimation(self.activityIndicator, false)
                case .completed:
                    break
                case .error:
                    break
                }
            
        }
    }
}
```

나중에생기는 데이터는 Observable<> 형태로 나타난다.

나중에 생기는 데이터를 사용할때는 .subscribe를 호출하면 되는데

event가 오는데 종류가 세가지다

next : 데이터가 전달될때

completed : 데이터 전달이 끝났을때

error : 에러 

```swift
func downloadJson(_ url: String) -> Observable<String?> {
        // 1. 비동기로 생기는 데이터를 Observable로 감싸서 리턴하는 방법
        return Observable.create() { emitter in
            emitter.onNext("Hello") //데이터 여러개 전달
            emitter.onNext("World")
            emitter.onCompleted() //데이터가 끝났다고
            
            return Disposables.create()
        }
    }

    // MARK: SYNC

    @IBOutlet var activityIndicator: UIActivityIndicatorView!
  
    @IBAction func onLoad() {
        editView.text = ""
        self.setVisibleWithAnimation(self.activityIndicator, true)
        
        // 2. Observable로 오는 데이터를 받아서 처리하는 방법
        downloadJson(MEMBER_LIST_URL)
            .subscribe { event in
                switch event {
                case .next(let json):
                    self.editView.text = json //여기에 Hello와 World가 전달됨
                    self.setVisibleWithAnimation(self.activityIndicator, false)
                case .completed:
                    break
                case .error:
                    break
                }
        }
    
    }
```

### Observable 의 생명주기

1. create() 로 만들어지고
2. subscribe() → 이때 동작 !!! (만들어 놓는다고 해서 실행되는게 아님)
3. next 로 데이터 전달 

---

종료

1. complete로 끝난다 / error가 남 
2. Disposed

*한번 끝난 observable은 같은 subscribe로 재사용이 안된다

다시 실행하려면 다른 subscribe가 필요하다

![스크린샷 2022-02-17 오후 10.58.18.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/74df2eea-94bc-41f7-baf9-86f87379dabd/스크린샷_2022-02-17_오후_10.58.18.png)

### .debug()

어떤 데이터가 전달되는지 디버그창에 찍히도록 하는 메서드 

```swift
@IBAction func onLoad() {
    editView.text = ""
    self.setVisibleWithAnimation(self.activityIndicator, true)
    
    // 2. Observable로 오는 데이터를 받아서 처리하는 방법
    downloadJson(MEMBER_LIST_URL)
        .debug() //어떤 데이터가 전달이되는지 디버그에 찍힘
        .subscribe { event in
            switch event {
            case .next(let json):
                DispatchQueue.main.async {
                    self.editView.text = json
                    self.setVisibleWithAnimation(self.activityIndicator, false)
                }
            case .completed:
                break
            case .error:
                break
            }
    }

}
```

![스크린샷 2022-02-17 오후 11.04.01.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2b15893a-acc8-492f-ad92-8f81b9a84538/스크린샷_2022-02-17_오후_11.04.01.png)

![스크린샷 2022-02-17 오후 11.04.16.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/70e502f5-4def-428c-9ca0-edd130b27b35/스크린샷_2022-02-17_오후_11.04.16.png)

```swift
func downloadJson(_ url: String) -> Observable<String?> {
    // 1. 비동기로 생기는 데이터를 Observable로 감싸서 리턴하는 방법
    return Observable.create() { emitter in
        let url = URL(string: url)!
        let task = URLSession.shared.dataTask(with: url) { data, _, error in
            guard error == nil else {
                emitter.onError(error!)
                return
            }
            
            if let data = data, let json = String(data: data, encoding: .utf8) {
                emitter.onNext(json) //데이터가 준비가 안됐으면 next를 안보내고 밑에 complete만 보냄
            }
            
            emitter.onCompleted()
        }
        
        task.resume()
        
        return Disposables.create() {
            task.cancel()
        }
    }

@IBOutlet var activityIndicator: UIActivityIndicatorView!

@IBAction func onLoad() {
    editView.text = ""
    self.setVisibleWithAnimation(self.activityIndicator, true)
    
    let observable = downloadJson(MEMBER_LIST_URL)
    // 2. Observable로 오는 데이터를 받아서 처리하는 방법
    observable.subscribe({ event in
        switch event {
        case .next(let json):
            DispatchQueue.main.async {
            self.editView.text = json
            }
         break
        case .error(let err):
            break
        case .completed:
            break
        }
        
    })
}
}
```

지금까지가 기본 사용법이고,

귀찮은 것들을 없애주기 위한 API들이 또 있음 

## Sugar API

생성, subsctibe(내가 원하는 값만 받을 수 있음..next..), 데이터 조작(중간에 쓰레드 바꾸기 등등) 모두 operator

종류... 

생성 just, from

데이터 변형

데이터 필터

여러 observable를 묶어서 처리...zip

에러 핸들링

[https://reactivex.io/documentation/operators.html](https://reactivex.io/documentation/operators.html)

- Observable.create()로 만들때 데이터 하나 만드는데 여러줄 쓰기 너무 귀찮다.

아래 5줄을 한줄로 대체할 수 있음 

just로 하나의 데이터를 보내면 됨 

```swift
func downloadJson(_ url: String) -> Observable<String?> {
    return Observable.just("Hello World")
//        return Observable.create() { emitter in
//            emitter.onNext("Hello World")
//            emitter.onCompleted()
//            return Disposables.create()
//        }
}
```

`just`로 여러개의 데이터 보내기도 가능 → 근데 하나의 배열로 오긴함 

```swift
func downloadJson(_ url: String) -> Observable<[String]> {
    return Observable.just(["Hello", "World"])
}
```

![스크린샷 2022-02-17 오후 11.31.36.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58c1671d-8142-43d7-9125-00e20207dd4b/스크린샷_2022-02-17_오후_11.31.36.png)

`from`을 쓰면 여러개의 데이터를 받을 수 도 있음 

```swift
func downloadJson(_ url: String) -> Observable<String> {
    return Observable.from(["Hello", "World"])
}
```

![스크린샷 2022-02-17 오후 11.32.52.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f3e1f2a-ad9b-4b6b-bc02-7a735cffc76b/스크린샷_2022-02-17_오후_11.32.52.png)

- 마찬가지로 .subscribe할때도 개귀찮

이렇게 onNext만 받아서 처리해줘도 된다.

```swift
let observable = downloadJson(MEMBER_LIST_URL)
    // 2. Observable로 오는 데이터를 받아서 처리하는 방법
    observable.subscribe(onNext: {print($0)}, onCompleted: { print("Complete")})
//        observable.subscribe({ event in
//            switch event {
//            case .next(let json):
//                print(json)
//             break
//            case .error(let err):
//                break
//            case .completed:
//                break
//            }
//
//        })
}
```

- dispatchQueue.maim.async {} 도 간단히 할 수 잌ㅆ음

→ .observeOn(MainScheduler.instance) 를 위에 추가해주면됨 

데이터가 observable → subscribe 로 전달하는 그 사이에 받아서

데이터를 바꿔치기하는 슈가는 `operator`

```swift
observable.observeOn(MainScheduler.instance)
          .subscribe(onNext: { json in
              self.editView.text = json
              self.setVisibleWithAnimation(self.activityIndicator, false)
          })

//위와 같은 코드 
observable.observeOn(MainScheduler.instance)
          .subscribe(onNext: { json in
                DispatchQueue.main.async {
        self.editView.text = json
        self.setVisibleWithAnimation(self.activityIndicator, false)
        }
          })

```

### operator가 용도별로 엄청많다!!!!

![스크린샷 2022-02-18 오전 12.00.41.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7b28b745-829c-4942-a15d-b754481dfe46/스크린샷_2022-02-18_오전_12.00.41.png)

색이 바뀐건 다른 스레드로 갔다는 것을 의미

동그라미는 데이터를 의미

박스는 오퍼레이터를 의미

observeOn은 다음줄에 영향을 

subscribeOn은 위치와 상관없이 첫째줄(시작스레드)에 영향을 주는 애임

아래 코드는 그래서 concurrnet 큐에서 시작해서 observeOn 에서 Main으로 바뀜 

```swift
observable.observeOn(MainScheduler.instance) //operator
        .map { json in json.count } //operator
        .map {"\($0)"} //operator
        .subscribeOn(ConcurrentDispatchQueueScheduler(qos: .default))
        .subscribe(onNext: { json in
            self.editView.text = json
            self.setVisibleWithAnimation(self.activityIndicator, false)
        })
```

- scan

자주 사용되는데 사용법이 자주 헷갈리는 애임 

두개를 내려보내면 둘이 더해져서 내려오는 애...,. 

• `combine`, `merge`, `zip`

merge 여러개의 observable 에서 내려오는 데이터를 단순히 하나로 합치는 것 

zip 위와 아래의 데이터를 하나씩 쌍으로 만들어서 내려보냄 

combineLatest zip과 같은데 쌍이 없더라도 가장 최근에 것을 찾아서 쌍으로 만듦

### zip

observable 두개 만들어서 리턴값 두개를 합쳐서 가져오기

json 데이터랑 Hello World 문자열 두개 zip으로 합쳐서 가져올수있음 

```swift
@IBAction func onLoad() {
    editView.text = ""
    self.setVisibleWithAnimation(self.activityIndicator, true)
    
    let jsonObservable = downloadJson(MEMBER_LIST_URL)
    let helloObservable = Observable.just("Hello World")
    
    Observable.zip(jsonObservable, helloObservable) {$1 + "\n" + $0}
                .observeOn(MainScheduler.instance)
                .subscribe(onNext: { json in
            self.editView.text = json
            self.setVisibleWithAnimation(self.activityIndicator, false)
        })
```

![스크린샷 2022-02-18 오전 1.52.32.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fafe5ff9-958f-4873-96ad-adb43ebfd237/스크린샷_2022-02-18_오전_1.52.32.png)

![스크린샷 2022-02-18 오전 1.55.03.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69247996-f58c-40a1-81c1-be64da9a251d/스크린샷_2022-02-18_오전_1.55.03.png)

subscribe 의 리턴값은 Disposable인데 

이 값을 가지고있다가 나중에 취소시킬수도있음 .dispose()로 

Rx코드가 보통 이런식으로 생김 ㅋㅋ.ㅋㅋ

```swift
let disposeBag = DisposeBag()
@IBAction func onLoad() {
    editView.text = ""
    self.setVisibleWithAnimation(self.activityIndicator, true)
    
    let jsonObservable = downloadJson(MEMBER_LIST_URL)
    let helloObservable = Observable.just("Hello World")
    
    Observable.zip(jsonObservable, helloObservable) {$1 + "\n" + $0}
                .observeOn(MainScheduler.instance)
                .subscribe(onNext: { json in
            self.editView.text = json
            self.setVisibleWithAnimation(self.activityIndicator, false)
        })
                .disposed(by: disposeBag)

}
```

### flatMap()

: Observable을 벗겨주는 애 

Subject 생산도 하고 소비도 하고

Relay 는 Subject린데, 스트림이 안끊김

→ import RxRelay

PublishRelay()

화면을 구현하는 패턴 MVP, MVVM

---

## RxSwift 4시간에 끝내기 시즌2

### Bear Fried Center 예제

모델의 값이 바뀌면 자동으로 테이블뷰를 업데이트하고싶다.

버튼 클릭시 total price가 100원씩 더해지도록 하기

Rx 적용 전 

데이터를 변경하고 나서 UI 갱신을 수동으로 해줘야 한다

```swift
class MenuListViewModel {
    var menus: [Menu] = [
        Menu(name: "튀김", price: 1000, count: 2),
        Menu(name: "ㅁㅁ", price: 2000, count: 23),
        Menu(name: "ㄴㄴ", price: 3000, count: 5),
        Menu(name: "ㅇㅇ", price: 4000, count: 21),
        Menu(name: "ㄹㄹ", price: 5000, count: 2)
    ]
    
    var itemsCount: Int = 5
    var totalPrice: Int = 0
}
```

```swift
class MenuViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        updateUI()

        @IBAction func onOrder(_ sender: UIButton) {
        viewModel.totalPrice += 100
        updateUI()
    }
    
    func updateUI() {
        itemCountLabel.text = viewModel.itemsCount.description
        totalPrice.text = viewModel.totalPrice.currencyKR().description
    }
}
```

적용 후 

View Model

- Observable 사용시

처음 데이터를 표시하는건 되는데 버튼 누를때 변경하는 것은 안됨.... 

```swift
class MenuListViewModel {
    var menus: [Menu] = [
        Menu(name: "튀김", price: 1000, count: 2),
        Menu(name: "ㅁㅁ", price: 2000, count: 23),
        Menu(name: "ㄴㄴ", price: 3000, count: 5),
        Menu(name: "ㅇㅇ", price: 4000, count: 21),
        Menu(name: "ㄹㄹ", price: 5000, count: 2)
    ]
    
    var itemsCount: Int = 5
    var totalPrice: Observable<Int> = Observable.just(10000) //just로 값을 넘겨주고 있음 ...
   
}
```

- Subject 사용시

```swift
class MenuListViewModel {
    var menus: [Menu] = [
        Menu(name: "튀김", price: 1000, count: 2),
        Menu(name: "ㅁㅁ", price: 2000, count: 23),
        Menu(name: "ㄴㄴ", price: 3000, count: 5),
        Menu(name: "ㅇㅇ", price: 4000, count: 21),
        Menu(name: "ㄹㄹ", price: 5000, count: 2)
    ]
    
    var itemsCount: Int = 5

    // Observable처럼 값을 넘겨주기도 하고 Observable 외부에서 값을 변경해줄수있는 방법? Subject
    var totalPrice: PublishSubject<Int> = PublishSubject()
}
```

```swift
class MenuViewController: UIViewController {
        // MARK: - Life Cycle
        
        let viewModel = MenuListViewModel()
        var disposeBag = DisposeBag()
        
        override func viewDidLoad() {
            super.viewDidLoad()
        //        updateUI()
            viewModel.totalPrice //observable
                .scan(0, accumulator: +) //초기값 0부터 시작해서 새로운 값이 들어오면 거기에 더하겠다
                .map { $0.currencyKR() }
                .subscribe(onNext: {
                    self.totalPrice.text = $0
                })
                .disposed(by: disposeBag)
        @IBAction func onOrder(_ sender: UIButton) {
            viewModel.totalPrice.onNext(100)
        }
}

```

이미 만들어진 Observable 밖에서 값을 컨트롤할 수 있게 됨

## Subject

- AsyncSubject

여러 놈이 구독하더라도 데이터를 다 안내려보내주고 complete되는 시점에 데이터를 내려보냄 

- **BehaviorSubject**

: 기본값을 하나 가지고 시작.

누군가가 subscribe 하자마자 기본값을 내려줘

새로운 subscriber가 생기면 일단 그 직전에 생성된 값을 줘,.

그리ㅣ고 데이터가 생성되자마자 모든 구독하는 애들한테 내ㅑ려줘

- **PublishSubject**

: Subject에 누군가가 sunscribe 할 수 있음 

다른 subscribe가 새롭게 할 수도 있음 

나중에 구독하면 그 전에 내려온 데이터를 가져올 수 없음 

- ReplaySubject

두번째로 구독하는 놈이 있으면 여태까지 전달되었던 데이터를 한꺼번에 받고 시작 

- map
    
    축약 전
    
    ```swift
    lazy var totalPrice = menuObservable.map { menus in
        menus.map { menu in
            menu.price * menu.count
        }.reduce(0, +)
    }
    ```
    
    축약 후 
    
    첫번째 $0 은 menus 배열이고, 두세번쨰 $0은 menu 인스턴스임 
    
    ```swift
    lazy var totalPrice2 = menuObservable.map {
        $0.map { $0.price * $0.count }.reduce(0, +)
    }
    ```
    

### ViewModel에서

[Menu] 가 내려오거나

Int로 변한 itemCount가 내려오거나

Int로 변한 totalPrice가 내려온다

→ 이 셋중 하나를 subscribe 하는 것 

```swift
class MenuListViewModel {
    var menuObservable = PublishSubject<[Menu]>()
    
    lazy var itemsCount = menuObservable.map {
        $0.map { $0.count }.reduce(0, +)
    }
    
    // menuObservable을 바라보면서 Menu의 값이 바뀔때마다 totalPrice도 변경
    lazy var totalPrice = menuObservable.map {
        $0.map { $0.price * $0.count }.reduce(0, +)
    }

    init() {
        var menus: [Menu] = [
            Menu(name: "튀김", price: 1000, count: 2),
            Menu(name: "ㅁㅁ", price: 2000, count: 23),
            Menu(name: "ㄴㄴ", price: 3000, count: 5),
            Menu(name: "ㅇㅇ", price: 4000, count: 21),
            Menu(name: "ㄹㄹ", price: 5000, count: 2)
        ]
        menuObservable.onNext(menus)
    }
}
```

## subscribe → bind

bind 이것 역시 슈가

subscribe를 축약해서 bind로 사용 가능

순환 참조 없이 들어온 데이터를 그대로 전달해준다 

- subscribe

```swift
class MenuViewController: UIViewController {
    let viewModel = MenuListViewModel()
    var disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        viewModel.itemsCount
            .map {"\($0)"}
            **.subscribe(onNext: {
                self.itemCountLabel.text = $0
            })**.disposed(by: disposeBag)
        
        viewModel.totalPrice //observable
            .map { $0.currencyKR() }
            .subscribe(onNext: { // 값이 바뀔때마다 자동으로 next에 전달이됨
                self.totalPrice.text = $0
            })
            .disposed(by: disposeBag)
    }
}
```

- bind

```swift
class MenuViewController: UIViewController {
    let viewModel = MenuListViewModel()
    var disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        viewModel.itemsCount
            .map {"\($0)"}
            **.bind(to: itemCountLabel.rx.text)**
            .disposed(by: disposeBag)
        
        viewModel.totalPrice //observable
            .map { $0.currencyKR() }
            .subscribe(onNext: { // 값이 바뀔때마다 자동으로 next에 전달이됨
                self.totalPrice.text = $0
            })
            .disposed(by: disposeBag)
    }
}
```

### PublishSubject → BehaviorSubject

![초록색 데이터 나오고 이후에 구독시 초록색 못받음 ](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3517545f-bdf7-42f7-adfc-076f96c1aba9/스크린샷_2022-03-01_오전_1.50.54.png)

초록색 데이터 나오고 이후에 구독시 초록색 못받음 

나중에 구독하더라도 가장 마지막에 들어간 데이터를 받아오고 싶다면

BehaviorSubject로 변경하기

얘는 초기값을 갖는다

![초록색 이후에 구독 시작하더라도 직전의 데이터인 초록색 받기 가능](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/02944206-6314-4fc9-a625-90f7a2a5c347/스크린샷_2022-03-01_오전_1.53.32.png)

초록색 이후에 구독 시작하더라도 직전의 데이터인 초록색 받기 가능

```swift
class MenuListViewModel {
    var menuObservable = BehaviorSubject<[Menu]>(value: []) //초기값을 갖는다 
}
```

뷰모델 생성시점에 테이블이 채워지고

나중에 구독시점에 마지막에 받은 메뉴 데이터를 받게되는 것 

### binder

바인드를 할 수 있는 대상

```swift
itemCountLabel.rx.text
totalPrice.rx.text
```

이런식으로 바인드를 해서 데이터를 전달할 수 있다. 

```swift
viewModel.menuObservable
    .bind(to: tableView.rx.items(cellIdentifier: "MenuItemTableViewCell", cellType: MenuItemTableViewCell.self)) { index, item, cell in
        cell.title.text = item.name
        cell.price.text = item.price.description
        cell.count.text = item.count.description
    }
    .disposed(by: disposeBag)

viewModel.itemsCount
    .map {"\($0)"}
    .bind(to: itemCountLabel.rx.text)
    .disposed(by: disposeBag)

viewModel.totalPrice //observable
    .map { $0.currencyKR() }
    .bind(to: totalPrice.rx.text)
    .disposed(by: disposeBag)
```

### tableView dataSource

UITableViewDataSource 사용하는 기존 방식

```swift
extension MenuViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return viewModel.menus.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "MenuItemTableViewCell") as! MenuItemTableViewCell

        cell.title.text = viewModel.menus[indexPath.row].name
        cell.price.text = viewModel.menus[indexPath.row].price.description
        cell.count.text = viewModel.menus[indexPath.row].count.description

        return cell
    }
}
```

bind를 통해 셀을 configure할 수 있다

```swift
tableView.dataSource = nil
viewModel.menuObservable
    .bind(to: tableView.rx.items(cellIdentifier: "MenuItemTableViewCell", cellType: MenuItemTableViewCell.self)) { index, item, cell in
        cell.title.text = item.name
        cell.price.text = item.price.description
        cell.count.text = item.count.description
    }
    .disposed(by: disposeBag)
```
