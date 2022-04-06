[The Basics - The Swift Programming Language (Swift 5.5)](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html)

# 학습내용

오늘은
Swift Language Guide 를 읽어보았다. 
디스코드에서 남는 시간에 공식 문서 읽는 연습을 하면 좋다는 답변을 보고 영어로 되어있어서 읽는 것을 미뤄왔던 공식 문서를 읽기 시작하기로 했다. 😅
한국말로 읽어도 생소한 내용이라 영어로 읽으면 이해하는데 훨씬 오래걸려서 귀찮지만... 열심히 해봐야겠다 !

## The Basics

### String interpolation : \()

```swift
var hello = "Bonjour"
print("Hello is \(hello) in French")
```

문자열 안에 변수나 상수의 값을 나타낼때 쓰이는 \() 이것을 string interpolation이라고 한다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67e12f7e-dda4-4c73-b22f-44924aabd676/스크린샷_2021-08-08_오전_2.34.22.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67e12f7e-dda4-4c73-b22f-44924aabd676/스크린샷_2021-08-08_오전_2.34.22.png)

### Semicolons ;

다른 언어들처럼 코드 뒤에 ;를 붙이라고 하지는 않지만(하고싶으면 할 수는 있음), 여러 개를 한줄에 쓰고 싶으면 ; 뒤에 쓴다.

```swift
let cat = "🐱"; print(cat)
// Prints "🐱"
```

### Integer Bounds

min, max 프로퍼티를 통해 Integer 형의 최소, 최대값에 접근할 수 있다

```swift
let minValue = UInt8.min  // minValue is equal to 0
let maxValue = UInt8.max  // maxValue is equal to 255
```

### Type Safety and Type Inference

- **Type Safety**

스위프트는 type-safe 언어라서 변수 선언시 정확하게 타입을 밝혀줘야하고, 서로 다른 타입을 할당할 수 없다. 

컴파일시 type check를 통해 error를 최대한 빨리 잡아낼 수 있도록 도와준다.

- **Type Inference**

그런데 모든 상수나 변수를 선언할 때 꼭 타입을 밝혀줘야하는 것은 아니다.

타입을 안밝혔을 때, 스위프트가 자동으로 타입을 예상한다.

```swift
let meaningOfLife = 42
// meaningOfLife is inferred to be of type Int
```

## Numeric Type Conversion

Int 타입을 보통 써라, 그 값이 0이나 양수일지라도

다른 integer 타입은 특별히 필요할 때만 써라

## Integer Conversion

다른 타입끼리는 더할 수 없으니 타입을 바꿔줘야 한다.

```swift
let twoThousand: UInt16 = 2_000
let one: UInt8 = 1
let twoThousandAndOne = twoThousand + UInt16(one)
```

## Type Aliases

기존 타입의 새로운 이름을 정의할때 typealias 를 사용한다.

왜 새로운 이름을 정의하나? : 문맥적으로 더 적절할 때

```swift
typealias AudioSample = UInt16
var maxAmplitudeFound = AudioSample.min
// maxAmplitudeFound is now 0
```

AudioSample.min 은 사실 UInt16.min 이다

## Booleans

불리언은 if 같은 조건문에서 유용하다.

```swift
let orangesAreOrange = true
let turnipsAreDelicious = false

if turnipsAreDelicious {
    print("Mmm, tasty turnips!")
} else {
    print("Eww, turnips are horrible.")
}
// Prints "Eww, turnips are horrible."
```

- Type safety

```swift
let i = 1
if i {
    // this example will not compile, and will report an error
}
```

```swift
let i = 1
if i == 1 {
    // this example will compile successfully
}
```

## Tuples

다수의 값들을 하나의 복합적인 값(compound value)으로 그룹짓는다
값들은 서로 다른 타입을 가져도 된다

```swift
let http404Error = (404, "Not Found")
// http404Error is of type (Int, String), and equals (404, "Not Found")
```

튜플 안 값을 활용하기 위해 튜플을 분해할수도 있다.

- 방법1

```swift
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
// Prints "The status code is 404"
print("The status message is \(statusMessage)")
// Prints "The status message is Not Found"
```

- 방법2

```swift
let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")
// Prints "The status code is 404"
```

- 방법3

```swift
print("The status code is \(http404Error.0)")
// Prints "The status code is 404"
print("The status message is \(http404Error.1)")
// Prints "The status message is Not Found"
```

- 방법4

```swift
let http200Status = (statusCode: 200, description: "OK")

print("The status code is \(http200Status.statusCode)")
// Prints "The status code is 200"
print("The status message is \(http200Status.description)")
// Prints "The status message is OK"
```

# 학습내용

어제에 이어 Swift Language Guide: The Basics 를 살펴보겠다.

[The Basics - The Swift Programming Language (Swift 5.5)](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html)

## Optionals

값이 없을 수도 있는 경우에 사용한다.
옵셔널은 두가지 경우를 나타내는데, 
1) 언래핑해서 값에 접근할 수 있음 2) 값이 없음

언제 사용하는가
String 값을 Int로 바꿀 때 그 값이 Int일 가능성도 있고 아무 값이 없을 수도 있어서( Bool 이나 String 은 가질 수 없음) 

*An optional Int is written as Int?, not Int. The question mark indicates that the value it contains is optional, meaning that it might contain some Int value, or it might contain no value at all.*

*(It can’t contain anything else, such as a Bool value or a String value. It’s either an Int, or it’s nothing at all.)*

```swift
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)
// convertedNumber is inferred to be of type "Int?", or "optional Int"
```

## nil

옵셔널 변수에만 nil값을 할당할 수 있다.
옵셔널 변수에 값을 할당하지 않고 선언하면 자동으로 nil값이 할당된다.

```swift
var surveyAnswer: String?
// surveyAnswer is automatically set to nil
```
