# 토요스터디 예제
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220307/622614f30983dd14920a2a02.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220307/622614f30983dd14920a2a02.png)

## Relay
PublishRelay와 BehaviorRelay는 PublishSubject과 BehaviorSubject를 래핑해서 가지고 있음

### 둘의 차이점은

- PublishRelay

구독한 subject에서 구독 이전에 발생한 이벤트들에 대해서는 알 수 없고, 구독 이후에 발생하는 이벤트들만 알 수 있음

- BehaviorRelay

구독한 subject 이전에 발생한 이벤트라도, subject를 선언할 때 설정해두었던 버퍼사이즈만큼의 이벤트는 불러옴

### **Subject 자체로써 쓰지 않고, 한번더 래핑하여 Relay의 형태로 사용하는 이유**

Subject의 경우 complete나 error를 맞이할 수 있다. 

하지만 ~Relay의 경우 complete나 error를 받지 않으므로 view에 사용하기 적합하다

### BehaviorRelay

`BehaviorRelay`는 variable 처럼 `BehaviorSubject`의 현재값을 알 수 있는 `value`라는 프로퍼티 가진다.

.value 로 현재 값을 가져오기 가능 

```swift
class UpDownGame {
    var randomValue = BehaviorRelay<Int>(value: 0)
    var tryCount = BehaviorRelay<Int>(value: 0)
    var urlSession: URLSessionProtocol
   
    ...
    func compareValue(with hitNumber: Int) -> HitResult {
        if tryCount.value >= 5 {
           return .Lose
        } else if **randomValue.value** == hitNumber {
            return .Win
        } else if hitNumber > **randomValue.value** {
            return .Down
        } else {
            return .Up
        }
    }
}
```

### subscribe와 bind 동일한 기능

슬라이더의 밸류가 바뀌면 바뀐 슬라이더밸류로 레이블의 텍스트를 바꿔라

- UIKit

```swift
@IBAction func slideValueChanged(_ sender: UISlider) {
        let integerValue = Int(sender.value)
        sliderValueLabel.text = String(integerValue)
    }
```

- subscribe, bind

```swift
slider.rx.value //슬라이더의 밸류가 바뀌면 바뀐 슬라이더밸류로 레이블의 텍스트를 바꿔라
    .map { "\(Int($0))"} // value가 넘어옴 : 타입은 Float
    .bind(to: sliderValueLabel.rx.text)

slider.rx.value
    .subscribe(onNext: {
        sliderValueLabel.text = "\(Int($0))"
    })
```

## 정의 부분
subscribe의 경우 onNext에 실행할 Action을 쓰는 반면
bind의 경우 이벤트를 받을 Observers를 쓰면 된다

### subscribe

```swift
public func subscribe<Object: AnyObject>(
    with object: Object,
    onNext: ((Object, Element) -> Void)? = nil,
    onError: ((Object, Swift.Error) -> Void)? = nil,
    onCompleted: ((Object) -> Void)? = nil,
    onDisposed: ((Object) -> Void)? = nil
) -> Disposable {
    subscribe(
        onNext: { [weak object] in
            guard let object = object else { return }
            onNext?(object, $0)
        },
        onError: { [weak object] in
            guard let object = object else { return }
            onError?(object, $0)
        },
        onCompleted: { [weak object] in
            guard let object = object else { return }
            onCompleted?(object)
        },
        onDisposed: { [weak object] in
            guard let object = object else { return }
            onDisposed?(object)
        }
    )
}
```

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220307/6226146d0983dd14920a29ff.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220307/6226146d0983dd14920a29ff.png)

### bind

```swift
/**
 Creates new subscription and sends elements to observer(s).
 In this form, it's equivalent to the `subscribe` method, but it better conveys intent, and enables
 writing more consistent binding code.
 - parameter observers: **Observers to receives events.**
 - returns: Disposable object that can be used to unsubscribe the observers.
 */
public func bind<Observer: ObserverType>(to observers: Observer...) -> Disposable where Observer.Element == Element? {
    self.map { $0 as Element? }
        .subscribe { event in
            observers.forEach { $0.on(event) }
        }
}
```
