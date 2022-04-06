# 학습내용
- 활동학습(프로토콜, 델리게이션)
- 의존모둠: 프로토콜 공식문서 읽기
- ARC: linked list에도 ARC가, weak와 unowned의 차이
- 활동학습 복습(프로토콜)


# 프로토콜

타입에 대한 청사진

받고 싶은 타입을 한정짓고 싶을 때 (일반화)

→ 물론 이런 목적에서만 쓰이는건 아니고 또 다른 목적들도 있다...  

### 예시

이건 안좋은 예시 : 열거형 사용 

열거형은 변하지 않을 것들을 쓸때 사용해야한다. 

: 열거형을 사용하는 코드가 이곳저곳 널려있을 때 열거형에 케이스가 추가되면 default를 통해 에러는 안나지만 코드가 망함 걍 어디서 잘못된지 못찾아서 

```swift
enum Customer {
    case camper(String), supporter(String)
    case reviewer, crew
}

func order(item: String, customer: Customer) {
    switch customer {
    case .camper(let name), .supporter(let name):
        print("\(name)가 \(item)을 주문함")
    default: //case가 추가되어도 다 여기로 빠져서 에러가 안나서 디버깅이 어려워짐 
        print("아몰랑")
    }
    
}
```

해결 방법: 프로토콜 사용

여러 타입을 하나의 타입으로 묶어줌... 

그래서 매개변수타입으로 프로토콜을 사용하면 그 프로토콜을 채택한 아무 타입이나 다 들어올 수 있는거 

```swift
protocol Customer {
    var name: String {get}
}

struct Supporter: Customer { //supporter을 추가해도 열거형이랑 다르게 기존 코드에 영향을 주지않음
    var name: String
}

let steven: Supporter = Supporter(name: "Steven")

struct Camper: Customer {
    var name: String
}

let ari: Camper = Camper(name: "Ari")
let cider: Camper = Camper(name: "cider")
```

```swift
protocol StoreItem: AnyObject { //AnyObject: 클래스만 채택가능하다는 명시
    var name: String { get }
    var stock: Int { get set }
}

class Towel: StoreItem {
    let name = "수건"
    var stock: Int = 100
}

class Shirt: StoreItem {
    let name = "반팔"
    var stock: Int = 100
}

let towel = Towel()
let shirt = Shirt()
```

```swift
//프로토콜도 타입으로 쓸 수 있어요: Customer이라는 요구사항을 지킨 애들은 다 들어올 수 있다.
//supporter 가 못사게 하고싶으면 supporter는 customer 프로토콜 채택 안하면된다
func order(item: StoreItem, customer: Customer) {
    guard item.stock > 0 else {
        print("재고없음 돌아가")
        return
    }
    item.stock -= 1
    print("\(customer.name)가 \(item.name)을 주문함")
}




print(order(item: towel, customer: ari))
print(order(item: shirt, customer: steven))
```

Towel과 Shirt를 Class 로 한 이유 : 프로토콜에 AnyObject라고 명시를해줘서 클래스만 채택가능하도록 제한해서 인스턴스의 프로퍼티를 변경 `item.stock -= 1` 할 수 있도록 해주기 위해. 

프로토콜끼리는 상속했다는 표현을 씀

다른 타입에서 프로토콜은 채택하는 것임


## ARC

### weak 
```swift
class Node {
    var value: String
    weak var head: Node? // weak를 사용해서 순환 참조 해결
    weak var tail: Node?
    
    init(value: String) {
        self.value = value
    }
    
    deinit {
        print("\(value)가 해제됨.")
    }
}

var namu: Node? = Node(value: "나무")
var jane: Node? = Node(value: "제인")
var hwang: Node? = Node(value: "허황")

namu?.tail = jane

jane?.head = namu
jane?.tail = hwang

hwang?.head = jane

jane = nil
namu?.tail?.tail?.value
```

weak를 써주지 않으면 순환참조가 일어나서 jane에 nil을 할당해도 메모리에서 해제가되지 않는다.

### unowned
unowned는 초기값이 nil일때만 옵셔널로 사용가능 

```swift
class Person {
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) 없어짐.")
    }
}

class Card {
    var name: String = "신한카드"
    unowned var owner: Person? = nil
}

var hwang: Person? = Person(name: "허황")
var card: Card? = Card()

card?.owner = hwang
hwang = nil // 참조하는 인스턴스가 먼저 메모리에서 해제될 가능성이 없는 경우

card?.owner // 에러 발생
```

카드의 주인이 있어야 사용가능한데 카드의 주인이 없으면 카드를 못쓰니까

카드의 주인이 카드보다 메모리에서 먼저 해제되지 않는다는 확신이 있을때 unowned사용

# 공부할것
- TDD 
- 델리게이션
