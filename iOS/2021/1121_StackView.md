학습내용
- Step 3 스택뷰 코드로 구현
- 스택뷰에 있는 모든 서브뷰 제거

1. 스택뷰
스택뷰에 스택이 몇개 들어올지 처음에는 모르는 상황에서, UI에 스택의 개수를 지정해줄 수 없어서 코드를 이용하여 스택이 생성될때마다 mainStackView에 서브뷰로 추가해주는 방식 사용 

처음에는 첫번째 stack view를 통째로 hidden해서 아예 안보였음 

해결 ) 두번째, 세번째 stack view 를 hidden해야함

2. 스택뷰의 서브뷰 제거(AC버튼)

[[iOS] Stack View에 대해서 3](https://bannavi.tistory.com/261?category=878332)

stackView는 두개의 배열을 가지고 있는데, 

subviews

arrangedSubviews

이 두가지다.

arrangedSubviews에 view추가시 subview에도 추가되는데,

삭제하려고 removearrangedsubview해도 arrangedSubviews에만 삭제되고 subview에선 삭제불가능 

해결 방법.
removeArrangedSubview(_:) 메서드를 호출한 후 뷰의 removeFromSuperview() 메서드를 호출하여 뷰를 명시적으로 제거하기.
