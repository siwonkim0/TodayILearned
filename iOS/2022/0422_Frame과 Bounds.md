# Frame, Bounds

frame은 view 자신의 superview의 좌표계 안에서 자신의 위치를 결정한다.

![](https://velog.velcdn.com/images/dev_jane/post/c9d0bfe5-fa09-4d2f-b17b-bfaa5b9aa6e0/image.png)

아래 discussion에 보니 view의 사이즈와 위치를 세팅할 때 이 frame을 사용하라고 한다.
frame을 세팅하면 view의 center 프로퍼티랑 bounds 가 자동으로 바뀐다.

![](https://velog.velcdn.com/images/dev_jane/post/1649308a-8e42-44ac-86d8-a93c4e128b25/image.png)

그럼 bounds는 뭐지?
슈퍼뷰하고 상관없이 자신만의 좌표시스템을 가진다고 한다

![](https://velog.velcdn.com/images/dev_jane/post/ccc39b77-5044-45a8-9840-cb9633f876ea/image.png)


UIView 문서를 보니 뷰를 생성할때는 슈퍼뷰가 될 뷰에 상대적으로 사이즈와 위치를 설정해야한다고 한다.
그래서 frame을 사용하는구나 ! 

![](https://velog.velcdn.com/images/dev_jane/post/c2290aa1-5894-4071-bcae-7803372d338d/image.png)

## frame과 bounds를 공부할 때 헷갈리는 상황 두가지가 있다

1. 뷰의 bounds를 바꿨을 때 왜 뷰가 아니라 서브뷰가 반대방향으로 이동하지?
    - frame을 바꿨을 땐 보통의 상식 대로 뷰가 이동함
    - bounds를 바꿨을 땐 서브뷰가 이동하는 것처럼 보이지만, 서브뷰가 직접 이동하는 것은 아니다.
        - 사실은 뷰가 이동을 해서 그 안에 들어간 서브뷰가 반대로 이동**하는 것처럼 보이**는 것임
2. 뷰를 회전시켰을때 왜 뷰의 frame이 변하지?
    - bounds는 그대로임
    - frame만 변하는데 그 이유는 frame의 정의가 뷰를 감싸고 있는 사각형이어서이다. 뷰를 회전시키면 비스듬히 되는데 그걸 감싸는 사각형을 그리면 보통 원래 뷰보다 너비와 높이가 길어짐

이 두개의 질문을 해결하기 위해서

view를 이동하고 회전하는 상황에서 frame과 bounds가 가진 두가지 속성, origin과 size가 어떻게 변하는지 비교해볼 것이다.

## 1. origin 이동

예제 그림으로 살펴보자

아래 그림에서 mainView의 frame의 origin은 superView의 origin로부터 50, 50 만큼 떨어져있음

![](https://velog.velcdn.com/images/dev_jane/post/c91be190-d59c-42bc-ab2f-c1f7fd271815/image.png)


```swift
//superView - Cyan
let superViewRect = CGRect(origin: CGPoint(x: 50, y: 100),
                   size: CGSize(width: 300, height: 300))
let superView = UIView(frame: superViewRect)
superView.backgroundColor = .cyan
self.view.addSubview(superView)

//mainView - Yellow
let mainViewRect = CGRect(origin: CGPoint(x: 50, y: 50),
                           size: CGSize(width: 200, height: 200))
let mainView = UIView(frame: mainViewRect)
mainView.backgroundColor = .yellow
superView.addSubview(mainView)
print("mainView frame:", mainView.frame) //(50.0, 50.0, 200.0, 200.0)
print("mainView bounds:", mainView.bounds) //(0.0, 0.0, 200.0, 200.0)
```

frame과 bounds를 찍어보니 이렇게 나왔음

![](https://velog.velcdn.com/images/dev_jane/post/e5fd9b9c-ff6e-4e47-86cd-0b467fd32c36/image.png)

frame은 슈퍼뷰에 비해 50, 50만큼 자신이 떨어져있다는 사실을 알고있고
bounds는 슈퍼뷰 상관안하고 그냥 자신이 기준이라 0,0임

## 1-1. frame의 origin을 이동했을 때

그렇다면 이제 frame을 움직여보면 어떤 일이 일어날까?
궁금해서 세개의 뷰가 있는 예제를 만들어 직접 간단하게 찍어보았다.
대충 만든거라 못생겼어도 양해를 부탁합니다,,,

![](https://velog.velcdn.com/images/dev_jane/post/349f4f2a-58b8-450a-a839-eac28c78e29a/image.png)


blueView, yellowView, greenView 세가지의 뷰들이 있는데
각각의 frame의 x좌표를 10만큼 움직이면 어떻게 될까?
![](https://velog.velcdn.com/images/dev_jane/post/b022d47c-9091-48b5-a6ad-278bc41f6da4/image.gif)


예상했던대로 frame은 자신의 슈퍼뷰와의 관계속에서 결정되니까
슈퍼뷰를 기준으로 10씩 오른쪽으로 움직이는군

## 1-2. bounds의 origin을 이동했을 때

그렇다면 bounds는?
![](https://velog.velcdn.com/images/dev_jane/post/003bc0cc-b03d-4d34-bb6f-fd4ccb6a6fd2/image.gif)

아니뭐야 blueView의 bounds의 x좌표를 10만큼 움직였더니 
blueView가 아니라 서브뷰인 YellowView의 frame이 움직이는 것 같이 보인다.
근데 yellowView의 frame은 변하지 않는 것 보이져?
blueView의 bounds만이 잘 바뀌고 있음 

yellowView의 bounds의 x좌표를 10만큼 움직였더니 
마찬가지로 또 yellowView가 아니라 서브뷰인 greenView가 움직이는 것 같이 보이네

마지막으로 greenView의 bounds의 x좌표를 10만큼 움직였는데 
이런,, 아무도 움직이지 않는다
greenView는 서브뷰가 없어서 greenView의 bounds가 변해도 티가 안나는 것이다
왜냐 저기 프린트되는 것을 보면 greenView의 bounds가 변하긴 하거든요

이 개념이 처음에 잘 이해가 안갔는데 아래 예시를 보고 바로 이해가 갔다.

### bounds의 개념은 스크롤뷰의 작동원리와도 관련이 있다.

우선 아이폰으로 풍경사진을 찍는다고 가정해보자.

카메라를 켜면 넓은 실제 풍경을 다 담을 수 없고 어느 한 부분만 아이폰 화면에 보이게 된다. 
그리고 우리가 아이폰을 이리저리 움직이면 보여지는 풍경이 달라진다. 

스크롤뷰도 이와 마찬가지인데, 화면보다 큰 내용을 가진 전체 페이지에서 사용자가 스크롤한 부분만 현재 보여주는 방식으로 작동을 한다. 
아이폰을 아래로 스크롤하는 상황에서, 
우리는 화면의 내용을 위로 올리고 있다고 생각하지만
사실은 화면이 내려가면서 페이지의 아래 부분을 보여주는 것이다. 

내가 아이폰에서 스크롤 캡쳐를 한건데 이해가 쉽도록 나와있어서 가져와보았다.
![](https://velog.velcdn.com/images/dev_jane/post/0b0ad59f-5b26-4879-84cd-4a4c67b45b9d/image.gif)

그렇다면 다시 내가 만든 예제로 돌아와서,
![](https://velog.velcdn.com/images/dev_jane/post/b6b68aae-075f-4166-b47e-c3bb0043391e/image.png)

저 blueView의 bounds의 x좌표를 10만큼 움직이면
blueView가 오른쪽으로 이동하면서, 마치blueView의 서브뷰인 yellowView가 왼쪽으로 이동하는 것처럼 보이는 것이다. 

이건 스크롤을 아래로 내릴때 실제로는 스마트폰 화면이 아래로 이동했는데 내용이 위로 올라가는 것처럼 보이는것과 같은 원리다. 

## 2. 회전

그렇다면 이제 뷰를 회전시켰을 때의 상황에 대해 알아보자.
검색하다가 발견한 좋은 예제이다.
![](https://velog.velcdn.com/images/dev_jane/post/c5d8bd25-c847-4c6d-88c9-2cbbaed07c73/image.gif)


뷰를 회전시키자 bounds는 그대로고 frame만 바뀌는 상황을 볼 수 있다.
bounds는 슈퍼뷰하고 상관없이 자신만의 좌표시스템을 가지기 때문에 회전해도 아무런 변화가 없다.

그렇담 frame은? 저 view를 감싸는 사각형이 frame인데
view가 회전되면 사각형의 크기가 달라진다.
따라서 변하는거


[GIF 출처](https://serialcoder.dev/text-tutorials/ios-tutorials/frame-vs-bounds-in-ios-implementing-a-visual-demonstration/)
