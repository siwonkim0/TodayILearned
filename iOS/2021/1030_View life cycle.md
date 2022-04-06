```swift
class ViewController: UIViewController {
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        print("init")
    }
    
    // 뷰가 만들어지는 시점 -> 이 메소드 호출이 끝나면 뷰가 생성이 되어있어야함
    override func loadView() {
        super.loadView()
        print("1-loadView")
    }
    
    // 뷰가 메모리에 로드된 직후에 실행됨 -> 이땐 아직 사용자가 뷰를 못보는 시점 / 개발자가 뷰가 만들어졌다고 인지할 수 있는 시점
    override func viewDidLoad() {
        super.viewDidLoad()
        print("1-viewDidLoad")
    }
    
    // 사용자에게 보여지기 직전에 해야할 일 구현
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        print("1-viewWillAppear")
    }
    
    //뷰가 사용자에게 드디어 보임 -> 보여지고 직후에 해야할 일 구현
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        print("1-viewDidAppear")
    }
    
    
    //뷰가 사라질 것이오
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        print("1-viewWillDisappear")
    }
    
    //뷰가 사라진 직후
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        print("1-viewDidDisappear")
    }
    
    deinit {
        print("deinit")
    }

}
```

왜 상속받은 부모 클래스의 메서드를 재정의할때 super.init() 을 안써줘도 작동하지?

→ 부모 클래스의 내용이 필요할 때만 super.init()를 불러서 써주면 된다. 

init, loadView, viewDidLoad → 인스턴스가 생성될 때 한번씩만 호출된다.
