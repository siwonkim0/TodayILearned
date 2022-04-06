# 학습 내용
- 활동학습: Responder Chain / Touch Event
- DispatchGroup: wait 와 notify의 차이
- extension 접근제어자: private 와 fileprivate 

## Responder Chain / Touch Event
다양한 이벤트 ex.터치 이벤트 (`UITouch`) 발생시 

적절한 responder에게 보내고, 적절한 메서드 ex. `touchesBegan(_:with:)` 를 부른다.

그럼 responder는 적절한 action을 발생시킨다. 

### UIResponder

앱은 responder 객체를 이용해서 이벤트를 받고, 핸들링함

대표적인 responder 객체 (UIResponder 상속받음)
: UIView, UIViewController, UIApplication.. 

앱이 이벤트를 받으면 UiKit는 first responder 에게 바로 보낸다.
이벤트 타입에 따라 first responder가 달라진다

이 responder 들은 raw 이벤트 데이터를 받아서 핸들링하든지 다른 responder 객체에게 넘겨주든지 둘중하나 해야하는데 넘겨주는 순서도 정해져있다. 

이벤트 핸들링하기 위해서는 UIResponder에 구현되어있는touchesBegan, tochesEnded 등의 메서드들을 오버라이드해야함 
사용시 사용자가 어디서 터치를 시작하고 어디서 끝냈는지 알수있음 

### UIEvent

유저 인터렉션을 설명하는 객체
앱은 다양한 종류의 이벤트를 받아들일수 있음 
이벤트를 수신하고나서 이벤트를 무시할거냐 처리할거냐는 개발자가 정할수있음

우리가 받아들일수있는 이벤트의 종류로는? 

: **UIEvent 의 종류는 크게 네가지**

`터치, 모션, remoteControl, press` 
(각각 subtype들도 있음)


### UITouuch

사용자가 스크린에서 터치한 위치, 사이즈, 세기까지 나타내는 객체임

UItouch 는 UIEvent 안에 항상 속해있는 객체이다

### touches, presses, gestures

터치는 이벤트에 속해있고,, 
제스처는 왜 필요할까?
: 사용자가 터치를 하는 정형화된 방식들이 있는데 우리가 그것을 인지를 쉽게할수있게 미리 방식들을 정의해놓았음
사용자가 어떤 제스처를 줚는지 뷰를 통해 계산하기는 어렵기때문에.. 정형화된 제스처들을 미리 구현해두고 이거 가져다가 너네가 제스처 인식하고싶으면 그때써라고 하는것임

터치를 직접적으로 track도 가능하고, 제스처인식기 (UITapGestureRecognizer)로도 델리게이션이나 타깃-액션 방식으로 처리할 수도 있다.

## DispatchGroup: wait 와 notify의 차이
DispatchGroup은 비동기적으로 처리되는 작업들을 그룹으로 묶어서 그룹의 작업 종료 시점을 추적할 수 있는 기능

Queue는 serial이든 concurrent이든 상관없다.
작업들이 꼭 같은 스레드에 있지 않아도 같이 묶어줄 수 있다. 

비동기적으로 처리되는 작업들만 추적하는 이유는 동기적으로 처리되는 작업은 작업 처리완료를 기다리기때문에 추적할 필요조차 없기 때문

그래서 그룹의 작업이 끝나고 나서 처리해줄 수 있는 방법은 두가지가 있는데, wait 와 notify 이다. 

처음에는 wait는 작업이 끝날때까지 그냥 기다려주는거고 notify는 작업이 끝날때까지 기다리는건 똑같은데 기다린 후에 수행할 작업도 같이 파라미터로 전달해줄 수 있다는 부분만 다른 줄 알았다.

그러나 코드로 실험을 해보니 wait의 경우 동기적으로 실행이 되어 앞의 모든 작업이 끝난 후 실행이 되고,

notify의 경우 비동기적으로 실행되고 그룹에 추가된 비동기적으로 실행되는 코드들`만`이 모두 끝난 후 실행되는 것을 확인했다. 

wait는 메인 스레드를 잠재우니까 유저에게 보여지는 인터페이스를 freeze 하기때문에 메인 스레드에서 사용하지 않아야한다..
notify를 사용하거나, 백그라운드 스레드에서 사용해야한다.


### wait 사용시
```swift
let red = DispatchWorkItem {
    for _ in 1...3 {
        print("🥵🥵🥵🥵🥵")
        sleep(1)
    }
}

let blue = DispatchWorkItem {
    for _ in 1...3 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

group.wait()
print("모든 작업이 끝났습니다.")
print("aa")
```
![](https://i.imgur.com/biRay3q.png)
동기적으로 동작하여 그룹의 모든 코드들이 실행될때까지 기다리는 동안 메인의 코드는 실행되지 못한다. 메인이 freeze 하기때문
그래서.. 차례대로 실행된다.

### Notify 사용시
```swift
let red = DispatchWorkItem {
    for _ in 1...3 {
        print("🥵🥵🥵🥵🥵")
        sleep(1)
    }
}

let blue = DispatchWorkItem {
    for _ in 1...3 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

group.notify(queue: .main) {
    print("모든 작업이 끝났습니다.")
}
print("aa")
// bankWorkGroup.wait()
bankWorkGroup.notify(queue: .main, execute: a)
```

![](https://i.imgur.com/BVE0hmf.png)
비동기적으로 동작하여 그룹에 등록된 코드들만이 다 끝날때 실행된다. 근데 메인큐는 그룹에 없는 코드들이 실행되든 말든 관심이없다.
따라서.. 메인에서 동작하는 print("aa")가 먼저 실행된다. 


Reference
[Swift: DispatchGroup](https://blog.vishalvshekkar.com/swift-dispatchgroup-an-effortless-way-to-handle-unrelated-asynchronous-operations-together-5d4d50b570c6)

## extension 접근제어자: private 와 fileprivate
재사용하기 어려운 extension의 경우 다른 파일에서는 인덱싱되지 않도록 사용하는 곳에서 private extension으로 구현하라는 피드백을 받았다.

구현했던 double extension을 private extension으로도 해보고 fileprivate extension으로도 변경해보았지만 

private 이나 fileprivate 이나 같은 파일 내에서는 extension의 기능을 동일하게 사용할 수 있었고...

private의 경우 정의된 scope내에서만 기능을 사용할 수 있다고 알고있었는데 한 파일 안에서 다른 scope에서도 기능 사용이 어떻게 가능한 것인지 의문점이 들었다. 

또한, private extension 으로 접근제어자를 설정해주었는데 다른 파일에서 사용하려고 하면 fileprivate 레벨이라서 사용이 불가능하다는 에러 메세지가 뜨는데, 이런 경우 private와 fileprivate 를 동일하게 취급하기 때문에 그런건지도 궁금했다. 

같은 파일에서 정의했다면 scope가 해당 파일이 되고.. 이때 
private과 fileprivate은 동일하게 취급된다고 한다

