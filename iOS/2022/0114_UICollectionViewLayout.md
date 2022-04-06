# 학습 내용
- UICollectionViewLayout은 추상 클래스이고 레이아웃 객체를 만들때 이 클래스를 상속받아서 만든다.
레이아웃 설정 방법 세가지
1. UICollectionViewLayout을 직접 상속받아서 커스텀 레이아웃 클래스 사용가능 
2. UICollectionViewFlowLayout
CollectionView에서 레이아웃을 만드는 방법중에 하나인 UICollectionViewFlowLayout
UICollectionViewLayout의 서브클래스이고 Grid CollectionView를 만들기 쉽도록 애플에서 제공하는 클래스이다. 

3. UICollectionViewCompositionalLayout
좀 더 최근에 나온 이 방법또한 UICollectionViewLayout의 서브클래스이다. Section, Group, Item을 가진다.
보통 유동적인 배열을 가진 컬렉션뷰를 만들때 사용하는데 이번 프로젝트에서는 list 레이아웃을 가진 컬렉션뷰를 만드는데 사용했다. 
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61e57f38f680980601ab181c.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61e57f38f680980601ab181c.png)

2, 3번 방식으로 각각의 레이아웃을 만든 후에 UICollectionView 이니셜라이저를 통해 list 와 grid 형식의 레이아웃을 가진 컬렉션뷰를 초기화해주었다.

```Swift
struct OpenMarketViewLayout {
    static let list: UICollectionViewCompositionalLayout = {
        let configuration = UICollectionLayoutListConfiguration(appearance: .plain)
        let layout = UICollectionViewCompositionalLayout.list(using: configuration)
        return layout
    }()
    
    static let grid: UICollectionViewFlowLayout = {
        let layout = UICollectionViewFlowLayout()
        layout.sectionInset = UIEdgeInsets.init(top: 5, left: 10, bottom: 5, right: 10)
        layout.minimumInteritemSpacing = 0
        layout.minimumLineSpacing = 10
        let halfWidth: CGFloat = UIScreen.main.bounds.width / 2.0
        layout.itemSize = CGSize(width: halfWidth - 15, height: 260)
        return layout
    }()
}
```

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61e57d48f680980601ab1818.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61e57d48f680980601ab1818.png)
```Swift
class OpenMarketViewController: UIViewController {
    private let listCollectionView = UICollectionView(frame: .zero, collectionViewLayout: OpenMarketViewLayout.list)
    private let gridCollectionView = UICollectionView(frame: .zero, collectionViewLayout: OpenMarketViewLayout.grid)
}
```




## Reference
[UICollectionView Flow Layout을 사용해 셀 크기를 조절해보기](https://www.zehye.kr/ios/2020/03/12/11iOS_UICollectionview_flow_layout/)
