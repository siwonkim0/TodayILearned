오픈마켓 프로젝트 get 메서드로 discountedPrice 받아오는것은 잘 됐는데 
이상하게 우리가 올리는것만 반영이 안됐다
우리 화면에서만 이상하게 보이는줄 알았는데 다른 캠퍼 시뮬레이터에서도 반영이 안되어서 보임
처음에는 post 로직이 잘못되었나 했는데 그랬으면 아예 post가 안됐을 것이라서..
알고보니 명세에서 선택적 값이고 기본값도 주고있어서 CodingKeys로 변환하지않아도 post가 됐었던 것이었다. 
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220127/61f26a172947bf40f08f3c20.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220127/61f26a172947bf40f08f3c20.png)
post시 다른 값들은 다 정상적으로 들어가는데 discountedPrice만 안들어갔던 이유는 
snake case로 키를 변환해주지 않아서 인코딩이 안되어서 서버에서 넣어주는 기본값 0이 들어갔기 때문 

방법 1 ) CodingKeys 사용 

```swift
struct NewProductInformation: Codable {
    let name: String
    let descriptions: String
    let price: Double
    let **discountedPrice**: Double
    let currency: Currency
    let stock: Int
    let secret: String
    
    init(name: String, descriptions: String, price: Double, discountedPrice: Double = 0, currency: Currency, stock: Int = 0, secret: String = "q?QZnt6zztVyw-NA") {
         self.name = name
         self.descriptions = descriptions
         self.price = price
         self.discountedPrice = discountedPrice
         self.currency = currency
         self.stock = stock
         self.secret = secret
     }
    
    enum CodingKeys: String, CodingKey {
        case discountedPrice = "discounted_price"
        case name, descriptions, price, currency, stock, secret
    }
}
```

방법 2 ) 인코더 설정 부분에서 converttonakecase 사용

```swift
let encoder = JSONEncoder()
encoder.keyEncodingStrategy = .convertToSnakeCase
```
