# 학습내용

### flatMap
매핑의 결과가 함수객체와 같은 컨텍스트를 반환하는 함수객체
```swift
func doubledEven(_ num: Int) -> Int? {
    if num.isMultiple(of:2) {
        return num * 2
    }
    return nil
}

print(Optional(3).flatMap(doubledEven)) //nil
```

2의 배수이면 2를 곱해서 리턴하고, 아니면 nil을 반환하는 함수 doubledEven을 Optional(3)의 flatMap에 전달하면 nil이 나온다. 

```swift
print(Optional(3).map(doubledEven)) //Optional(nil)
```
하지만 그자리에 map을 사용하면 Optional(nil)이 나온다. 왜지?

플랫맵은 맵과는 다르게 컨텍스트 내부의 컨텍스트를 모두 같은 위상으로 만들어주기 때문이다. 

compactMap는 시퀀스 타입이 옵셔널 타입의 element를 포장한 경우에 flatMap 대신 사용한다. 이 경우 말고는 모두 flatMap이라는 이름을 사용한다.

`Optional(3).compactMap(doubledEven) //에러`
그래서 이렇게 사용하려하면 에러임

### compactMap
map만 사용하는 경우 이중 컨테이너의 형태를 띄고 있지만, compactMap(flatMap)을 사용하는 경우 알아서 내부 컨테어너의 값을 추출해서 내부의 값을 1차원적으로 펼쳐놓는다.
따라서, 여러 컨테이너의 값을 연달아서 처리할 때 유용하게 사용할 수 있다. 값이 nil인 경우 별도의 예외처리 없이 빈 컨테이너를 반환해서 그 이후에 호출되는 메서드는 무시된다. (마치 옵셔널 체이닝느낌)

```swift
let optionals: [Int?] = [1,2, nil, 5]

let mapped: [Int?] = optionals.map{$0}
let compactMapped: [Int] = optionals.compactMap{$0}

print(mapped) //[Optional(1), Optional(2), nil, Optional(5)]
print(compactMapped) //[1, 2, 5]

# 학습 내용
- 스터디에서 고차함수 map, reduce, filter의 기능을 구현해봤고 
- flatMap 에 대해 알아봤다

1. 실행의 의미인 괄호()

```swift
class Character {
    let name: String = "AA"
    let runningSkill: () -> Void = {print("달린다")} // \(self)는 왜 안될까?
    lazy var runningSkill2: () -> Void = {print("\(self.name)달린다")} // lazy 로 바꾸면 가능: 왜?
    let healingSkill: () -> Void = {print("치료한다")}
    let walkingSkill: () -> Void = {print("걷는다")}
    let protectingSkill: () -> Void = {print("보호한다")}
}

func tabShiftKey(skill: () -> Void) {
    skill() //실행해준다.
}

let dog = Character()
tabShiftKey(skill: dog.healingSkill)
tabShiftKey { dog.healingSkill() } //클로저를 전달하면 실행()의 의미인 괄호를 써줘야한다
```

함수를 함수의 매개변수로 전달할때는 () 안써도 되지만, 클로저 형태로 전달할 경우에 () 써야한다. 

2. lazy 키워드와 타입 내의 self 키워드 사용

```swift
class Character {
    typealias Skill = () -> Void
    let name: String
    init(name: String) {
        self.name = name
    }
    
    lazy var 휘두르기2: Skill = { print("\(**self.name**) 휘두르기!") }
    lazy var 휘두르기: Skill = tabShiftKeyWithName
    let 순간이동: Skill = { print("순간 이동!") }
    let 디버깅: Skill = { print("LLDB!") }
    
    func tabShiftKey(skill: () -> Void) {
        skill()
    }
    func tabShiftKeyWithName() {
        print(name,"휘두르기!")
    }
}
let steven = Character(name: "스티븐")
steven.tabShiftKey(skill: steven.휘두르기)
steven.tabShiftKey(skill: steven.휘두르기2)
steven.tabShiftKey(skill: steven.순간이동)
steven.tabShiftKey(skill: steven.디버깅)
```

타입의 저장속성에 또다른 저장속성을 가져오려면 `lazy`키워드를 사용해야한다.

왜? 인스턴스가 초기화되고나서야 인스턴스의 저장속성을 가져올 수 있어서 

3. flatMap : 명시해준 타입 따라서 배열 안의 요소를 모두 평평하게 펼쳐주는 애 

→ flatMap과 compactMap 의 차이??

둘이 완전 동일한앤줄 알았는데 코드 구현방식이 다름 

```swift
//Collection
let flatMapCollection1: [Int] = [[1, 2], [3, 4]].flatMap { $0 } //[1, 2, 3, 4] <- 2차원 배열을 1차원 배열로 flat하게 만들어줌

print("***2")
let flatMapCollection2 = [[1, 2], nil, [3, 4]].flatMap { $0 } //DEPRECATED(use compactMap): [[1, 2], [3, 4]]
print(flatMapCollection2) // [[1, 2], [3, 4]] - nil 배제

// 주의 - 상수의 타입 지정에 따라 결과가 다름!
print("***31")
let flatMapCollection31 = [[1, nil, 2], [3, 4]].flatMap { $0 } // 타입 지정 안했을 경우
print(flatMapCollection31) // [Optional(1), nil, Optional(2), Optional(3), Optional(4)] - nil 배제되지 않음 (가장 밖의 포장만 풀어서 nil을 없앰)

print("***32")
let flatMapCollection32: [Int?] = [[1, nil, 2], [3, 4]].flatMap { $0 }
print(flatMapCollection32) // [Optional(1), nil, Optional(2), Optional(3), Optional(4)]

print("***33")
let flatMapCollection33: [Int?] = [[1, nil, 2], [3, 4]].flatMap { $0 }.flatMap { $0 }
print(flatMapCollection33) // [Optional(1), nil, Optional(2), Optional(3), Optional(4)] - flatMap을 연속 2번해도 nil이 남아있음

print("***34")
let flatMapCollection34: [[Int?]] = [[1, nil, 2], [3, 4]].flatMap { $0 }
print(flatMapCollection34) // [[Optional(1), nil, Optional(2)], [Optional(3), Optional(4)]] - 초기 배열 타입으로 지정하면 동일한 배열이 반환됨
```

4. map의 기능 직접 구현

```Swift
func stringConverter(recieve: Int) -> String {
    return String(recieve)
}

func myMap<S,T>(_ inputArray: [S], _ converter: (S) -> T) -> [T] {
    var resultArray: [T] = []
    for element in inputArray {
        resultArray.append(converter(element))
    }
    return resultArray
}

let myMapA = myMap([1,2,3], stringConverter(recieve:))
print(myMapA)

let myMapB = myMap(["1", "2"]) { element in
    Int(element)!
}
print(myMapB)
```
