![](https://velog.velcdn.com/images/dev_jane/post/37301875-9899-4aff-b8ab-d6a1b9a4779a/image.png)

CGRect가 뭐지? 

View를 그릴때마다 항상 나오는 개념이라서 어렴풋이는 알고 있었는데 
이번 기회에 확실히 정리해보려 한다

# CGPoint, CGSize, CGRect

![](https://velog.velcdn.com/images/dev_jane/post/dcb3f6cb-f52d-4b26-aa01-f4674316e680/image.png)


View를 그릴 때는 시작 위치와 크기가 필요하다.
- CGPoint는 시작 위치인 **x, y 좌표**를 가지고
- CGSize는 시작 위치에서 얼만큼 갈 것인지에 대한 정보인 **width, height**를 가진다
- 그리고 CGRect는 CGPoint와 CGSize를 사용하여 정의될 수 있다.

![](https://velog.velcdn.com/images/dev_jane/post/3cd293e2-37e9-4820-83ba-e8dbfa178856/image.png)

예제를 보자ㅏ
CGPoint와 CGSize로 정의한
CGRect를 이용하여 View를 그릴 수 있다.

```swift
let point = CGPoint(x: 1, y: 1)
let size = CGSize(width: 2, height: 2)

let rect = CGRect(x: 1, y: 1, width: 2, height: 2)
let rect2 = CGRect(origin: point, size: size)
```

![](https://velog.velcdn.com/images/dev_jane/post/046bfeb5-2d56-48d3-b2a7-e942e2e202f8/image.png)


많이들 들어봤을 frame과 bounds의 타입이 바로,, CGRect이다.
