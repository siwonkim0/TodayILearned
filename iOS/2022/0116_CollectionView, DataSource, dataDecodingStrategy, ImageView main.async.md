# 학습 내용
- CollectionView에서 List <-> Grid 레이아웃으로 전환시 뷰가 느리게 전환됨
   - 해결 방안들
      - 1. ViewController의 view 바꿔끼우기
      - 2. UIImage 를 JSON데이터를 디코딩할때 불러오기
- UIImage: main.async 로 이미지가 느리게 불러와지는 경우 다른 뷰가 먼저 보이고 이미지는 나중에 보이게하기

## 문제 상황 : CollectionView에서 List <-> Grid 레이아웃으로 전환시 뷰가 느리게 전환됨
### 우리가 파악한 이유
collectionView와 dataSource를 각각 하나씩만 두고 layout과 cell을 list/grid 상황에 맞게 바꾸었더니
ListCollectionView와 GridCollectionView간에 전환을 시킬때 전환이 되고나서 뷰가 느리게 바뀌는 상황이 발생함
해결방법을 찾다가 공식문서를 보니, DiffableDataSource 로 데이터를 한번 configure하고나서 내용을 바꾸지 말라는 주의사항이 있었다
#### UICollectionViewDiffableDataSource 공식문서
Don’t change the dataSource on the collection view after you configure it with a diffable data source. If the collection view needs a new data source after you configure it initially, create and configure a new collection view and diffable data source.

- 개선 전 코드
: DataSource와 CollectionView를 하나씩 두고 버튼을 누르면 dataSource에서 각각의 셀을 디큐하도록 했다. 
-> dataSource를 세팅할때 클로저에서 상황에맞는 셀을 할당해주는것이 dataSource를 변경하는 것이라고 생각하여 개선해야겠다고 생각함
```Swift
let collectionView = UICollectionView(frame: .zero, collectionViewLayout: OpenMarketViewLayout.list)
var dataSource: UICollectionViewDiffableDataSource<Section, ProductDetail>?
var isListLayout = true

func setupCollectionView() {
        dataSource = UICollectionViewDiffableDataSource<Section, ProductDetail>(collectionView: collectionView) { collectionView, indexPath, product in
            if self.isListLayout {
                return self.productListCell(indexPath: indexPath, with: product)
            }
            return self.productGridCell(indexPath: indexPath, with: product)
        }
    }
  
func productListCell(indexPath: IndexPath, with product: ProductDetail) -> UICollectionViewCell {
    guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: ProductListCell.identifier, for: indexPath) as? ProductListCell else {
        return UICollectionViewListCell()
    }
    cell.setup(with: product)
    return cell
}

func productGridCell(indexPath: IndexPath, with product: ProductDetail) -> UICollectionViewCell {
    guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: ProductGridCell.identifier, for: indexPath) as? ProductGridCell else {
        return UICollectionViewListCell()
    }
    cell.setup(with: product)
    return cell
}
```
```Swift
@IBAction func segementChanged(_ sender: UISegmentedControl) {
    switch sender.selectedSegmentIndex {
    case 0:
        isListLayout = true
        collectionView.collectionViewLayout = OpenMarketViewLayout.list
        collectionView.reloadData()
    case 1:
        isListLayout = false
        collectionView.collectionViewLayout = OpenMarketViewLayout.grid
        collectionView.reloadData()
    default:
        return
    }
}
```

- 개선 후 코드
: List의 DataSource와CollectionView, Grid의 DataSource와CollectionView를 각각 두고 버튼을 누를때마다 ViewController의 view 를 바꿔끼워서 선택한 view가 보이게 하였음
```Swift
let listCollectionView = UICollectionView(frame: .zero, collectionViewLayout: OpenMarketViewLayout.list)
let gridCollectionView = UICollectionView(frame: .zero, collectionViewLayout: OpenMarketViewLayout.grid)
var listDataSource: UICollectionViewDiffableDataSource<Section, ProductDetail>?
var gridDataSource: UICollectionViewDiffableDataSource<Section, ProductDetail>?

func setupListCollectionView() {
    listDataSource = UICollectionViewDiffableDataSource<Section, ProductDetail>(collectionView: listCollectionView) { collectionView, indexPath, product in
        
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: ProductListCell.identifier, for: indexPath) as? ProductListCell else {
            return UICollectionViewListCell()
        }
        cell.setup(with: product)
        return cell
    }
}

func setupGridCollectionView() {
    gridDataSource = UICollectionViewDiffableDataSource<Section, ProductDetail>(collectionView: gridCollectionView) { collectionView, indexPath, product in
        
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: ProductGridCell.identifier, for: indexPath) as? ProductGridCell else {
            return UICollectionViewListCell()
        }
        cell.setup(with: product)
        return cell
    }
}
```
```Swift
@IBAction func segementChanged(_ sender: UISegmentedControl) {
        switch sender.selectedSegmentIndex {
        case 0:
            view = listCollectionView
        case 1:
            view = gridCollectionView
        default:
            return
        }
    }
```

### 조언받은 이유
리뷰어께서 이미지 네트워크 통신시에 사용되는 Data(contentsOf:) 때문에 느리게 전환되는 것 같다고 지적해주셔서 네트워크 통신때 이미지까지 가져온 후 뷰단에서 이미 가져와진 이미지를 업데이트하도록 개선해보았다.

- 개선 전 코드 : 커스텀뷰 셀에서 ProductDetail의 thumbnail 프로퍼티에 담긴 String 타입의 URL을 이용하여 UIImage를 가져왔음 
```Swift
class ProductGridCell: UICollectionViewCell {
   private func setupImage(with product: ProductDetail) {
      guard let imageURL = URL(string: product.thumbnail),
           let imageData = try? Data(contentsOf: imageURL),
           let image = UIImage(data: imageData) else {
             print("잘못된 이미지 URL입니다.")
             return
       }
      gridThumbnailImage.image = image
  }
}
```

- 개선 후 코드
: dataDecodingStrategy 사용해서 JSON데이터에서는 String타입인 thumbnail을 Data타입으로 변경
```Swift
struct ProductDetail: Codable, Hashable {
    //...
    let thumbnail: Data
    //...
}
```
```Swift
enum JSONParser {
    
    static func decodeData<T: Decodable>(of JSON: Data, type: T.Type) -> T? {
        let decoder = JSONDecoder()
        
        var decodedData: T?
        do {
            decoder.dataDecodingStrategy = .custom(imageDataDecoder)
            decodedData = try decoder.decode(type, from: JSON)
        } catch {
            print(error.localizedDescription)
        }
        
        return decodedData
    }
    
    static func imageDataDecoder(decoder: Decoder) throws -> Data {
        let container = try decoder.singleValueContainer()
        let str = try container.decode(URL.self)
        let imageData = try Data(contentsOf: str)
        
        return imageData
    }

}
```

```Swift
class ProductGridCell: UICollectionViewCell {
   func setupImage(with product: ProductDetail) {
        gridThumbnailImage.image = UIImage(data: product.thumbnail)
    }
}
```

## UIImage: main.async
: 이미지가 느리게 불러와지는 경우 다른 뷰가 먼저 보이고 이미지는 나중에 보이게하기

```Swift
func setupImage(with product: ProductDetail) {
    DispatchQueue.main.async {
        guard let imageURL = URL(string: product.thumbnail),
            let imageData = try? Data(contentsOf: imageURL),
           let image = UIImage(data: imageData) else {
               print("잘못된 이미지 URL입니다.")
               return
           }
        self.thumbnailImage.image = image
    } 
}
```

## Reference
[Custom JSON data decode](https://stackoverflow.com/questions/54812376/custom-json-data-decode-using-swift-4-jsondecoder-datadecodingstrategy-custom)
