# 학습 내용
- TDD 예제
- 계산기 프로젝트 step 1

# 학습 키워드
- TDD
- 제네릭
- Queue의 시간복잡도


## TDD

테스트를 작성하면서 코드를 완성시키는 방법론
테스트가 가능한 코드를 만들기 위해서는 테스트 코드를 염두해두고 코드를 작성해야한다. 
무턱대고 기능 구현을 했다가는 테스트가 불가능한 코드가 될 수도 있다. 

#### 방법 3단계
1. 실패하는 테스트 작성
2. 테스트 통과위한 최소한의 변경 -> 테스트 성공
3. 테스트 성공 유지하며 코드 개선

장점: 안전하고 의존성이 낮은 코드
단점: 개발 속도.... 

### Code Coverage

- 실제 테스트에서 어떤 코드가 실행되었는지
- 정확성, 성능에 대해 얼마나 충분히 테스트가 이루어졌는지
- 테스트가 포함하고 있지 않은 코드는 무엇인지

를 알 수 있다.

이런 기능까지 엑스코드에 있다는 사실이 놀라웠다..
엑스코드 개발은 어떤 천재들이 했을까...?

## 제네릭

스위프트 표준 라이브러리의 대부분은 제네릭으로 작성되어 있다.
지금까지 사용한 배열, 딕셔너리.. 도 Generic collection

예제를 보자.

```swift=
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```
두 정수의 값을 바꿔주는 함수를 구현하였는데,
정수뿐만 아니라 실수나 문자열로도 하고싶다면...

각각 구현해주는 것 대신에 제네릭을 사용해서 함수를 실행할 때 타입을 결정해줄 수 있다. 

```swift=
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```
저 T는 타입 파라미터라고 불린다.

### 타입제약

모든 타입이 아니라 특정한 타입만 사용 가능하게 하고싶을 때
타입 파라미터를 특정 클래스를 상속받게하거나, 특정 프로토콜을 준수하게 하는 제약을 걸 수 있다.

예를 들어, 딕셔너리의 key에는 Hashable이라는 프로토콜을 준수하고 있는 애들만 들어올 수 있다. 
그래서 지금까지 사용해왔던 Int, String, Double.. 등등은 그 프로토콜을 준수하고 있는 것이다.

```swift=
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```

예제2
배열에서 찾는 값이 몇번째 인덱스에 있는지 반환해주는 함수
값의 타입을 지정해주지 않아서 아무 타입이나 들어올 수 있음

근데, 모든 타입이 다 == 로 비교가능하진 않아서 에러가 생김
따라서 제네릭에 Equatable 프로토콜을 준수하는 애들만 들어올 수 있다고 타입제약을 걸어주면 사용 가능하다.
```swift=
func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

## Queue 의 시간복잡도

queue의 특성상 dequeue시에 시간복잡도가 O(n)으로 오버헤드 가 발생한다. 

원래는 queue 를 구현하는 과정에서 removefirst()로 맨 처음 집어넣은 요소를 빼주려고 했다.

그러나 removefirst()는 맨 앞자리가 빠지면 뒤에 애들이 한칸씩 앞으로 와야해서 시간복잡도는 O(n)이 된다...

cf. 배열에서 맨 마지막 요소를 삭제하는 removeLast()는 자기 자신만 삭제되어서 시간복잡도가 O(1)로 작다.

## 의문점 - 해결방향..
- queue는 맨 마지막에 입력한걸 바로 지우는 기능이 없는데(FIFO)라서 계산기의 CE는 어떻게 구현하지?
-> 숫자 + 부호까지 입력이 되어애 비로소 queue에 넣고, 그 전까지는 임시값 형태로 들고있어서 확실해지면 넣기
