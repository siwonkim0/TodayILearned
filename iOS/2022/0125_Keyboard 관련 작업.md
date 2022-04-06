### UIResponder란?

이벤트에 반응하고 처리하는 추상 클래스

이벤트가 발생하면 UIKit는 이벤트를 앱의 responder object들에게 보낸다...

responder object들이 받은 이벤트를 처리하기위해서는 UIResponder의 특정 메서드들을 오버라이드하면된다. ⇒  UIViewController, UIView 객체들이 다 responder object들이다. 

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220125/61f006d92947bf40f08f38d7.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220125/61f006d92947bf40f08f38d7.png)

- 이벤트의 종류에는 touch, press, remotecontrol.. 등이 있고 각 종류마다 메서드들이 있음
    - touchesBegan
    - touchesEnded ... 등이 대표적으로 있음
- 이러한 이벤트들을 받아서 앱의 인터페이스를 업데이트하는거임

이벤트 처리하는것 외에도 하는 일이 하나 더 있는데, 처리되지않은 이벤트들을 다른데로 보내는 역할도한다 

responder chain의 object들에게 차례대로 보냄

UITextView를 사용자가 터치하면 자동으로 얘가 first responder가 되어서 키보드를 보여준다,, 

(터치 이벤트 발생 → UITextView가 이벤트를 처음으로 받게됨)

그리고 키보드가 나타나거나 사라질때, iOS는 아래의 노티피케이션 등록된 observer들에게 보낸다

*first responder가 임의로 되게하고싶으면 `.becomeFirstResponder()`

*first responder를 해제하고싶으면 `.resignFirstResponder()`
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220125/61f007362947bf40f08f38d9.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220125/61f007362947bf40f08f38d9.png)


### UIResponder의 타입 프로퍼티들

- UIKeyboardWillShowNotification
- UIKeyboardDidShowNotification
- UIKeyboardWillHideNotification
- UIKeyboardDidHideNotification

→ 오늘은 여기서 willshow랑 willhide 쓸거임 

→ Notification에는 userInfo가 담겨서 온다 (키보드 사이즈 정보)

순서

1. `addObserver(_:selector:name:object:)` 

이 메서드를 사용하여 뷰컨을 observer로 등록하여 노티를 받으면 실행할 함수를 설정할 수 있다. 

그러니까 키보드가 나타난다는 노티를 받고 그에 대한 처리를 뷰컨이 해줄수가 있는것이다.

- observer : self (뷰컨)
- selector
    - 노티 받으면 실행할 함수
    - 선언부에 @objc 표시 - 사용부에 #selector()로 감싸기
    - 파라미터가 하나이고, NSNotification 타입이어야함
- name : 받을 노티가 뭔지 적어주기

*원래는 `removeObserver(_:)` 를 사용하여 사용후에 observer를 remove해줘야하는데 iOS 9 이상부터는 자동으로 다음 post가 일어날때 남아있는 옵져버를 클린해줘서 괜찮음 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillShow(_:)), name: UIResponder.keyboardWillShowNotification, object: nil)
    NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillHide(_:)), name: UIResponder.keyboardWillHideNotification, object: nil)
}
```

1. selector에 들어갈 액션함수 정의하기

: 키보드가 나타난다는 노티를 받고 그에대한 처리 정의해주기

- userInfo를 이용하여 키보드 사이즈 가져오기
- 스크롤뷰의 bottom content inset을 키보드 높이만큼 조정하기

```swift
@objc func keyboardWillShow(_ sender: Notification) {
    let userInfo: NSDictionary = sender.userInfo! as NSDictionary
    guard let keyboardFrame: NSValue = userInfo.value(forKey: UIResponder.keyboardFrameEndUserInfoKey) as? NSValue else {return}

    let keyboardRect = keyboardFrame.cgRectValue
    scrollView.contentInset.bottom = keyboardRect.height
    scrollView.scrollIndicatorInsets = scrollView.contentInset
}

@objc func keyboardWillHide(_ sender: Notification) {
    scrollView.contentInset = .zero
    scrollView.scrollIndicatorInsets = .zero
}
```

[Managing the Keyboard](https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html#//apple_ref/doc/uid/TP40009542-CH5-SW1)


![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220125/61f007392947bf40f08f38db.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220125/61f007392947bf40f08f38db.png)
## textField 마다 다른 키보드 타입 설정하기

```swift
UITextField().keyboardType = .numberPad 
```



### return 누르면 다음 textField로 가게하기

```swift
extension LoginView: UITextFieldDelegate {
  func textFieldShouldReturn(_ textField: UITextField) -> Bool {
    if textField == emailTextField {
      passwordTextField.becomeFirstResponder()
    } else {
      passwordTextField.resignFirstResponder()
    }
    return true
  }
}
```

### 빈 뷰 누르면 키보드 내려가게하기

```swift
extension UIViewController {
    func hideKeyboardWhenTappedAround() {
        let tap = UITapGestureRecognizer(target: self, action: #selector(UIViewController.dismissKeyboard))
        tap.cancelsTouchesInView = false
        view.addGestureRecognizer(tap)
    }
    
    @objc func dismissKeyboard() {
        view.endEditing(true)
    }
}
```
