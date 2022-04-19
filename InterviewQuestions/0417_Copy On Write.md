# Copy On Write를 사용하는 이유

값타입의 경우 변수에 할당하거나 함수의 파라미터로 넘길 때 할당된 값이 복사가 된다.

그러나 만약 아주 큰 값타입을 다룰때는 복사를 한다면 비용이 매우 많이 들 것이다.

따라서 이러한 이슈를 최소화하기 위해 특정 값타입의 경우 수정이 일어나거나, 참조가 한번 이상 될때만 복사가 되도록하여 성능을 개선하였다.

다시 말해서 수정 (쓰기)이 일어날 때 복사한다는 뜻으로,
단순히 값을 전달할 때는 복사가 아니라 기존 값의 참조를 통해 불필요한 복사를 줄여서 메모리를 절약할 수 있다.

작동 예시를 보자.
변수 array에 담긴 배열을 array2에 할당할때는 주소값이 같은데,
array2를 수정하니 그제서야 array2의 주소값이 달라진다.

```swift
var array = [1,2,3] 
var array2 = array
address(o1: &array) //0x100517000 
address(o1: &array2) //0x100517000 

array2.append(4) // array2가 변경 
address(o1: &array) //0x100517000 
address(o1: &array2) //0x102b04330
```

- 주소 구하는 함수 adress
    
    ```swift
    func address(o1: UnsafeRawPointer) {
        let address = String(format: "%p", Int(bitPattern: o1))
        print(address)
    
    }
    func address<T: AnyObject>(o2: UnsafePointer<T>) {
        let address = String(format: "%p", Int(bitPattern: o2))
        print(o2.pointee)
        print(address)
    
        withUnsafePointer(to: T.self) { pointer in
            pointer.pointee
            pointer
        }
    }
    ```
    

## 꼬리 질문

### 모든 값타입의 경우 Copy On Write를 지원하나요?

NO

Swift Standard Library에 정의된 값타입인 **컬렉션타입 (Array, Dictionary, Set)**의 경우에만 COW가 구현되어있다. 사용자 정의 값타입의 경우에는 지원하지 않는데, 이때 사용자가 직접 COW를 구현할 수 있다.

### 사용자 정의 구조체에서 Copy On Write를 구현하는 방법에 대해 설명해보세요.

```swift
final class Ref<T> {
  var val : T
  init(_ v : T) {val = v}
}

struct Box<T> {
    var ref : Ref<T>
    init(_ x : T) { ref = Ref(x) }

    var value: T {
        get {
            return ref.val
        }
        set {
            if (!isKnownUniquelyReferenced(&ref)) {
                //참조카운트 1이 아니라면 (=여러곳에서 참조되고 있다면)
                print(isKnownUniquelyReferenced(&ref))
            ref = Ref(newValue)
                print("새로운 인스턴스 생성")
            return
          }
          ref.val = newValue
        }
    }
}

//주소 구하는 함수
func address(o1: UnsafeRawPointer) {
    let address = String(format: "%p", Int(bitPattern: o1))
    print(address)

}
```

```swift
var box = Box<[Int]>([1,2,3])
var copy = box
address(o1: &box.value) //0x10072bda0
address(o1: &copy.value) //0x10072bda0

box.value = [3]
address(o1: &box.value) //**0x10072c1c0**
address(o1: &copy.value) //0x10072bda0
```

1. 제네릭 타입의 프로퍼티를 가진 class 타입을 정의한다.
2. 위에서 정의한 class 타입의 프로퍼티를 가진 struct를 정의한다.
3. 그리고 struct에 value라는 계산 프로퍼티를 정의하여 getter과 setter을 가지게 한다.
4. getter이 불린 경우에는 그대로 값을 전달하고
5. setter이 불린 경우에는 내장 전역 함수인 isKnownUniquelyReferenced 함수를 사용하여
    
    해당 객체의 참조 카운트가 1이면 true, 1보다 크면 false를 반환한다.
    - 객체의 참조 카운트가 2개 이상이면 다른 곳에서 사용되고 있다는 뜻이니까 새로운 복사본을 만들어서 수정도 같이 진행한다.
    - 객체의 참조 카운트가 1이면 그냥 수정한다.

### COW 한계

첫 수정 작업을 할 때는 두 세번째보다 실행 시간이 더 걸린다 → 오버헤드가 발생함

### 의문점

- 꼭 구조체의 프로퍼티가 클래스로 구현되지 않더라도 주소값이 같네. 뭐냐 대체?
- 이렇게 구조체 안에 클래스를 넣어서 구현할 바에는 차라리 클래스로 구현하는게 낫지 않나?

# Reference

[https://medium.com/@lucianoalmeida1/understanding-swift-copy-on-write-mechanisms-52ac31d68f2f](https://medium.com/@lucianoalmeida1/understanding-swift-copy-on-write-mechanisms-52ac31d68f2f)

[https://github.com/apple/swift/blob/main/docs/OptimizationTips.rst#id28](https://github.com/apple/swift/blob/main/docs/OptimizationTips.rst#id28)
