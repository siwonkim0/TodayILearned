### 싱글턴?

→ 이걸 이해하기 위해서는 값타입 / 참조타입을 이해해야함

- 왜 `private init()`? → 새로운 인스턴스를 못만들게 막아놓은거
- 왜 `static let`? → 유일한 인스턴스를 생성해서 타입 프로퍼티에 할당해 어디서든 접근가능하게

프로젝트 모듈 안에서 딱 하나만 존재해야할 필요가 있는 경우

디자인패턴중 하나

이 타입으로부터 단 하나의 인스턴스만 존재해야할 때 싱글턴을 사용해라 

ex) 앱을 사용하고 있는 사용자정보 

- 싱글턴은 클래스(참조타입)로 만들어야함

: 왜 구조체는 안돼? 

이 앱에 **하나만 존재**해야하는데 값타입이라면 다른곳에서 복사를 해갈 수 있으니  

```swift
class AccountInfo {
        //인스턴스 프로퍼티
    var id: String = ""
    var password: String = ""
    
        //타입 프로퍼티
    static let shared: AccountInfo = AccountInfo() //본인의 인스턴스를 하나 만들어서 그것의 주소값을 담고있음 -> 이렇게 잘 사용하진 않음 
    
    private init() {}
}

//shared는 타입 프로퍼티라 AccountInfo().shared 가 아니라 
let sharedAccountInfo = AccountInfo.shared //만들어놓은 단 하나의 인스턴스를 불러온거임 
```

shared 의 기본값은 AccountInfo 의 주소값을 들고 있는거고, let이기 때문에 값이 변하지 않음

몇번을 호출해도 같은 값일 것이다.

#### 싱글턴 적용 : 프로젝트 내 하나의 클래스 인스턴스만 사용할 경우 클래스를 싱글턴으로 만들어 사용할 수 있다.
    - 싱글턴은 앱 생명주기 내 **한번이라도 호출** 되어야 **인스턴스를 생성**한다.
    - 호출 == 뷰컨트롤러 안에서 호출되어야함

```swift
class RegistrantManager {
    static let shared = RegistrantManager()

    init() {
        notificationCenter.addObserver(self, selector: #selector(addResistrant), name: didRegistrantInfo, object: nil)
    }
}

```
