## 학습 내용
- 클로저

```swift
//MARK: - 함수 타입이란?
//클로저는 함수를 비롯해서 모든 동작을 나타내는 요소

func addNumbers(_ first: Int, _ second: Int) -> Int {
    return first + second
}

func sayHello() {
    print("hello")
}

// 3가지 요소가 필요하다
// 매개변수 개수 + 매개변수 타입 + 리턴 타입

func subtract(_ first: Int, _ second: Int) -> Int {
    return first + second
}

//함수를 묶을 수 있다! 함수도 추상화된 타입으로 표현할 수 있다.
let functions: [(Int, Int) -> Int] = [addNumbers(_:_:), subtract(_:_:)]

//MARK: - 함수와 클로저 사이의 관계 (함수는 결국 이름 붙은 클로저이다)
//Closure가 큰 개념, function은 이름 있는 Closure의 한 종류

let myClosure: () -> Void = {
    print("안녕?")
}
//클로저 호출하여 실행
myClosure() //안녕?

let myClosuree: () -> Void = sayHello //타입만 맞으면 이 변수에 다른 클로저를 넣어도됨

//MARK: - 기본 클로저 문법


func addNumbers2(_ first: Int, _ second: Int) -> Int {
    return first + second
}

let addNumberSecond: (Int, Int) -> Int = { (first: Int, second: Int) -> Int in
    return first + second
}

addNumberSecond(1,2)

//MARK: - 일급 객체
//변수나 상수는 일급 객체이다
//클로저는 일급 객체이다
//변수 상수가 들어갈 수 있는 곳에 클로저도 들어갈 수 있다

// 이 함수가 호출되는 시점에서 어떤 연산을 할 지 정해줄 수 있다.
// 세개의 파라미터를 받음: 정수, 정수, 클로저(어떻게 연산할지에 대한)
func operateNumbers(first: Int, second: Int, operateHandler: (Int, Int) -> Int) -> Int {
    return operateHandler(first, second) //너가 뭐하는 애인진 모르겠지만 이때 실행할거야!
}
//하나의 함수인데 어떤 클로저를 넣으냐에 따라 다른 일을 하게 할 수도 있음
operateNumbers(first: 1, second: 2, operateHandler: addNumbers(_:_:))
operateNumbers(first: 2, second: 1, operateHandler: subtract(_:_:))

operateNumbers(first: 5, second: 5, operateHandler: { (a: Int, b: Int) -> Int in return a * b })

operateNumbers(first: 6, second: 7) { $0 * $1 }

//MARK: - 시키는건 다하는 로봇 만들기

struct Robot {
    func actAnything(action: () -> Void) {
        action() //함수 실행한다
    }
}
let robot = Robot()
robot.actAnything(action: sayHello) //()가 안붙는 이유: sayHello를 호출해주는 시점이 아니라 뭔지 알려주는 부분
robot.actAnything(action: { print("안녕하세요") })

//MARK: - 다양한 스킬을 사용하는 게임 캐릭터 만들기

struct Character {
    let 흑염룡: () -> String = {
        return "내 손의 흑염룡이 깨어난다"
    }
    let 백염룡: () -> String = {
        return "내 손의 백염룡이 깨어난다"
    }
    
    func shiftKeySkill(closure: () -> String) -> String {
        closure() //함수가 정의되어있지 않은 상태에서 호출
    }
}

let coda = Character()
print(coda.shiftKeySkill(closure: coda.백염룡)) //뭐할지 미리 정의해놓고 갈아끼울 수 있음 - 호출부에서 정의를 해야함

//MARK: - 함수타입 작성 / 정의부 / 호출부
//보통 기존의 함수는 정의를 하고 (정의된 함수를)호출하지만,
//클로저를 함수의 파라미터로 사용시, 정의되지 않고 그 타입만 알고있는 상태로 호출이 가능하다. 클로저가 품고 있는 함수를 호출할 때 정의해준다.


//MARK: - 클로저 축약

//인터넷작업이 끝나고 서버에서 뭘 받아왔을 때, 받아온 후에 뭘 할지 클로저가해줌
```
