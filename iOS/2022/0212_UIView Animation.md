# UIView Animation

## 코드로 버튼과 이미지뷰로 이루어진 스택뷰를 구현해보았다.
### 깨달은 점
1. trailingAnchor과 bottomAnchor는 constant를 음수로 주기 잊지말기

```swift
func setupConstraints() {
    NSLayoutConstraint.activate([
        yagomImage.heightAnchor.constraint(equalTo: yagomImage.widthAnchor),
        totalStackView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 30),
        totalStackView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -30),
        totalStackView.topAnchor.constraint(equalTo: view.topAnchor, constant: 100),
        totalStackView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -100)])
}
```

2. stackView의 distribution이 filEqually 일때 

setContentHuggingPriority나 setContentCompressionResistancePriority를 설정해도 먹히지 않는다

이미지뷰와 버튼스택뷰 사이에 spacing을 많이 주고 이미지뷰는 가만히 있고 버튼스택뷰가 줄어들게 하고싶을 때

→ 만약 하고싶으면 distribution을 fil로 주고 hugging/resistance를 설정해야한다.

이렇게 

```swift
let yagomImage: UIImageView = {
    let image = UIImageView()
    image.image = UIImage(named: "yagom")
    **image.setContentCompressionResistancePriority(.init(rawValue: 1000), for: .vertical)
    image.setContentHuggingPriority(.init(rawValue: 1000), for: .vertical)**
    image.translatesAutoresizingMaskIntoConstraints = false
    return image
}()

let buttonStackView: UIStackView = {
    let stackView = UIStackView()
    stackView.alignment = .center
    stackView.axis = .vertical
    stackView.spacing = 1
    stackView.distribution = .fillEqually
    **stackView.setContentCompressionResistancePriority(.defaultLow, for: .vertical)
    stackView.setContentHuggingPriority(.defaultLow, for: .vertical)**
        stackView.translatesAutoresizingMaskIntoConstraints = false
    return stackView
}()

let totalStackView: UIStackView = {
    let stackView = UIStackView()
    stackView.alignment = .center
    stackView.axis = .vertical
    **stackView.spacing = 200**
    stackView.translatesAutoresizingMaskIntoConstraints = false
    stackView.distribution = .fill
    return stackView
}()
```
.
### 애니메이션까지 구현한 전체 코드

```swift
import UIKit

class ViewController: UIViewController {
    
    let yagomImage: UIImageView = {
        let image = UIImageView()
        image.image = UIImage(named: "yagom")
//        image.setContentCompressionResistancePriority(.init(rawValue: 1000), for: .vertical)
        image.setContentHuggingPriority(.init(rawValue: 1000), for: .vertical)
        image.translatesAutoresizingMaskIntoConstraints = false
        return image
    }()
    
    let stopButton: UIButton = {
        let button = UIButton()
        button.setTitle("STOP", for: .normal)
        button.setTitleColor(.black, for: .normal)
        button.addTarget(self, action: #selector(stopButtonTapped), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }()
    
    let macBookOwnerYagomButton: UIButton = {
        let button = UIButton()
        button.setTitle("맥북을 산 야곰", for: .normal)
        button.setTitleColor(.black, for: .normal)
        button.addTarget(self, action: #selector(macButtonTapped), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }()
    
    let errorCaughtYagomButton: UIButton = {
        let button = UIButton()
        button.setTitle("에러를 만난 야곰", for: .normal)
        button.setTitleColor(.black, for: .normal)
        button.addTarget(self, action: #selector(errorButtonTapped), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }()
    
    let buttonStackView: UIStackView = {
        let stackView = UIStackView()
        stackView.alignment = .center
        stackView.axis = .vertical
        stackView.spacing = 1
        stackView.translatesAutoresizingMaskIntoConstraints = false
        stackView.distribution = .fillEqually
//        stackView.setContentCompressionResistancePriority(.defaultLow, for: .vertical)
        stackView.setContentHuggingPriority(.defaultLow, for: .vertical)
        return stackView
    }()
    
    let totalStackView: UIStackView = {
        let stackView = UIStackView()
        stackView.alignment = .center
        stackView.axis = .vertical
        stackView.spacing = 100
        stackView.translatesAutoresizingMaskIntoConstraints = false
        stackView.distribution = .fill
        return stackView
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupViews()
        setupConstraints()
    }
    
    @objc func stopButtonTapped() {
        yagomImage.layer.removeAllAnimations()
    }
    
    @objc func macButtonTapped() {
        UIView.animateKeyframes(withDuration: 0.2, delay: 0, options: [.repeat, .autoreverse]) {
            UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 1) {
                self.yagomImage.transform = CGAffineTransform(scaleX: 1, y: 1.5)
            }
            UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 1) {
                self.yagomImage.transform = CGAffineTransform(scaleX: 1.1, y: 1)
            }
        } completion: { _ in
        }
    }
    
    @objc func errorButtonTapped() {
        UIView.animate(withDuration: 0.03, delay: 0.0, options: [.repeat, .autoreverse]) {
            self.yagomImage.center = CGPoint(x: self.yagomImage.center.x + 10, y: self.yagomImage.center.y)
        } completion: { _ in
            print(self.yagomImage.center)
        }

    }
    
    func setupViews() {
        view.addSubview(totalStackView)
        totalStackView.addArrangedSubview(yagomImage)
        totalStackView.addArrangedSubview(buttonStackView)
        buttonStackView.addArrangedSubview(stopButton)
        buttonStackView.addArrangedSubview(macBookOwnerYagomButton)
        buttonStackView.addArrangedSubview(errorCaughtYagomButton)
    }
    
    func setupConstraints() {
        NSLayoutConstraint.activate([
            yagomImage.heightAnchor.constraint(equalTo: yagomImage.widthAnchor),
            totalStackView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 30),
            totalStackView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -30),
            totalStackView.topAnchor.constraint(equalTo: view.topAnchor, constant: 100),
            totalStackView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -150)])
    }

}
```

### 애니메이션 실행: UIView의 타입 메서드 두개

- animate(withDuration:animations:) : 한가지 애니메이션
- animateKeyframes(withDuration:delay:options:animations:completion:) : 여러가지 애니메이션 연속적으로 가능 (겹치게도 가능)
    - addKeyframe(withRelativeStartTime:relativeDuration:animations:) : 이 메서드를 애니메이션 파라미터에 여러개 함께 넣을 수 있음

### 어떤 동작 할건지: UIView의 프로퍼티 두개

- center : 위치 지정시 이거 변경 (타입 CGPoint)
- transform : 크기나 회전시 이거 변경 (타입 CGAffineTransform)
