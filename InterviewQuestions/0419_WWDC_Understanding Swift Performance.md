# Understanding Swift Performance

내가 작성하고 있는 코드의 성능을 향상시키려면 항상 다음과 같은 질문을 염두해 두어야 한다.

- 인스턴스가 스택에 저장될까 힙에 저장될까?
- 인스턴스를 전달시 RC 는 얼마나 증가될까?
- 인스턴스의 메서드는 static, dynamic 중에 어떻게 dispatch될까?

## 구조체의 메모리 저장 방식
![](https://velog.velcdn.com/images/dev_jane/post/40fcd729-5eb9-4af3-a433-75063c918c7b/image.png)

스택 영역에 값만 저장이 된다.
point1 변수를 point2 변수에 할당시 스택 영역에서 새로운 값이 생성된다.
두 변수가 가지고 있는 값은 다른 값이라서 point2의 프로퍼티를 변경해도 point1에는 아무런 영향을 미치지 않는다.

## 클래스의 메모리 저장 방식
![](https://velog.velcdn.com/images/dev_jane/post/dd5a5aa2-a3d6-4ff6-b6cb-88d086302e8c/image.png)  

스택 영역에는 힙 영역을 가리키는 주소값이 저장이 되고,
힙 영역에 실제 값(인스턴스)이 저장이 된다.
구조체와 다르게 클래스는 힙 영역에 두개의 메모리 공간이 더 생기는데, RC값을 저기다 저장한다.(+ 다형성을 위한 type도 저장)

point1 변수를 point2 변수에 할당시 레퍼런스만 전달이 되어서 
두 변수 모두 힙 영역에 있는 같은 값을 가리키게 된다.
같은 값을 가리키고 있기 때문에 point2의 프로퍼티를 변경시 point1도 변하게 된다. 
→ 의도치 않은 값 변경이 생길 수 있다.

클래스는 구조체보다 비용이 비싼데, 그 이유가 힙 영역을 사용하기 때문이다.

## 근데 만약 구조체가 클래스를 가지고 있다면

그냥 클래스를 사용하는 것보다 비용이 두배가 들어간다고 한다.  

![](https://velog.velcdn.com/images/dev_jane/post/ce50dd65-9660-4ae4-bc54-a4d1b6dee543/image.png)

![](https://velog.velcdn.com/images/dev_jane/post/e6430269-4cd7-453c-b5d3-6574de5b56d2/image.png)

# static dispatch vs dynamic dispatch

static dispatch보다 dynamic dispatch가 비싸다
struct의 경우 static dispatch, class의 경우 dynamic dispatch사용 

## Class 상속시 발생하는 다형성을 위해 dynamic dispatch 사용

![](https://velog.velcdn.com/images/dev_jane/post/03ef1594-6d0c-4d2d-ab50-eb2633beb309/image.png)


Drawable 클래스를 상속하는 Point와 Line 클래스를 동시에 담고있는 배열을 만들 수 있다.
배열에는 reference만 담기기 때문에 다 같은 크기라서 괜찮음 
컴파일 타임에는 저 draw() 메서드가 Point 것인지 Line 것인지 알지 못하고,, 
draw() 메서드를 실행시 virtual table에서 어떤 draw()인지 찾은 후에 전달함
virtual table은 아래 그림처럼 어떤 draw()인지 알고있는 포인터를 가지고 있음 

![](https://velog.velcdn.com/images/dev_jane/post/d041cedf-b28b-4d25-923c-d1a8fef636ce/image.png)


## dynamic polymorphism을 Protocol + Struct로도 구현가능

Drawable 프로토콜을 채택하는 Point와 Line 구조체를 동시에 담고있는 배열을 만들 수 있다.
그럼 이때는,, 어떤 struct의 draw가 불려야될지 어떻게 아는걸까??
결론적으로는 Protocol Witness Table에서 찾는데,, 이게 좀 복잡함

![](https://velog.velcdn.com/images/dev_jane/post/bc422427-5430-4769-9efc-e17122a0018f/image.png)

### 일단 각 인스턴스마다 Existential Container 가 생김

생기는 이유는, Drawable 타입을 채택하는 struct들이 다 같은 크기가 아닌데 배열에 저장하려면 같은 크기여야 해서 Existential Container 라는 저장을 위한 레이아웃을 사용하는 것이다.
![](https://velog.velcdn.com/images/dev_jane/post/a1c0ad3c-6f05-4b2e-a331-54a9456fce01/image.png)

### 그럼 이제 Existential Container의 구성을 자세히 보자

세가지 부분으로 이루어져있음

- valueBuffer
- Value Witness Table (의 reference)
- Protocol Witness Table (의 reference)
![](https://velog.velcdn.com/images/dev_jane/post/15ef6925-31c8-4106-bacb-5f796ca47b7e/image.png)

1. **valueBuffer**

여기에는 처음 세개가 value buffer 자리임

- Point처럼 작은 값의 경우 value buffer에 바로 저장하고,
- Line처럼 큰 값의 경우 Heap에 메모리를 할당하고 힙에 대한 포인터를 저장함

![](https://velog.velcdn.com/images/dev_jane/post/a6e3bb17-1f3c-460b-893e-87b6991989e4/image.png)


1. **Value Witness Table**

그럼 이런 작은 값과 큰 값이 저장되는 방식의 차이를 어케 관리하지?
Value Witness Table로 관리함
이건 각 타입마다 하나씩 가지고 있는데.. (Point와 Line 각각 Value Witness Table 가지고 있음)
Value가 메모리에 할당되고 해제되는 lifetime을 관리하는 역할을 한다고 보면 됨

예로 큰 값의 경우인 Line 타입의 Value Witness Table을 봐보자 
Line의 경우 아까 value buffer 설명에서
Existential Container의 value buffer자리에 바로 들어갈 수 없어서 
힙에 메모리를 할당하고 힙에 대한 포인터만 Existential Container에 저장했었다.

![](https://velog.velcdn.com/images/dev_jane/post/0777b298-4568-42f4-811e-3d761becb882/image.png)

근데 이 저장하고 메모리에서 해제하기까지 과정이 4단계임 

- allocate : buffer에 메모리 할당할건지, 힙에 저장하고 레퍼런스만 저장할건지 결정
- copy: 값이 할당됨
- destruct: draw() 메서드 실행완료후 RC 감소시킴
- deallocate : 힙 할당해제, 스택의 buffer의 레퍼런스도 해제

1. **Protocol Witness Table**

아니 그러면 나머지 하나인 Protocol Witness Table은 언제 사용하는거냐?
그건 바로 copy와 destruct 사이에 draw() 메서드가 실행될 때 
Point 타입의 draw() 메서드인지 Line 타입의 draw() 메서드인지 확인하기 위해서 사용하는거다

## struct 안에 struct 타입 복사시 expensive

struct 안에 struct 타입의 프로퍼티를 가진 인스턴스를 복사하는 경우,
저 힙에 있는 메모리가 그대로 복사되어서 메모리를 엄청나게 씀 

![](https://velog.velcdn.com/images/dev_jane/post/716fdcb8-5610-4389-a17a-85b79739d074/image.png)

하 그럼 이걸 또 어케 해결하냐?
저 struct 타입 안의 struct타입 대신 class 타입을 넣는거임
이러케 ㅋ
valueBuffer에 어짜피 reference 들어갈 수 있으니 클래스를 사용하는거지
그럼 복사시 저 힙의 클래스는 복사 안되고 reference만 전달이 되어서 메모리를 아낄 수 있음

![](https://velog.velcdn.com/images/dev_jane/post/46179e1d-f02c-41d3-b391-f99edf57d0bc/image.png)

근데 또 여기서 문제 ㅎ
클래스 사용시 의도치 않은 값 변경이 일어날 수 있다는 점..
그래서 Copy On Write를 도입하면 된다.

## struct 안에 class 타입 + Copy On Write으로 해결~!

Copy On Write가 무엇이냐면, 

클래스를 수정해야 할 때 그 클래스의 RC를 확인해서 만약 RC=1이면 그대로 수정하고,

RC가 1보다 크면 클래스 인스턴스를 새로 생성해서 그걸 수정하는 방식으로 사용하는 것이다. 

이렇게 되면 애초에 힙 영역에 Line 인스턴스는 하나만 생성이 되고 

Pair 타입 복사시 레퍼런스만 네개되고 다 같은 인스턴스를 참조하고 있어서 훨씬 cheaper 하다 ㅎ 

![](https://velog.velcdn.com/images/dev_jane/post/d34d1841-8733-4523-8363-a1995f441990/image.png)

# 결론

- 구조체를 사용할 수 있는 곳에서는 구조체를 사용하자
    - 값들을 String에 모아놓지 말고 구조체에 모아놓자 (String은 내부적으로 Heap 사용)
- static dispatch보다 dynamic dispatch가 비싸기 때문에 될수있으면 static 사용하자
    - class의 경우 final키워드 붙인다던지
- 다형성을 위해 dynamic dispatch를 사용해야 하는 경우 클래스보다는 구조체 + 프로토콜 조합으로 사용
    - 근데 구조체가 가진 데이터가 크다면 내부적으로 Heap에 저장되기 때문에 상황에 따라서 클래스 + Copy On Write 를 사용해서 참조만 전달하는 경우가 더 나을 수도 있다

# Reference

[https://developer.apple.com/videos/play/wwdc2016/416/](https://developer.apple.com/videos/play/wwdc2016/416/)

[https://zeddios.tistory.com/597?category=685736](https://zeddios.tistory.com/597?category=685736)
