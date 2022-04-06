### Data(contentsOf:) URL로 이미지 불러오기,

개선 전 

```swift
  func imageDataDecoder(urlString: String) -> UIImage {
      let imageURL = URL(string: urlString)!
      let imageData = try? Data(contentsOf: imageURL)
      let image = UIImage(data: imageData!)!
      
      return image
  }
```

개선 후 

```swift
func setupImage(with productImage: Image) {
    guard let imageURL = URL(string: productImage.url) else { return }
    URLSession.shared.dataTask(with: imageURL) { data, _, _ in
        let image = UIImage(data: data!)!
        DispatchQueue.main.async {
            self.productImageView.image = image
        }
    }.resume()
}
```

[https://developer.apple.com/documentation/foundation/nsdata/1413892-init](https://developer.apple.com/documentation/foundation/nsdata/1413892-init)

동기적으로 동작하는 메서드여서 현재 스레드를 block할 가능성이 있어서 큰 파일을 읽어올때는 적합하지 않다.

따라서 URLSession의 dataTask 메서드를 대신 사용해서 이미지를 가져와주었다.

### UICollectionViewDiffableDataSource 사용하여 Image를 CollectionView에 가져오기

기존 방식으로 collectionView의 UICollectionViewDataSource를 채택하여 cellForItemAt메서드를 사용해보려했지만, 데이터 통신의 결과로 가져온 이미지를 프로퍼티에 저장하고나서 cellForItemAt메서드에서 그 프로퍼티에 저장된 이미지를 가지고 셀을 configure 해줘야해서 순서가 맞지않아서

- images 프로퍼티를 옵셔널로주면 처음에 Nil인 상태로 실행이되어서 앱이 꺼짐
- images 프로퍼티에 빈배열을 주고 이미지가 빈상태로 화면을 구성한후 데이터를 받아오고나서 이미지를 넣으면 되긴한다(데이터 통신 부분에서 프로퍼티에 받아온 이미지 할당한후 reloadData 하면됨)

```swift
class ProductDetailViewController: UIViewController, UICollectionViewDataSource {

    //**var images: [Image]? -> 아예 처음에 nil이라서 앱이 실행자체가 안됨
    var images = [Image]() //빈 배열을 넣어줘야지 처음에 이미지 없는상태로 실행됐다가 데이터 받아오고 이미지가 들어감 (reloadData)**

  func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
      images.count
  }
  
  func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
      guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: ProductImageCollectionViewCell.identifier, for: indexPath) as? ProductImageCollectionViewCell else {
          return UICollectionViewCell()
      }
      
      cell.setupImage(with: (images[indexPath.row])!)
      return cell
  }

    override func viewDidLoad() {
      super.viewDidLoad()
      **imageCollectionView.dataSource = self**
      setupImageCollectionView()
      getProductDetail()
  }

    private func getProductDetail() {
      guard let productID = productID else { return }
      apiManager.checkProductDetail(id: productID) { result in
          switch result {
          case .success(let product):
              DispatchQueue.main.async {
                  self.setup(with: product)
                  **self.images = product.images!
                  self.imageCollectionView.reloadData()**
              }
          case .failure(let error):
              print(error.localizedDescription)
          }
      }
  }
}
```

하지만 모던한 방식으로 Diffable DataSource를 가지고 하면 데이터를 가져오는 순서가 상관이 없이 작동을 하는 것 같아서 이 방법을 사용했다. 사실.. 아직도 정확히 어떤식으로 작동하는지는 잘 모르겠지만 일단 흐름을 정리해놓는다,, 

1. 뷰컨에서 dataSource를 정의해준다

```swift
@IBOutlet weak var imageCollectionView: UICollectionView!
var imageDataSource: UICollectionViewDiffableDataSource<Section, Image>?
```

1. Section을 만들어준다

```swift
enum Section {
    case main
}
```

1. 컬렉션뷰에서 셀을 디큐해서 데이터소스를 채워넣는다 - 커스텀셀의 메서드로 셀에 데이터를 채워넣기

```swift
func setupImageCollectionViewDataSource() {
    imageDataSource = UICollectionViewDiffableDataSource<Section, Image>(collectionView: imageCollectionView) { collectionView, indexPath, product in
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: ProductImageCollectionViewCell.identifier, for: indexPath) as? ProductImageCollectionViewCell else {
            return UICollectionViewCell()
        }
        
        cell.setupImage(with: product)
        return cell
    }
}
```

1. 디큐할때 사용하는 커스텀셀 클래스도 만들어준다
- setupImage 메서드로 받아올 데이터 타입인 Image를 파라미터로 받아서 미리 데이터를 imageView에 넣는 로직을 구현해놓는다.

```swift
class ProductImageCollectionViewCell: UICollectionViewCell {
    
    @IBOutlet weak var productImageView: UIImageView!
    static let identifier = "ProductImageCell"
    
    func setupImage(with productImage: Image) {
        guard let imageURL = URL(string: productImage.url) else { return }
        URLSession.shared.dataTask(with: imageURL) { data, _, _ in
            let image = UIImage(data: data!)!
            DispatchQueue.main.async {
                self.productImageView.image = image
            }
        }.resume()
    }
}
```

1. 스냅샷을 찍는 populate 메서드를 만든다
- 여러장의 이미지를 가져올거니깐 [Image]
- 앞에서 설정해준 Section과 데이터인 Item을 append해준다.
- 데이터소스에 만든 snapshot를 적용해준다.

```swift
private func populate(with products: [Image]) {
    var snapshot = NSDiffableDataSourceSnapshot<Section, Image>()
    snapshot.appendSections([.main])
    snapshot.appendItems(products)
    imageDataSource?.apply(snapshot)
}
```

1. 데이터 통신을 해서 데이터를 가져온다
- checkProductDetail은 URLSession.shared.datatask 로 데이터통신을 하는 메서드
- id를 받아서 url을 만들어서 데이터를 가져온다
- 메인스레드에서 뷰 세팅하는 메서드인 setup을 호출한다
- setup 안에 populate가 들어가있음
- getProductDetail과 setupImageCollectionViewDataSource을 뷰디드로드에서 실행한다.

```swift
private func getProductDetail() {
    guard let productID = productID else { return }
    apiManager.checkProductDetail(id: productID) { result in
        switch result {
        case .success(let product):
            DispatchQueue.main.async {
                self.setup(with: product) //populate 여기서 실행
            }
        case .failure(let error):
            print(error.localizedDescription)
        }
    }
}
```

### 컬렉션뷰의 페이지 수 계산하기

스크롤을 넘길때마다 현재의 contentOffSet의 좌표를 계산해서 페이지수를 계산함 

offset 이란? 좌표임

contentOffset 이란? 스크롤뷰의 origin으로부터 떨어진 content view의 origin 의 거리

- 가로로 스크롤을 한번 하면 scrollView.contentOffset = (355, 0)

→ 컨텐츠뷰가 스크롤뷰의 원점대비 355 만큼 갔다는 소리  

- 두번 스크롤하면 scrollView.contentOffset = (710, 0)
- scrollView.frame.width 는 항상 355이다.

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220129/61f4d182f20b440d47f5aea7.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220129/61f4d182f20b440d47f5aea7.png)

```swift
extension ProductDetailViewController: UICollectionViewDelegate {
    // MARK: - Calculate Page Method
    func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
        let currentPage = Int(scrollView.contentOffset.x) / Int(scrollView.frame.width) + 1
        currentPageLabel.text = String(currentPage)
    }
}
```

근데 꼭 페이징이랑 같이 구현해야함

```swift
imageCollectionView.decelerationRate = .fast
imageCollectionView.isPagingEnabled = true
```

왜? 아니면 애매하게 스크롤이 멈춰서 scrollViewDidEndDecelerating 메서드가 불리기 어려움

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if segue.destination is AddProductViewController {
        let aa = storyboard?.instantiateViewController(withIdentifier: "AddProductViewController") as? AddProductViewController
        aa?.hello = nameLabel.text!
        
    }
}
```

```swift
@IBAction func editButton(_ sender: Any) {
//        performSegue(withIdentifier: "edit", sender: nil)
    
    let vc = storyboard?.instantiateViewController(withIdentifier: "EditViewController") as? EditViewController
    vc?.name = "aa"

    self.present(vc!, animated: true)
    
}
```
