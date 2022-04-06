# 학습 내용
- URL로 이미지를 셀에 추가하는 방법 
- 리스트 컬렉션뷰
   - UICollectionLayoutListConfiguration
   - UIContentConfiguration
   - UICollectionViewDiffableDataSource
   - NSDiffableDataSourceSnapshot
- 셀 오토 레이아웃 programatically


## URL로 이미지를 셀에 추가하는 방법

URL → Data → UIImage 순서로 변환하기

```swift
var content = cell.defaultContentConfiguration()

content.text = productDetail.name

let imageURL = URL(string: "url주소")!
let imageData = try! Data(contentsOf: imageURL)
content.image = UIImage(data: imageData, scale: CGFloat(10))

cell.contentConfiguration = content
```

---



## 리스트 컬렉션뷰 만들기
UICollectionLayoutListConfiguration 으로 리스트 컬렉션뷰 만들고
UIContentConfiguration 으로 셀 생성하고
UICollectionViewDiffableDataSource로 셀에 데이터를 할당하고
NSDiffableDataSourceSnapshot으로 스냅샷을 찍어 dataSource 만듦

### Mainview 커스텀 뷰 생성
: 뷰컨에서의 view의 역할을 할 뷰, 컬렉션뷰를 가지고 있음

1. 기본 세팅 : 이니셜라이저

```swift
class MainView: UIView {
    init() {
        super.init(frame: .zero)
        setupSubviews()
        setup()
        
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

2. 컬렉션뷰의 레이아웃 세팅
    1. config
    2. layout
    3. collectionview를 layout으로 생성 

```swift
lazy var collectionView: UICollectionView = {
    //1.config
    var configuration = UICollectionLayoutListConfiguration(appearance: .plain)
    configuration.backgroundColor = .white
    //2. layout
    let layout = UICollectionViewCompositionalLayout.list(using: configuration )
    //3. collectionview
    let collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
    return collectionView
}()
```

3. 슈퍼뷰에 addsubview 하기 + 컬렉션뷰 오토레이아웃 세팅

```swift
func setupSubviews() {
    //3)컬렉션뷰를 상위뷰에 addSubview
    addSubview(collectionView)
    //2)컬렉션뷰 오토레이아웃
    collectionView.translatesAutoresizingMaskIntoConstraints = false
    NSLayoutConstraint.activate([
        collectionView.topAnchor.constraint(equalTo: topAnchor),
        collectionView.leadingAnchor.constraint(equalTo: leadingAnchor),
        collectionView.trailingAnchor.constraint(equalTo: trailingAnchor),
        collectionView.bottomAnchor.constraint(equalTo: bottomAnchor)
    ])
}
```

<mainview 전체 코드>

```swift
import Foundation
import UIKit

class MainView: UIView {
    lazy var collectionView: UICollectionView = {
        //1)컬렉션뷰의 레이아웃 세팅: layout list config
        //1.config
        var configuration = UICollectionLayoutListConfiguration(appearance: .plain)
        configuration.backgroundColor = .white
        //2. layout
        let layout = UICollectionViewCompositionalLayout.list(using: configuration )
        //3. collectionview
        let collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
        return collectionView
    }()
    
    lazy var button: UIButton = {
        let button = UIButton()
        button.setTitle("fffffr", for: .normal)
        return button
    }()
    
    init() {
        super.init(frame: .zero)
        setupSubviews()
        setup()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func setup() {
        backgroundColor = .systemCyan
    }
    
    func setupSubviews() {
        //3)컬렉션뷰를 상위뷰에 addSubview
        addSubview(collectionView)
        //2)컬렉션뷰 오토레이아웃
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: bottomAnchor)
        ])
    }
}
```

### 뷰컨트롤러, MyContentView 생성

1. 아까 만들어준 커스텀뷰 인스턴스 생성 후 view에 커스텀뷰 할당

```swift
class ViewController: UIViewController {
    let mainView = MainView()
    var dataSource: UICollectionViewDiffableDataSource<Section, ProductDetail>?
    
    override func loadView() {
        view = mainView
    }
}
```

2. 컬렉션뷰의 셀 registration : contentConfiguration으로 하기
    - ContentConfiguration 사용 방법 두가지
        - `.defaultContentConfiguration()`
        
        디폴트 UIListContentConfiguration에 데이터를 할당한 후 그걸 cell의 contentConfiguration에 할당하기
        
        - custom content configuration
            
            [The Developer's Guide to Cell Content Configuration in iOS 14 - BiTE Interactive](https://www.biteinteractive.com/cell-content-configuration-in-ios-14/)
            
        
        커스텀으로 만든 UIListContentConfiguration에서 데이터, 오토레이아웃을 설정한 후 그걸 cell의 contentConfiguration에 할당하기
        
        왜 이렇게 하냐?
        
        기존 방식은 셀을 디큐해서 그 셀의 속성에 직접 데이터를 할당해주는 것이었는데...
        
```swift
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: cellID, for: indexPath)
            let label = // ...
            label.text = "Hello, world"
            return cell
    }
```
        
사실 data source는 뷰를 직접 알면 이상함... 그냥 데이터를 셀한테주면 셀이 알아서 할당하는게 더 좋은 방법이다.
따라서 셀에 있는 content view 의 양식을 먼저 채워가지고 셀의 content view에 할당해주는 방식 !
        
**custom content configuration 만드는 순서**
1. UIView, UIContentView 프로토콜 채택한 MyContentView 생성 
    1. UIContentConfiguration 타입 프로퍼티로 만들기
    2. 셀에 원하는 구성요소들 프로퍼티로 만들어주기
    3. 이니셜라이저 파라미터로 UIContentConfiguration 받기
        1. b의 셀 구성요소들 addSubView하기
        2. b의 셀 구성요소들 오토레이아웃 잡기
        3. 받은 configuration으로 셀 구성요소들에 데이터 채우기
2. UIContentConsiguration 프로토콜 채택한 타입 생성
    1. label, secondaryLabel, imageView의 정보를 담을 text, secondaryText, image 프로퍼티 만들기
    2. 프로토콜의 필수구현 메서드 2개 구현하기
        - 1에서만든 UIContentView타입에 configuration담아서 리턴하기

```swift
class MyContentView: UIView, UIContentView {
    var configuration: UIContentConfiguration {
        didSet {
            self.configure(configuration: configuration)
        }
    }
    
    let label = UILabel()
    let secondLabel = UILabel()
    let imageView = UIImageView()
    
        //configuration이 주는 정보를 가지고 자신이 가진 뷰를 configure함
    init(_ configuration: UIContentConfiguration) {
        self.configuration = configuration
        super.init(frame: .zero)
        self.addSubview(label)
        self.addSubview(secondLabel)
        self.addSubview(imageView)
        
        
        setupLayout()
        self.configure(configuration: configuration)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func configure(configuration: UIContentConfiguration) {
        guard let configuration = configuration as? MyContentConfiguration else { return }
        self.label.text = configuration.text
        self.secondLabel.text = configuration.secondaryText
        self.imageView.image = configuration.image
    }
    
    func setupLayout() {
        label.translatesAutoresizingMaskIntoConstraints = false
        secondLabel.translatesAutoresizingMaskIntoConstraints = false
        imageView.translatesAutoresizingMaskIntoConstraints = false
        
        NSLayoutConstraint.activate([
            imageView.topAnchor.constraint(equalTo: self.topAnchor, constant: 10),
            imageView.bottomAnchor.constraint(equalTo: self.bottomAnchor, constant: -10),
            imageView.leadingAnchor.constraint(equalTo: self.leadingAnchor, constant: 10),
            imageView.trailingAnchor.constraint(equalTo: label.leadingAnchor, constant: -10),
            //1:1 주는 방법
            imageView.heightAnchor.constraint(lessThanOrEqualToConstant: 100),
            imageView.widthAnchor.constraint(equalTo: imageView.heightAnchor),
            
            
            label.topAnchor.constraint(equalTo: self.topAnchor, constant: 10),
            label.bottomAnchor.constraint(equalTo: secondLabel.topAnchor, constant: -10),
            label.trailingAnchor.constraint(equalTo: self.trailingAnchor, constant: -10),
            label.widthAnchor.constraint(equalTo: imageView.widthAnchor, multiplier: 4),
            
            secondLabel.bottomAnchor.constraint(equalTo: self.bottomAnchor, constant: -10),
            secondLabel.leadingAnchor.constraint(equalTo: label.leadingAnchor),
            secondLabel.widthAnchor.constraint(equalTo: label.widthAnchor, multiplier: 1)
        ])
    }
}
```

```swift
struct MyContentConfiguration: UIContentConfiguration {
    var text = ""
    var secondaryText = ""
    var image: UIImage?
    
    func makeContentView() -> UIView & UIContentView {
        return MyContentView(self)
    }
    
    func updated(for state: UIConfigurationState) -> MyContentConfiguration {
        return self
    }
}
```

그리고 나서 뷰컨의 setupCollectionView() 메서드 안에서 cell registration을 위에서 만들어준 content configuration을 통해 해준다.

2. diffableDataSource의 init(collectionView:cellProvider:) 이니셜라이저의 cellProvider 클로저에서 컬렉션뷰에서 셀을 디큐하여 2의 registration을 사용해서 셀을 configure한다

```swift
func setupCollectionView() {
//1. registration 셀을 등록: 어떤 타입의 셀을 사용할지, object를 사용할지 넘겨줘야함: 그럼 리스트셀, 인덱스패스, object가 클로저로 넘어옴
    let registration = UICollectionView.CellRegistration<UICollectionViewListCell, ProductDetail> { cell, indexPath, productDetail in
      var content = MyContentConfiguration()
      content.text = productDetail.name
      content.secondaryText = productDetail.currency + String(productDetail.bargainPrice)

      let imageURL = URL(string: productDetail.thumbnail)!
      let imageData = try! Data(contentsOf: imageURL)
      content.image = UIImage(data: imageData)
      
      cell.contentConfiguration = content
      cell.accessories = [.checkmark()]
      }
    
 //2. 셀 디큐할때 registration 설정한 것 셀에 적용해서 꺼내오기
    dataSource = UICollectionViewDiffableDataSource<Section, ProductDetail>(collectionView: mainView.collectionView) { collectionView, indexPath, productDetail in
        let cell = collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: productDetail)
        return cell
    }
}
```

3. NSDiffableDataSourceSnapshot

스냅샷을 만들려면 섹션과 데이터가 필요하다

스냅샷을 만들어서 dataSource 프로퍼티에 할당하기 

```swift
enum Section {
   case main
}
```

```swift
func populate(with productDetail: [ProductDetail]) {
    var snapshot = NSDiffableDataSourceSnapshot<Section, ProductDetail>()
    snapshot.appendSections([.main])
    snapshot.appendItems(productDetail)
    dataSource?.apply(snapshot)
}
```

 <뷰컨트롤러 전체코드>

```swift
class ViewController: UIViewController {
    //뷰를 커스텀뷰로 만듦
    let mainView = MainView()
    var dataSource: UICollectionViewDiffableDataSource<Section, ProductDetail>?
    
    override func loadView() {
        view = mainView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupCollectionView()
        getUsers()
    }
    
    //컬렉션뷰의 셀 세팅
    func setupCollectionView() {
        let registration = UICollectionView.CellRegistration<UICollectionViewListCell, ProductDetail> { cell, indexPath, productDetail in
            
            //디폴트 UIListContentConfiguration에 여러가지 설정을 한 후
            var content = MyContentConfiguration()
            content.text = productDetail.name
            content.secondaryText = productDetail.currency + String(productDetail.bargainPrice)

            let imageURL = URL(string: productDetail.thumbnail)!
            let imageData = try! Data(contentsOf: imageURL)
            content.image = UIImage(data: imageData)
            
            cell.contentConfiguration = content
            cell.accessories = [.checkmark()]
        }
        
        dataSource = UICollectionViewDiffableDataSource<Section, ProductDetail>(collectionView: mainView.collectionView) { collectionView, indexPath, productDetail in
            let cell = collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: productDetail)
            return cell
        }
    }
    
    func getUsers() {
        Networking.checkProductList { result in
            switch result {
            case .success(let data):
                print(data)
                self.populate(with: data)
            case .failure(let error):
                print(error)
            }
        }
    }
    
    //3. snapshot 만듷
    func populate(with productDetail: [ProductDetail]) {
        var snapshot = NSDiffableDataSourceSnapshot<Section, ProductDetail>()
        snapshot.appendSections([.main])
        snapshot.appendItems(productDetail)
        dataSource?.apply(snapshot)
    }
    
}

extension ViewController {
    //enum - 기본값이 hashable함
    enum Section {
        case main
    }
}
```









