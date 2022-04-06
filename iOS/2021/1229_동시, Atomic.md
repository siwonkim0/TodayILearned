TIL 1230

# 학습 내용
- '동시'의 의미
- Atomic

## '동시'의 의미
공유자원에 여러 스레드가 동시에 접근하는 상황에서 생긴 레이스컨디션을 해결하기 위해 객체를 thread-safe하게 만드는 방법에 대해서 찾아보다가 동시의 의미에 대해 의문이 생겼다.
동시가 진짜 동시일까? 아니면 빠르게 번갈아가며 접근하는것일까?

**다음은 리뷰어 찰리에게 질문을 드린 내용이다.**
저희가 문제를 해결하다보니 “동시”라는 부분에 대해 궁금한 점이 생겨서 질문드립니다!

![](https://i.imgur.com/Wz7ufdI.png)


우선 동시에 대해 다음과 같이 이해해봤습니다
논리적인 동시 == 여러 쓰레드가 빠르게 번갈아가며 접근하는 것
물리적인 동시 == 여러 쓰레드가 정말 ‘동시에’ 접근하는 것 

thread-safe 의 정의를 다시 한 번 확인해보니 아래와 같았습니다.
스레드 안전은 멀티 스레드 프로그래밍에서 일반적으로 어떤 함수나 변수혹은 객체가 여러 스레드로부터 “동시에” 접근이 이루어져도 프로그램의 실행에 문제가 없음을 뜻한다.

→ 가정1 에 따르면... 여기서 동시의 의미가 논리적인 동시인것인가요?
멀티 쓰레드 프로그래밍 환경에서 말하는 동시의 의미는 모두 논리적인 동시인것이라고 생각했습니다. 쓰레드는 실제로 동시에 작동할 수 없고 마치 동시에 처리되는 것 처럼 보이게 빠르게 작동만 하는 것이라 생각했습니다. 

 즉 그렇다면 thread-unsafe 상태 또한 논리적인 동시 상태에서 발생할 수 있는 문제로 볼 수 있을까요? 이에 공유 자원에 대한 쓰레드의 진입에 제한(semaphore, lock)을 두어서 thread-safe하게 만들어줬다고 이해해볼 수 있을까요?

→ 가정2 에 따르면... 만약 여기서 말하는 것이 물리적인 동시라면?
물리적인 동시를 말하는 것이라면 thread-unsafe한 상태가 물리적인 동시가 가능한 상태여서 문제가 발생하는 것이고,이를 논리적인 동시로 변경하여 thread-safe하게 만들어주는 것이 semaphore, Lock 등 인지 궁금합니다!

**찰리의 답은**
가정 1이 맞다고 하셨다.
**눈에 보이지 않는 속도로 스레드를 바꿔**가면서 일이 처리가 된다는 것이 **동시**의 의미이다. 

스레드가 일을 할 수 있으려면 해당 스레드가 활동할 수 있는 시간을 OS가 스케쥴링 해주어야 할텐데요.
그 스케쥴링을 동시에 한다는 것이 말이 안될 것 같구요
저기서 말하는 '동시' 라는 말은
A스레드가 스케쥴링되어 C Method에 접근했고 C 가 완료되기 전에
B스레드가 스케쥴링 되어 C메서드에 접근한 상황을
동시라고 할 것 같습니다.
이 상황에서는 A가 먼저 스케쥴링 되었다고 해서 A가 먼저 완료되는 것을 보장할 수도 없고요. 

'동시' 에 접근 했다고 해서 항상 오류가 발생한다고 말할 수도 없습니다.
C 메서드는 사실 기계어로 보았을 때 Critical Section이 더 좁은 범위일테니까요. 운좋게 걸리지 않을 수도 있어요. 
이를 '보장' 할 수 없으니 unsafe한 상태이고, 이를 해결하기 위해 우리는 논리레벨에서 제한을 한 것입니다.'

정확한 말은 아니지만, 초당 수행할 수 있는 요청의 양이라고 생각하면 쉬워요. 3.6 GHZ CPU는 대략 초당 36억 개의 명령어를 수행할 수 있는 것이고요.
그러면 명령어간 시간 차이는 1/36억 단위로 쪼개질 것이고요.
OS는 그 36억개의 리소스를 누구한테 줄지 판단해서 각 스레드에게 권한을 줍니다.
18억번째 스레드 1이 할당되어 일을 시작했고요. 19억번 째 요청에서 스레드 2가 할당되어 일을 시작했다고 가정했을 떄
정말 찰나일거에요.
찰나지만 정말 빠르기 떄문에 동시라고 할 수 있습니다.
동시는 위에서 말한 것 처럼 각 스레드가 특정 동작을 끝내기 전에 동시에 접근 했을 때....

## Atomic
공유자원에 여러 스레드가 동시에 접근하는 상황에서 생긴 레이스컨디션을 해결하기 위해 객체를 thread-safe하게 만드는 방법에 대해서 찾아보다가 리뷰어께서 제안해주셔서 객체를 Atomic하게 만드는 방법에 대해 학습해보았다.

property wrapper을 사용해서 Queue의 items 프로퍼티(linked list 타입)를 Atomic 으로 만들어주었다. property wrapper내부에 NSLock 객체를 활용하여 프로퍼티에 get이나 set 방식으로 접근할때 모두 lock을 걸어주는 방식으로 프로퍼티에 접근할 수 있는 스레드의 수를 제어했다.(고 생각했다...)

[참고한 링크](https://www.onswiftwings.com/posts/atomic-property-wrapper/)

```swift
@propertyWrapper
struct Atomic<Value> {

    private var value: Value
    private let lock = NSLock()

    init(wrappedValue value: Value) {
        self.value = value
    }

    var wrappedValue: Value {
      get { return load() }
      set { store(newValue: newValue) }
    }

    func load() -> Value {
        lock.lock()
        defer { lock.unlock() }
        return value
    }

    mutating func store(newValue: Value) {
        lock.lock()
        defer { lock.unlock() }
        value = newValue
    }
}

class Queue<Element> {
    @Atomic private var items = LinkedList<Element>()
}
```
```swift
class BankClerk {
    //코드 생략
    func work() {
        let workGroup = DispatchGroup()
        guard let customer = self.bank?.dequeue() else {
            return
        }
        
        workGroup.enter()
        self.processWork(of: customer, group: workGroup)
        workGroup.leave()
        
        workGroup.wait()
    }
    //코드 생략
}
```

```swift
class Bank: BankTransactionable {
    //코드 생략
    func open() {
            let group = DispatchGroup()
            let depositQueue = DispatchQueue(label: "deposit", attributes: .concurrent)
            let loanQueue = DispatchQueue(label: "loan")
            let semaphore = DispatchSemaphore(value: 2)
            
            while !customerQueue.isEmpty {
                switch customerQueue.peek()?.task {
                case .deposit:
                    depositQueue.async(group: group) {
                        semaphore.wait()
                        self.bankClerk.work()
                        semaphore.signal()
                    }
                case .loan:
                    loanQueue.async {
                        self.bankClerk.work()
                    }
                    
                default:
                    return
                }
            }
            
            group.wait()
        }
    
    //코드 생략
}
```

그러나 무슨 이유에서인지 코드에 적용을해도 레이스컨디션이 해결이되지않고 같은 고객 번호가 두번 뽑히는 현상이 아직도 발생하였다.. 

이 모든 상황이 peek() 메서드를 통해 고객큐에 어떤 업무를 들고있는 고객이 있는지 확인후에 실제 dequeue는 다른 큐에 보내서 하기 때문에 생겨난 것이라고 생각하고.. 

조건을 dequeue가 되는 것으로 변경했다. 
dequeue한 customer의 task 종류에따라 각각 해당하는 큐로 보내서 업무처리를 하도록 구현했다. 
```swift
    while let customer = customerQueue.dequeue() {
        switch customer.task {
        case .deposit:
            depositQueue.async(group: bankGroup) {
                semaphore.wait()
                self.bankClerk.work(with: customer)
                semaphore.signal()
            }
        case .loan:
            loanQueue.async(group: bankGroup) {
                self.bankClerk.work(with: customer)
            }
        }
        processedCustomers += 1
    }
```

사실 이렇게 로직을 구현하며 오류를 해결하지 못했던 공유자원에 접근하는 부분이 없도록 변경하였다. customerQueue에는 무조건 main 스레드가 한번만 접근하도록 한 것이다.
이 방법으로는 구현에 성공했지만, 각 큐가 dequeue하게하는 로직도 돌아가게 만들고 싶었는데 시간상 포기하게 되었다. 
