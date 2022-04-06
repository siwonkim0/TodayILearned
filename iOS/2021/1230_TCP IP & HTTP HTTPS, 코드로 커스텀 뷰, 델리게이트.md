# 학습 내용 
- TCP/IP & HTTP/HTTPS 
- 코드로 뷰 짜기
- 커스텀 뷰 만들기: UIButton, UIStackView, UILabel
- 델리게이트 패턴
- Git branch

## TCP/IP & HTTP/HTTPS 
둘다 통신 프로토콜(규약)임
** 통신을 주고받는 상호간의 규약을 의미
TCP/IP 는 옛날방식이고..사람이 이해하기 어려움(로우레벨)

새로운 방식이 HTTP/HTTPS.. 사람이 이해하기 쉬움

IP주소와 도메인주소는 연결되어있다
IP주소는 기억하기 어려우니 더 기억하기 쉽게 이름을 붙인게 도메인주소임

### Request - Http 요청의 핵심요소
- Get: 자료를 요청할 때 사용
- Post: 자료의 생성을 요청할 때 사용
- Put: 자료의 전체 수정을 요청할 때 사용
- Patch: 자료의 일부 수정을 요청할 때 사용
    - PATCH 메소드는 PUT 메소드와 달리 멱등성을 가지지 않는데, 이는 동일한 patch 요청이 다른 결과를 야기할 수도 있음을 뜻합니다.
    - 멱등성 : 반면 PATCH 메소드는 PUT 메소드와 달리 멱등성을 가지지 않는데, 이는 동일한 patch 요청이 다른 결과를 야기할 수도 있음을 뜻합니다.
- Delete: 자료의 삭제를 요청할 때 사용 

#### request객체
: API를 컨트롤하기 위한 메서드를 세개 가지고있음 
- request-query
: request method를 이용해 API를 컨트롤한다.
- request-body
: XML, JSON, Multi Form 등의 데이터를 담는 곳 
- request-param
: 주소에 포함된 변수를 담음(?)

#### content-type
자원을 전송할 때 어떤 타입으로 보낼지 명시하는 것 
- 파일을 전송할때 주로 사용하는 content-type는 application/json

### Response - Http 응답의 핵심요소
1xx(정보) : 요청을 받았으며 프로세스를 계속 진행합니다.
2xx(성공) : 요청을 성공적으로 받았으며 인식했고 수용하였습니다.
3xx(리다이렉션) : 요청 완료를 위해 추가 작업 조치가 필요합니다.
4xx(**클라이언트** 오류) : 요청의 문법이 잘못되었거나 요청을 처리할 수 없습니다.
5xx(**서버** 오류) : 서버가 명백히 유효한 요청에 대한 충족을 실패했습니다.

- 가장 기본적인 성공 요청 응답의 status 코드는 무엇일까요? 200
- 인증 또는 권한등의 오류에 대해서 주로 사용되는 status 코드는 무엇일까요? 401
- 잘못된 접근, 존재하지 않는 경로에 대한 접근에 주로사용되는 status 코드는 무엇일까요? 404
- 서버측 오류에 대해서 주로 사용되는 status 코드는 무엇일까요? 500

## 커스텀 뷰 만들기
init(frame:) 와 init(coder:) 생성자를 필수로 구현하고, 가운데에 내가 원하는 속성을 설정할 수 있는 이니셜라이저를 구현하면 된다. 
스택뷰, 레이블, 버튼 모두 동일한 형태로 구현가능하다.

```swift
class CustomStackView: UIStackView {
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    init(axis: NSLayoutConstraint.Axis, spacing: CGFloat, distribution: Distribution, alignment: Alignment) {
        super.init(frame: .zero)
        self.axis = axis
        self.spacing = spacing
        self.distribution = distribution
        self.alignment = alignment
    }
    
    required init(coder: NSCoder) {
        super.init(coder: coder)
    }
}
```

```swift
class CustomLabel: UILabel {
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    init(order: Int, type: Task) {
        super.init(frame: CGRect.zero)
        self.text = "\(order) - \(type)"
        self.textColor = type == .deposit ? .black : .purple
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
}
```

```swift
class CustomButton: UIButton {
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    init(title: String, textColor: UIColor) {
        super.init(frame: CGRect.zero)
        self.setTitle(title, for: .normal)
        self.setTitleColor(textColor, for: .normal)
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
}
```
이렇게 해준후 뷰컨에서 view.addArrangedSubview를 하면된다.

## 코드로 뷰짜기

커스텀뷰로 만든 버튼 두개를 스택뷰안에 넣기
- addTarget 로 액션메서드와 연결해주고
- 스택뷰에 addArrangedSubview를 하면된다
- 나중에 스택뷰도 view에 서브뷰로 등록하면 된다

```swift
    func setupButtons() {
        buttonStackView = CustomStackView(axis: .horizontal, spacing: .zero, distribution: .fillEqually, alignment: .center)
        let addCustomerButton = CustomButton(title: "고객 10명 추가", textColor: .blue)
        let resetButton = CustomButton(title: "초기화", textColor: .red)
        addCustomerButton.addTarget(self, action: #selector(setupCustomerQueue), for: .touchUpInside)
        
        [addCustomerButton, resetButton].forEach {
            buttonStackView?.addArrangedSubview($0)
        }
    }

    @objc func setupCustomerQueue() {
        bank?.setupCustomerQueue(with: 10)
    }
```

## 델리게이트 패턴
BankTimer 타입에서 작업에 걸린 시간을 재고나서 바로 UI에 업데이트해주는 대신,
TimerDelegate 프로토콜 타입인 delegate 프로퍼티를 가져서 TimerDelegate 프로토콜을 채택한 뷰컨에게 UI 업데이트하는 일을 위임함

UI에 직접 접근하여 업데이트 하는 대신
뷰컨의 setupLabel 메서드를 부르는 것이다. 
```swift
protocol TimerDelegate: AnyObject {
    func setupLabel(second: Int, milisecond: Int, nanoSecond: Int)
}

extension ViewController: TimerDelegate {
    func setupLabel(second: Int, milisecond: Int, nanoSecond: Int) {
        let formattedSecond = second < 10 ? "0\(second)" : "\(second)"
        processingTimeLabel?.text = "업무시간 - \(formattedSecond):\(milisecond):\(nanoSecond)"
    }
}

class BankTimer {
    //생략
    private weak var delegate: TimerDelegate?
    
    //생략
    @objc func timeUp() {
        //생략
        delegate?.setupLabel(second: second, milisecond: miliSecond, nanoSecond: nanoSecond)
    }
    //생략
}

```

## Git branch
- 로컬에서 새브랜치 만들고 원격으로 올리고싶을때
git branch 새로운브랜치
git checkout 새로운브랜치
git push --set-upstream origin 새로운브랜치
