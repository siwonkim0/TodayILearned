## 학습 내용
- 쥬스메이커 STEP 2 리팩토링
    * StringConverter 구조체로 따로 분리
- 사전과제 서로 비교하기
* KVO 공식문서 읽기 

## 학습 키워드
- 뷰컨의 storyboard ID를 이용한 화면전환(모달)
* 노티피케이션 센터 + 프로퍼티 감시자 적용
* KVO

## 의문점
- 프로퍼티 감시자가 이니셜라이즈 되는 시점에서는 작동하지 않는 문제: 재고가 변하면 프로퍼티 감시자가 작동해서 뷰에 알려줘야하는데, 초기 화면에서는 작동하지 않아서 그냥 viewDidLoad 안에 초기값을 넣어주는 식으로 해결했다. 

---
## 뷰컨트롤러의 storyboard ID를 이용한 화면전환(모달)
재고수정 버튼을 클릭시 재고수정 화면으로 전환되도록 코드로 구현하였다. 네비게이션 컨트롤러로 화면을 전환하면 자동으로 연결된 뷰컨트롤러가 보여진다. 

재고수정 버튼 + Ctrl 드래그로 재고수정 뷰에 끌어다 놓아서 present modally 라고 설정해줄 수도 있지만, 코드로 구현해보고 싶어서 도전해보았다. 

![](https://images.velog.io/images/dev_jane/post/40b80254-05f3-47ec-8d47-1d0413b13057/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-10-29%20%EC%98%A4%EC%A0%84%2012.08.29.png)

```swift
import UIKit

class StockModifyNavController: UINavigationController {
// 만약 화면 전환이 여러 곳에서 일어나게 되면 매번 문자열로 적어줘야하기 때문에 내부 프로퍼티로 만들어줌.    

    let storybordName = "Main" 
// storyboard.instantiateViewController(identifier: ...) identifier 전달인자로 넘겨줄 문자열
    let storybordID = "StockModifyNavController" 

    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
}
```
먼저, 뷰 컨트롤러의 storybordName과 storyboardID를 담아두는 클래스를 하나 만들었다. (속성에 접근해서 문자열을 꺼내쓰기 위해)


```swift
@IBAction func pressStockModifyButton(_ sender: UIBarButtonItem) {
    let stockModifyNavController = StockModifyNavController()
        
    let storyboard = UIStoryboard(name: stockModifyNavController.storybordName, bundle: nil)
    let stockModifyNC = storyboard.instantiateViewController(identifier: stockModifyNavController.storybordID)
        
    present(stockModifyNC, animated: true, completion: nil)
}
```

그리고 재고수정하기 버튼을 누르면 수행할 액션을 정의했는데,
일단 StockModifyNavController의 인스턴스를 선언하고, 그 속성에 접근하여 storybordName과 storyboardID를 가져왔다. 
그리고 present함수를 이용해서 모달로 창을 띄워주었다. 

**참고한 공식문서**
![공식문서 참고](https://images.velog.io/images/dev_jane/post/44d78582-2729-4937-9623-de4c5ad93983/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-10-29%20%EC%98%A4%EC%A0%84%2012.15.22.png)

---
## Notification Center 과 프로퍼티 감시자

```swift
class FruitStore {
    static let shared = FruitStore()
    private static let initialFruitQuantity = 10
    
    let didChangeStock = Notification.Name("재고 수정 완료")
    private var stock: [Fruit: Int] {
        didSet {
            NotificationCenter.default.post(name: didChangeStock, object: nil)
        }
    }
    
    private init(quantity: Int = FruitStore.initialFruitQuantity) {
        self.stock = FruitStore.initializeStock(quantity: quantity)
    }
 }
```
Notification.Name을 정의해준 후에, 
프로퍼티 감시자를 사용하여 FruitStore 클래스의 프로퍼티인 stock에 재고가 변하면 post를 보내는 함수를 추가하였다. 



```swift
class JuiceMakerViewController: UIViewController {

    @IBOutlet weak var strawberryStockLabel: UILabel!
    //코드 생략
    let juiceMaker = JuiceMaker()
    
    override func viewDidLoad() {
        super.viewDidLoad()
  
        initJuiceMakerViewController()
    }
    
    func initJuiceMakerViewController() {
        changeStockLabel()
        NotificationCenter.default.addObserver(self, selector: #selector(changeStockLabel), name: FruitStore.shared.didChangeStock, object: nil)
    }

    @objc
    func changeStockLabel() {
        strawberryStockLabel.text =  FruitStore.shared.showStock(of: .strawberry)
       //코드 생략
    }
```
그리고 뷰컨트롤러에 addObserver 함수를 사용하여 방송을 듣고나서 실행할 함수를 @objc로 표시해주었다. 

그랬더니 재고가 변할때마다 뷰에 변동사항이 잘 반영이 되었다.
어제 노티피케이션 센터를 토요 스터디 자료로 열심히 복습했더니 프로젝트에 쉽게 적용이 되어서 너무 기뻤다.

그런데 앱을 처음 실행할 때 FruitStore 싱글턴 인스턴스가 이니셜라이즈될 때는 노티피케이션센터가 작동하지 않아서 뷰에 재고가 표시되지 않는 문제가 발생해서 일단 viewDidLoad에 재고 레이블을 업데이트해주는 함수인 changeStockLabel()을 넣어놨다.
이 부분은 개선할 수 있을 것 같은데 더 고민해봐야겠다. 

---

## KVO
KVO 공식문서를 읽어보았다.
NSObject에서 상속받은 클래스끼리만 사용 가능한 코코아 프로그래밍 패턴이다. 모델과 뷰처럼 논리적으로 분리된 부분끼리의 소통을 할 때 유용하게 쓰인다. 

- 관찰할 프로퍼티에 주석을 단다: @objc dynamic
- 관찰자 클래스를 정의한다: observe 메서드 사용
- 관찰할 프로퍼티의 클래스의 인스턴스를 정의하고, 관찰자 클래스의 인스턴스를 정의할 때 매개변수로 넘겨준다. 
- 프로퍼티가 변할때 자동으로 관찰자의 change handler가 호출된다.

의문점
- 프로퍼티 옵저버와의 차이는 뭐지?
- 노티피케이션 센터와의 차이는 뭐지?








