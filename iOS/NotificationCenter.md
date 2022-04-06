### 노티피케이션 센터 적용하기

1. 노티피케이션 센터 `.default`선언
2. 노티피케이션 이름 설정 (노티 종류 선언)
3. `post`: 노티피케이션 센터에게 노티요청
4. 함수 앞에 `@objc` 키워드 붙여서 방송 들으면 실행할 함수 선언
5. `addObserver` 로 노티피케이션 센터 구독신청하기

```swift
import UIKit

//1st 어떤 방송을 때릴지
let didPressButton: Notification.Name = Notification.Name("버튼누름")
let notificationCenter: NotificationCenter = .default

// Model
// 정보가 바뀌지 않으니 구조체가 나음 (클래스보다)
struct Registrant {
    let name: String
    let phoneNumber: String
    
    
}

class ViewController: UIViewController {
    
    @IBOutlet weak var nameTextField: UITextField!
    @IBOutlet weak var phoneNumberTextField: UITextField!
    
    @IBOutlet weak var nameLabel: UILabel!
    @IBOutlet weak var phoneNumberLabel: UILabel!
    

    // Model
    var registrantList: [Registrant] = []
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        //구독신청 4???
        notificationCenter.addObserver(self,
                                        selector: #selector(showInfo), //[name]방송을 들으면 [selector]를 실행하겠다!
                                        name: didPressButton,
                                        object: nil)
    }
    
    //controller
    @IBAction func hitRegisterButton(_ sender: Any) {
        register()
        
        //view
        nameTextField.text = ""
        phoneNumberTextField.text = ""
    }
    
    @IBAction func hitCheckButton(_ sender: Any) {
    
    }
    
    @objc
    func showInfo() {
        //방송을 듣고나서 해야할 일 3
        let currentIndex = registrantList.count - 1
        nameLabel.text = registrantList[currentIndex].name
        phoneNumberLabel.text = registrantList[currentIndex].phoneNumber
    }
    
    
    
    func register() {
        guard let nameText = nameTextField.text else {
            return
        }
        
        guard let phoneNumberText = phoneNumberTextField.text else {
            return
        }
        
        let personalInfo = Registrant(name: nameText, phoneNumber: phoneNumberText)
        registrantList.append(personalInfo)
        
        // 배열에 연락처 추가한 후에 방송해달라고 방송국에게 요청 2
        notificationCenter.post(name: didPressButton, object: nil)
    }
}
```


