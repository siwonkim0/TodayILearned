# 학습 내용 
- LinkedList: 클래스와 구조체
- POP 

## LinkedList: 클래스와 구조체
은행창구 매니저 프로젝트 STEP1의 자료구조를 설계하다가 
LinkedList 안의 Node가 class일 때 LinkedList를 class 로 만들지 struct로 만들지 고민하였다.

이에 두가지 의문이 생겼는데,
1. class로 만들어진 classLinkedList를 복사해서 copyClass를 만들 경우 LinkedList 안의 Node는 어떻게 될까?

2. struct로 만들어진 structLinkedList를 복사해서 copyStruct를 만들 경우 LinkedList 안의 Node는 어떻게 될까?



이를 확인하기 위해 lldb를 이용하여 메모리 주소를 찍어보았다.

**1. LinkedList가 class 인 경우**
```swift
let classLinkedList = LinkedList<Int>()
classLinkedList.append(value: 1)
classLinkedList.append(value: 2)
classLinkedList.append(value: 3)

let copyClass: LinkedList<Int> = classLinkedList
```
![](https://i.imgur.com/masRF1q.png)
1. classLinkedList 자체의 메모리 주소값을 그대로 copyClass에 전달해주었다. 
2. 안의 Node들의 메모리 주소도 서로 동일하게 가지게 된다.


**2. LinkedList가 struct 인 경우**
```swift
var structLinkedList = LinkedListt<Int>()
structLinkedList.append(value: 1)
structLinkedList.append(value: 2)
structLinkedList.append(value: 3)

var copyStruct: LinkedListt<Int> = structLinkedList
```
![](https://i.imgur.com/0mKLtMI.png)
1. 구조체이기때문에 structLinkedList는 메모리 주소값이 없다.(변수가 스택 영역에 저장)
2. 하지만 안의 Node들의 메모리 주소는 structLinkedList와 copyStruct가 서로 동일하게 가지게 된다.

값타입의 경우 복사시 값만 복사된다고 알고 있었는데 값타입 안에 참조타입이 들어가 있으면 참조타입의 메모리주소가 복사되어 들어간다니...

이를 통해 구조체와 클래스를 겹쳐서 사용하게 되는 경우 구조체의 값타입을 값타입처럼 사용할 수 없게 된다는 사실을 확인했다.

이렇게 내부 참조타입의 메모리주소가 복사되어 들어가지만, 껍데기는 구조체이므로 원본과 다르게 관리가 된다. 

**copyClass와 copyStruct의 요소들을 변경했을 때**
따라서 copyStruct의 내부 값들을 모두 지워도 원본인 structLinkedList에는 영향을 주지 않는다.
```swift
copyStruct.removeAll()
```
![](https://i.imgur.com/kSepsyV.png)



반면, 내부요소와 껍데기 모두 클래스인 classLinkedList를 복사한 copyClass는 사실상 classLinkedList의 메모리 주소를 전달받은 것이 되기 때문에 copyClass의 내부 값들을 모두 지우면 원본인 classLinkedList의 내부 값들도 모두 지워진다. 
```swift
copyClass.removeAll()
```

![](https://i.imgur.com/i3cj4ie.png)

값타입내에 참조타입이 구현되어있으면 값타입의 값을 복사했을 때 안에있는 요소들은 원본과 같은 주소값을 가지고 있어서 값타입의 특성이 사라진다.

따라서, 값타입은 값타입과 함께, 참조타입은 참조타입과 함께 사용해야 각자의 특성을 유지할 수 있어 바람직하다는 결론을 내렸다. 

## POP : 프로토콜 지향 프로그래밍

### 상속의 단점
- 클래스에서만 사용가능
- 다중 상속이 불가능
- 부모 클래스의 기능을 모두 다 상속받아야해서 필요없는 기능까지 어쩔수 없이 가지고 있어야 하는 경우가 생길 수 있음
- 참조타입의 한계 -> 동적할당과 참조카운팅에 자원 소모

### 프로토콜을 사용함으로써 개선된점
- 클래스, 구조체, 열거형 모두에서 사용가능
- 여러 프로토콜을 동시에 채택가능
- 프로토콜 + 익스텐션으로 기본 구현 제공 가능
- 값타입을 사용 가능

## 프로토콜의 한계
- 저장 프로퍼티는 확장으로 기본구현 불가능
- Objective-C 의 프로토콜은 확장으로 기본구현 불가능
- 프로토콜 채택시 요구되는 기능을 모두 구현해야함(익스텐션으로 기본 구현 안해주면..) <-> 상속의 경우에는 자동으로 구현됨

#### 기억해둘 것 
상속은 수직확장, 프로토콜은 수평확장
프로토콜도 서로를 상속가능 
프로토콜 + 익스텐션 + 제네릭 좋은 조합 
상속과 프로토콜을 동시에 사용도 가능 

프로토콜에서 정의한 기능이고 확장에서 기본구현을 해줬지만 프로토콜을 채택한 곳에서도 구현하면 당연히 프로토콜을 채택한곳에서 구현된 메서드가 불린다.

그러나, 
프로토콜에서는 정의하지 않았는데 확장에서 구현해준 기능을 프로토콜을 채택한 곳에서 구현하면 인스턴스의 `타입에 따라` 확장의 메서드가 불릴지 프로토콜을 채택한 타입의 메서드가 불릴지 결정된다.

#### 느낀 점 
무조건 프로토콜이 상속보다 좋은 것이라고 생각하면 안됨
적절히 상황에 맞게 선택하고 둘을 같이 사용하는 방법도 있다는 것을 염두해두기
