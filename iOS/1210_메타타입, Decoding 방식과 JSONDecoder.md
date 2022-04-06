# 학습 내용
- 메타타입
- Encoding and Decoding

## 메타타입

- Self는 언제 어떤 의미를 나타내나요? (deocde가 누구의 메서드이고, 그 누구에서 Self는 무엇을 의미할까요?)
    - `Self` 구체적인 타입이 아닌 class, structure, enum 등의 타입을 선언부 안쪽에서 현재 어떤 타입인지 편리하게 지칭할 수 있는 수단입니다.
    - decode 메서드는 Foundation 의 JSONDecoder 클래스에 구현된 인스턴스 메서드 입니다.
    - 따라서 decode 메서드 입장에서 Self 는 JSONDeccoder 타입을 의미합니다.
    
- 제네릭 인스턴스 메서드인 decode에서 언급해주신 T.Type이 의미하는 것은 무엇인가요?
    - decode 메서드의 T.Type 타입으로 받아오는 type 인자는 공급받은 JSON 객체로 부터 디코딩할 값의 타입을 지정해주는 역할을 합니다.
    - T는 decode가 제네릭 인스턴스 메서드이기 때문에 함수가 사용되는 시점에서 T가 의미하는 타입이 결정되도록하는 타입 파라미터입니다.
    - 클래스, 구조체, 열거형 타입의 메타 타입은 그 타입의 이름이며 `.Type` 이 뒤에 따릅니다.
    - 따라서 메타 타입이 들어와야 하기 때문에 변환시키고자하는 타입의 메타타입을 넘겨줘야합니다.
    - 따라서 Exposition.self, ExpositionItem.self 와 같은 코드를 통해 메타 타입을 얻어서 decoder 의 인자로 넣어줍니다.
    
- `GroceryProduct.self`에서 GroceryProduct는 타입이죠!? `타입.self`가 나타내는 것은 무엇일까요?
    
    메타타입입니다. 
    
- 메타타입은 어떻게 얻을 수 있을까요?
    - 타입.self 사용
    - 인스턴스의 다이나믹 타입을 알려주는 type(of:) 메서드 사용
    

다이나믹 타입 = 런타임 타입

스태틱 타입 = 컴파일 타입

type(of:) 메서드는 인스턴스의 다이나믹 타입을 알려줌

`Self` 

- 타입 내부(선언부)에서 자기의 타입을 대신 지칭할 때

`self`

- 타입 내부(선언부)에서 자기 타입의 인스턴스를 지칭할 때
- 타입 선언부 밖에서 메타 타입을 "값"으로 접근하기 위해서 사용

메타타입을 학습하기 전에 알고 있었던 `self` 와 `Self` 키워드의 개념은

self는 타입의 인스턴스를 지칭하고, Self는 타입 자체를 지칭한다는 것이었습니다.

그러나 메타타입은 타입.self 를 하면 그 타입의 인스턴스가 아니라 타입의 value 자체를 지칭한다고 하여 혼란이 왔습니다. 

예제 코드를 보니 타입 내부에서 쓰이는 self와 타입 외부에서 쓰이는 self가 다른 개념임을 깨달았습니다.

- 타입 내부에서 사용시

```swift
class Superclass {
    func f() -> Self { return self }
}

let x = Superclass()
print(type(of: x.f()))
// Prints "Superclass"
```

SuperClass 라는 클래스 내부에서 쓰인 Self는 자기 자신의 타입을 의미

self는 자기 자신의 인스턴스를 의미

- 타입 외부에서 사용시 → 메타 타입

메타타입이란 `타입.self`를 붙여서 타입 이름을 value 로 접근할 수 있게 하는 것 

```swift
class SomeBaseClass {
    class func printClassName() {
        
    }
}

class AnotherSubClass: SomeBaseClass {
    let string: String
    required init(string: String) {
        self.string = string
    }
    override class func printClassName() {
        print("AnotherSubClass")
    }
}

let metatype: AnotherSubClass.Type = AnotherSubClass.self
let anotherInstance = metatype.init(string: "some string")

print(metatype) //AnotherSubClass
print(anotherInstance) //__lldb_expr_34.AnotherSubClass

print(type(of: AnotherSubClass(string: "some string")) //AnotherSubClass
```

타입 외부에서 사용시 자신의 인스턴스를 의미하는 것이 아니라 자신의 타입 그자체를 의미한다는 사실을 알게 되었습니다. 

type(of: 인스턴스)를 사용해도 AnotherSubClass.self 와 동일하게 자신의 타입 그자체 == 메타타입을 의미합니다.


## Encoding and Decoding

언어 자체에서 인코딩이나 디코딩하는 방법을 알려주고있음 → 스위프트 문서 (큰 범위)

코딩키를 이용하는 방법 선택가능 - 여기서 제공하는 기능중 하나가 프로퍼티와 키값이 다를때 이렇게 쓸수있다는 방식

JSON같은 데이터가 왔을때 모델로 만들어주는걸 디코딩이라고 하는데 

Foundatuon 의 jsondecoder 를 통해 할거야! 

디코더를 JSONDecoder 를 사용했을떄

key에 대한 전략을 세우는게 *JSONDecoder* 의 keyDecodingStrategy (좀 세부 범위)

모델링이란?
 : 화면, 도메인 , 요구사항이 들어왔을때 그것에 맞게/유리하게 객체를 만드는 것 
-> 제이슨 파일을 보고도 어떻게 나타낼지도 연습이 될수있음
