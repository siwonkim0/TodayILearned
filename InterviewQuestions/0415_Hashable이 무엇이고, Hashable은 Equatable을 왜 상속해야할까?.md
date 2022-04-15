# Hashable/Equatable

Hashable이 무엇이고,
Hashable은 Equatable을 왜 상속해야할까?

## Hashable이 무엇일까

Hash가 될 수 있다 == `HashTable`에서 찾을 수 있다 (자신의 해시값을 찾을 수 있어야 한다)

해시테이블에서 해시값(**Hash Value**)을 찾으려면 key로 찾아야 하는데 그 key는 unique해야한다

```swift
protocol Hashable: Equatable {
  var **hashValue**: Int { get }
  func hash(into hasher: inout Hasher)
}
```

### Hash Value가 뭐지
 - 어떤 데이터가 와도 64비트짜리 정수로 치환하는거 (암호화)
 - 원본 데이터를 특정 규칙에 따라 처리하여 간단한 숫자로 만든 것
 - 2개의 데이터 비교시 데이터가 동일할 때 둘의 해시값은 같다.
   - 하지만 조금이라도 데이터가 다르면 둘의 해시값은 완전히 다르다.

## Hashable은 Equatable을 왜 상속해야할까


Hashable은 Equatable 프로토콜을 상속받고 있는데
그 이유는 hashValue는 고유값이어야 하기 때문에 고유값인지 식별해줄 수 있는 == 함수가 필요함

우리가 값을 비교할 때 `==`를 사용할 수 있었던 이유는
일반적으로 사용하는 Int, String, Double 등은 Hashable 타입을 상속받고 있어서 가능했던 것이다. 

![](https://velog.velcdn.com/images/dev_jane/post/6cd065f3-e026-4275-808c-8ced3f4347c3/image.png)


정말 그렇네

## 그럼 언제 커스텀 타입에 Hashable을 채택해야 하는걸까?

- dictionary의 key의 타입으로 사용할 때
- set의 타입으로 사용할 때

### 근데 왜 set이나 dictionary key가 되려면 Hashable이어야 하는걸까

set과 dictionary는 array와 다르게 순서가 없어서...
검색을 빨리하기 위해
그리고 중복을 허용하지 않는 이유도 hashable해서 인듯하다

## 자동으로 Hashable한 타입은 뭐가 있지

### 1. 스위프트 스탠다드 라이브러리 중 거의 다 이미 Hashable을 채택하고 있음
- String, Integer, Float, Bool, Double

![](https://velog.velcdn.com/images/dev_jane/post/8f819822-e271-4346-8b12-69dabedf7a0d/image.png)

### 2. 커스텀 타입 중에서는 
- 연관값이 없는 enum

## 커스텀 타입을 만약 Hashable을 채택하도록 하려면

이렇게 공식 문서에도 나와있듯이
![](https://velog.velcdn.com/images/dev_jane/post/1965f970-ec5c-45e2-b8e3-8951b9b1810b/image.png)


두가지 경우가 있음

### 1. 그냥 Hashable 프로토콜만 채택해주면 알아서 hash(into:) 구현해주는 케이스
 - 모든 저장 프로퍼티가 Hashable한 struct
 - 모든 연관값이 Hashable한 enum 인 경우는
        
![](https://velog.velcdn.com/images/dev_jane/post/56d6d9bf-c57b-4f9a-8e28-777bf6050a7d/image.png)
        

이렇게 모든 저장 프로퍼티가 Hashable한 Int타입인 경우 Hashable만 채택하면 
hash(into:) 구현하지 않아도 자동으로 생김

### 2. 그 외의 경우에는 두가지를 구현해줘야 하는데
1. `== 함수` (Equatable 프로토콜을 따라야해서)
2. `hash(into:) 함수` (Hashable 프로토콜을 따라야해서)

예를 들어 위의 struct를 class로 바꿨더니 그냥 Hashable만 채택해서는 안됐다
-> == 랑 hash(into:)를 구현해줘야 했음 

![](https://velog.velcdn.com/images/dev_jane/post/881592ad-bea6-4817-a02b-13b1faa5d292/image.png)
# Reference

[https://developer.apple.com/documentation/swift/hashable](https://developer.apple.com/documentation/swift/hashable)
[https://ko.wikipedia.org/wiki/해시_함수](https://ko.wikipedia.org/wiki/%ED%95%B4%EC%8B%9C_%ED%95%A8%EC%88%98)
[https://zeddios.tistory.com/498](https://zeddios.tistory.com/498)
