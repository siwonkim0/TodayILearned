### 학습 내용
1. dismiss(animated:completion:) : 모달 방식으로 present된 뷰컨트롤러를 dismiss하는 메서드

뷰의 네비게이션바에 바버튼아이템과 연결하여 닫기버튼을 만들어주었다.

```swift
@IBAction func dismissButton(_ sender: UIBarButtonItem) {
        self.dismiss(animated: true, completion: nil)
    }
```

2. UIAlertAction handler
재고가 부족할 때 뜨는 얼럿창에 재고 수정하기 버튼을 클릭시, 재고수정화면으로 가는 기능 추가.
UIAlertAction에 후행 클로져로 presentStockModifyView() 를 가져와서 재고수정화면을 모달창으로 띄워 이동했다. 
```swift
let ok = UIAlertAction(title: "재고 수정하기", style: .default) { _ in self.presentStockModifyView() }
```


### 고민한 부분
- 얼럿 버튼 선택시 하고싶은 일 구현?
-> UIAlertAction handler 로 클로져를 넘겨줌
- 스태퍼 구현시 재고값을 어디서 가져오면 거기에 더하는 방법이 없어서 고민했음 
-> 디폴트값을 아예 10으로 설정하기?
- 과일 라벨을 IBOutlet로 하나하나 연결해서 프로퍼티를 만들었는데, 하나의 프로퍼티로 다 관리하는 방법은 없을까?
* 뷰 간에 데이터 전달 방법을 좀 더 고민해봐야겠다.
-> 현재 사용한 방법은 재고를 싱글턴으로 하여 재고가 변할때마다 노티를 뷰컨 두개에 동시에 보내서 재고 업데이트하게함 
