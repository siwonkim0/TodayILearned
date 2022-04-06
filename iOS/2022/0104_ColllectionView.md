# Collection View

테이블뷰와 다른 점 3가지 

1. **셀의 기본 스타일**이 없음 

애초부터 처음에 셀 모양이나 셀 크기를 지정해서 사용해야한다

테이블뷰보다는 해줄 일이 살짝 더 많다. 

1. indexPath: section 이란 표현은 같은데.. 테이블뷰는 row 대신 **item**을 사용

“이 section의 몇번째 item” 이렇게 말함 

1. 레이아웃 직접지정가능 

# DataSource : UICollectionViewDataSource

**UICollectionViewDiffableDataSource 클래스**

- `UICollectionViewDataSource` 프로토콜을 채택하고있음 (얘 채택하는 커스텀 데이터소스를 만들기도가능)
- 데이터와 셀을 관리하는 클래스
- 프로토콜 채택한 곳에서 이 두 메서드 필수로 구현해야함

```swift
func collectionView(_ collectionView: UICollectionView, 
numberOfItemsInSection section: Int) -> Int

func collectionView(_ collectionView: UICollectionView, 
      cellForItemAt indexPath: IndexPath) -> UICollectionViewCell
```

![스크린샷 2022-01-02 오후 11.31.30.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/baacf129-3b7c-4dd9-bad3-2622e3f69559/스크린샷_2022-01-02_오후_11.31.30.png)

**1.2.** **Diffable data source 를 collection view 와 연결하고, cell provider로 셀 configure하기**

: 이니셜라이저를 통해 연결하는데, 파라미터로 collectionView와 cellprovider을 넘겨준다.

- collection view : data source 와 연결하고싶은 컬렉션뷰
- cell provider: cell을 configure하고 UI에 어케 보여줄지 결정하는 클로저 전달

```swift
dataSource = UICollectionViewDiffableDataSource<Int, UUID>(collectionView: collectionView) {
    (collectionView: UICollectionView, indexPath: IndexPath, itemIdentifier: UUID) -> UICollectionViewCell? in
    // Configure and return cell.
}
```

**3.4.** **data 의 현재 상태를 만들고 UI에 보여주기**

**NSDiffableDataSourceSnapshot**

뷰가 디스플레이하는 데이터의 초기 상태를 세팅하거나, 데이터의 변화를 반영하는 데에 사용한다. 

```swift
// Create a snapshot.
var snapshot = NSDiffableDataSourceSnapshot<Int, UUID>()        

// Populate the snapshot.
snapshot.appendSections([0])
snapshot.appendItems([UUID(), UUID(), UUID()])

// Apply the snapshot.
dataSource.apply(snapshot, animatingDifferences: true)
```

# Layout: **UICollectionViewLayout**

이거 직접 구현하기 쉽지 않으니 튜토리얼을 먼저 보고 문서와서 확인하는 방식으로 공부하기 (부스트코스 수강하기)

+.아카이브 문서 읽어보기 

**추상** 클래스임 : 얘를 직접 구현하지 않고 상속받아서 사용한다.

cells, supplementary views, decoration views 등의 자리를 결정하는 객체이다.

You create a custom subclass of `UICollectionViewController` for each collection view that you want to manage. When you initialize the controller, using the `init(collectionViewLayout:)` method, you specify the layout the collection view should have.

컬렉션뷰컨 만들면 꼭 레이아웃 이니셜라이즈해야함

컬렉션뷰는 리스트 형태로 주르르 표시하는게 아니라 얘는 크고 얘는 작고.. 다양하게 구성할 수 있어서 

레이아웃에는 세가지요소가 있음 

- **cells**
    - 레이아웃으로 위치를 잡는 메인 요소들이다.
    - 하나의 셀은 하나의 데이터를 표시한다
    - 셀을 인터렉티브하게 만들기도 가능 : 선택, 드래그, 재배치 등
    - 하나의 그룹에 모든 셀들을 다 넣을수도 있고, 다수의 섹션으로 나눌수도 있음
    - layout 객체에 의해 사용과 위치가 결정된다

- **supplementary views**
    - header 나 footer view 처럼 데이터를 보여주지만 사용자가 선택은 불가한 곳
    - optional 이다.
    - layout 객체에 의해 사용과 위치가 결정된다

- **decoration views**
    - 배지 등 시각적 장식이다. (뒷 배경도)
    - supplementary view 의 다른 타입임
    - 컬렉션뷰의 데이터와 연결되어있지 않음
    - layout 객체에 의해 사용과 위치가 결정된다

컬렉션뷰는 layout object에게 레이아웃 정보를 달라고 자주 요청한다

화면에 나타나는 모든 셀과 뷰는 layout object의 정보로 인해 위치가 결정됨 

![스크린샷 2022-01-02 오후 11.58.43.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4692d6b1-2d0d-42ef-b95f-0d8530638d33/스크린샷_2022-01-02_오후_11.58.43.png)

이 메서드들 오버라이드 해야함 

아이템이 매번 삭제되거나 추가되면 컬렉션뷰가 layout object한테 업데이트 요청을 하고 추가적인 layout pass 가 일어난다.

스크린에 보여지는 애들만 레이아웃 적용되게 (?) 하는듯

---

### UITableViewController vs UITableView 선택기준?

화면에 테이블뷰만 있는 상황이라면 유용하게 사용가능 

근데 화면에 다른 뷰 요소들이 있을때는 그냥 뷰컨사용

근데.. 커스터마이징이 많이 제한이됨 (확장가능성 떨어진다)

기획자들이 나한테 와서 무슨얘기를 할줄몰라서

야곰의 경우에는 그냥 뷰컨에 테이블뷰 따로 구현해서 사용한다고함 

### CollectionView vs TableView

컬렉션뷰로 다 하면되는데 굳이 테이블뷰를 사용할 필요가 있는가? 

기획자의 의도에 따라 다르게 고를 수 있음 

그리드 모양으로 갈수도있어요 ~ → 컬렉션뷰의 list

절대로 list형태를 벗어나지 않아요 → 테이블뷰 

configuration만 꽂아주기때문에 서로 바꾸는게 어렵지는 않다.. 

configuration 공유해서 사용함 컬렉션, 리스트 둘이 

드래그 앤 드롭

- 이 테이블뷰에서 다른 테이블뷰로 보낼때임

이 앱에서 저 앱으로 보낼때 (아이패드의 스플릿뷰,, ) 

→ 하나의 앱에서 두 화면을 쓸때 (서로 다른 씬으로 보낼때.. ) 

- 하나의 테이블 , 컬렉션 뷰 안에서 아이템이나 셀을 옮기는 것 은 셀의 이동이지

드래그 앤 드롭이 아님
