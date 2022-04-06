
# 학습 내용
은행창구매니저 step1 PR 리뷰, 동시성 공부함
- Unit Test
- LinkedList 의 removeAll 구현방식
- DispatchGroup
- Race Condition과 Thread Safe

## Unit Test
- 테스트 코드 작성시에는 실제 use case 와 비슷한 경우를 작성하는 것이 좋다 
"실제 UseCase와 비슷한 환경을 가진 테스트가 추가되면 좋을 것 같습니다. enqueue와 dequeue가 교차하며 호출되는 시나리오 등"
- 테스트 코드는 코드를 잘 짜는 것보다 무결성 관리가 더 중요하다
=> 강제 추출(!)이 필요할때는 사용해도 된다.
"테스트 코드는 스트릭하게 규칙을 정하지 않으셨으면 좋겠어요. 테스트 코드는 코드 퀄리티 보다 무결성 관리가 더 중요하니까요." 

## LinkedList 의 removeAll 구현방식
"head와 tail만 날리게 되면, head 의 next에 연결된 다른 node들은 해제 불가능한 trash가 되어버리지 않을까요." 라는 피드백을 받고 생각해보았다. 

~~이전 코드에서는 head 와 tail만 날리면 linkedList 전체가 사라진다고 생각했는데, node를 상수에 할당해줌으로써 각각의 node 인스턴스와 상수간에 참조를 하고 있고, 그 상황에서 head tail 에만 nil 을 넣어주면 가운데의 node들은 메모리에서 해제가되지 않는다. 따라서, head의 next가 nil이 될때까지 LinkedList를 while문으로 순회하며 모두 head에 nil을 넣어주었다.~~
-> 아니었음 
node를 상수에 할당해주는 부분은 함수 내에서 이루어지므로 함수의 실행이 끝나면 상수도 메모리에서 해제되므로 그냥 head 와 tail에 nil을 할당해주기만 하면 가운데의 노드들은 자동으로 메모리에서 해제된다. 
이전 코드
```swift
func removeAll() {
    head = nil
    tail = nil
}
```
개선했다고 생각한 코드
```swift
func removeAll() {
    while let next = head?.next {
        head = nil
        head = next
    }
    
    head = nil
    tail = nil
}
```

## DispatchGroup

비동기(async)적으로 처리되는 작업들을 그룹으로 묶어서 그룹 단위로 작업 상태를 추적할 수 있는 기능

그래서 그 그룹의 작업이 끝나는 시점을 추적해서 어떤 동작을 수행할 수 있는 것 

→ async 작업들이 꼭 같은 큐나 스레드에 있지 않아도 묶임 !!!

왜 DispatchGroup은 async에서만 사용가능하나?

당연하다. 동기로 처리되는 작업은 끝나는 시점을 예측가능함 (작업이 끝날때까지 기다리렸다가 다음 작업 수행해서) → 따로 작업 종료시점을 추적하지 않아도 됨

enter, leave 로 작업들을 그룹으로 지정후 notify나 wait을 사용한다.

## Race Condition과 Thread Safe
스위프트 대부분의 타입들은 thread unsafe해서 == 여러 스레드에서 동시에 접근이 가능해서

race condition이 발생한다

→ 해결 방법 

- DispatchSemaphore
- Serial Queue

### 1. DispatchSemaphore

꼭 thread unsafe 때문에 발생한 race condition 을 해결하기 위한 방법은 아니지만, 

공유 자원에 접근할 수 있는 스레드의 수를 제어해줘서 해결방안이 될 수 있는 것 

1st DispatchSemaphore 인스턴스 생성, 접근 가능한 스레드의 수 파라미터로 전달 

```swift
let semaphore = DispatchSemaphore(value: 1) // count = 1
```

2nd wait() 으로 값에 접근했다고 알리고, 작업이 끝나면 signal()로 볼일 다봤다고 알림 

```swift
DispatchQueue.global().async {
    semaphore.wait() // count -= 1

    semaphore.signal() // count += 1
}
```

### 2. **Serial Queue 활용**
Serial Queue를 통해 ****DispatchSemaphore에서 했던 것 처럼 `질서`를 만들어주기
커스텀 큐로 serial queue를 만들어준다.
global에 비동기로 보내고 나서 다시 커스텀 serial 큐로 동기 방식으로 보냄,,


