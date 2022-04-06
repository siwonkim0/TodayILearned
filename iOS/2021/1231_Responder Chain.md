# 학습 내용
- Responder Chain

## Responder Chain
```swift
class ViewController: UIViewController {
    
    @IBOutlet weak var greenView: Green!
    override func viewDidLoad() {
        super.viewDidLoad()
        let gesture = UITapGestureRecognizer(target: self, action: #selector(tapGreenView(_:)))
        greenView.addGestureRecognizer(gesture)
    }
    
    @objc func tapGreenView(_ gesture: UITapGestureRecognizer) {
        print("그린뷰 탭됨!!!")
    }
}
```

그린뷰에 gesture recognizer을 추가하여 그린뷰를 터치했을때 tapGreenView 메서드가 불리게 하고싶었다.

1. IBOutlet 으로 greenView 연결 
2. viewDidLoad에서 UITapGestureRecognizer 인스턴스 생성
    - target 은 뷰컨트롤러
    - action은 뷰컨에 구현된 objc 메서드인 tapGreenView 
    *selector 안에는 objc 메서드만 들어올 수 있다. 
3. viewDidLoad에서 2에서 생성한 UITapGestureRecognizer 인스턴스를 전달하여greenView의 addGestureRecognizer 메서드 실행


### UIGestureRecognizer: init(target:action:)

**action은 반드시 target에서 구현된 메서드**여야한다.
만약 target가 뷰컨이라면, action은 반드시 뷰컨에서 구현한 objc 메서드여야함

그러니.. 만약 target를 그린뷰로 바꾸고 싶다면, 앞서 뷰컨에서 구현한 메서드를 그린뷰로 옮겨주기만하면 된다. 

- target : An object that is the recipient of action messages sent by the receiver when it recognizes a gesture. 
- action : A selector that identifies the method **implemented by the target** to handle the gesture recognized by the receiver. 

![](https://i.imgur.com/XUckX7J.png)

target과 action의 주체가 맞지 않으면 selector을 인식할 수 없다는 에러가 뜨게된다. 

![](https://i.imgur.com/L1fxKZu.png)


touchesBegan 메서드를 그린뷰로 옮기고 target을 greenView로 지정해준 예시. 
```swift
class ViewController: UIViewController {
    
    @IBOutlet weak var greenView: Green!
    override func viewDidLoad() {
        super.viewDidLoad()
        let gesture = UITapGestureRecognizer(target: greenView, action: #selector(greenView.tapGreenView(_:)))
        greenView.addGestureRecognizer(gesture)
    }

}
```

```swift
class Green: UIView {
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        print("greeeeen")
        super.touchesBegan(touches, with: event)
    }
}
```

### Hit Test로 First Responder 찾고 -> (Gesture Recognizer) -> Responder Chain

1st UIView 의 hitTest(_:with:) 메서드는 뷰 계층을 돌아다니며 이벤트가 발생한 터치 지점을 포함하고 있는 뷰계층 최하단의 서브뷰를 찾음: 그것을 터치 이벤트의 First Responder 로 지정

*만약 터치 이벤트가 발생한 위치가 뷰의 bounds 의 밖에 있다면, hitTest(_:with:) 는 해당 뷰와 서브뷰를 모두 무시함

2nd 뷰 앞에 Gestsure recognizer 가 있다면, Gesture recognizer 가 먼저 이벤트를 전달받고 처리를 시도한다. 

3rd 그리고 Gestsure recognizer가 처리하지 못하면 responder chain에 이벤트가 전달된다. 

