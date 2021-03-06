# DELETE
## 기능 명세

상품을 삭제하기 위해서는 post를 통해 상품의 secret를 조회한 후에
secret를 URL에 넣어서 삭제요청을 해야함 

## [POST] 2.2. 상품 삭제 Secret 조회

URL 형식(path)

```swift
{{api_host}}/api/products/{{product_id}}/secret
```

> 상품 삭제를 하기 위해 필요한 상품의 secret 을 조회합니다.
> 
> 
> 상품의 소유자는 판매자이기에 판매자를 식별할수 있는
> 
> `identifier` 를 요구합니다.
> 

## **Request**

[application/json](https://www.notion.so/df0dafde45704b2287f7fac10b2b75fb)

## **Response**

`text/plain`

`secret`

### **HEADERS**

**identifier**

80c47530-58bb-11ec-bf7f-d188f1cd5f22

### **BODY (raw)**

```swift
{"secret": "password"}
```

Example Request

```swift
import Foundation
#if canImport(FoundationNetworking)
import FoundationNetworking
#endif

var semaphore = DispatchSemaphore (value: 0)

let parameters = "{\"secret\": \"password\"}"
let postData = parameters.data(using: .utf8)

var request = URLRequest(url: URL(string: "{{api_host}}/api/products/{{product_id}}/secret")!,timeoutInterval: Double.infinity)
request.addValue("80c47530-58bb-11ec-bf7f-d188f1cd5f22", forHTTPHeaderField: "identifier")

request.httpMethod = "POST"
request.httpBody = postData

let task = URLSession.shared.dataTask(with: request) { data, response, error in 
  guard let data = data else {
    print(String(describing: error))
    semaphore.signal()
    return
  }
  print(String(data: data, encoding: .utf8)!)
  semaphore.signal()
}

task.resume()
semaphore.wait()
```

Example Response

```swift
2ceab673-58bf-11ec-8722-a5973153b704
```

# 프로젝트 코드

## [**POST] 2.2. 상품 삭제 Secret 조회**

```swift
func checkProductSecret(id: Int, completion: @escaping (Result<Data, Error>) -> Void) {
    guard let url = URLManager.checkProductSecret(id: id).url else { return }
    var request = URLRequest(url: url, method: .post)
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")
    request.httpBody = JSONParser.encodeToData(with: vendorSecret)
    createDataTask(with: request, completion: completion)
}
```

1. URLManager 이용해 명세대로 URL 생성함

```swift
enum URLManager {
    
    static let apiHost: String = "https://market-training.yagom-academy.kr/"
        //생략..
    case checkProductSecret(id: Int)
    
    var url: URL? {
        switch self {
        //생략..
        case .checkProductSecret(let id):
            return URL(string: URLManager.apiHost + "api/products/" + "\(id)/" + "secret")
        }
    }
    
}
```

2. URLRequest 를 확장하여 request도 생성함

```swift
extension URLRequest {
    init(url: URL, method: HttpMethod) {
        self.init(url: url)
        switch method {
        case .get:
            self.httpMethod = "GET"
        case .post:
            self.httpMethod = "POST"
        case .put:
            self.httpMethod = "PUT"
        case .patch:
            self.httpMethod = "PATCH"
        case .delete:
            self.httpMethod = "DELETE"
        }
    }
    
    enum HttpMethod {
        case get
        case post
        case put
        case patch
        case delete
    }
}
```

3. POST 라서 content-type를 header에 명시해줌

```swift
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
```

4. 명세에 나온대로 identifier을 header에 추가해줌

```swift
request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")
```

5. 명세에 나온대로 판매자 비밀번호인 secret을 body에 형식에 맞게 추가해줌

```swift
request.httpBody = Data("{\"secret\": \"EE5ud*rBT9Nu38_d\"}".utf8)
```

사실 원래 이렇게 JSON 형식으로 바로 Data()를 사용해서 변환해줘도 되는데 

JSONEncoder을 사용해서 인코딩하는 방식으로 코드 개선함

```swift
func checkProductSecret(id: Int, completion: @escaping (Result<Data, Error>) -> Void) {
    guard let url = URLManager.checkSecret(id: id).url else { return }
    var request = URLRequest(url: url, method: .post)
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")
    request.httpBody = JSONParser.encodeToData(with: vendorSecret)
    createDataTask(with: request, completion: completion)
}

struct VendorSecret: Codable {
    var secret: String = "EE5ud*rBT9Nu38_d"
}
```

```swift
static func encodeToData<T: Encodable>(with modelData: T) -> Data? {
    let encoder = JSONEncoder()
    
    var data: Data?
    do {
        data = try encoder.encode(modelData)
    } catch let error as NSError {
        NSLog("Error in read(from:ofType:) domain= \(error.domain), description= \(error.localizedDescription)")
    }
    return data
}
```

6. request를 받아서 데이터통신을 하는 메서드 재사용함

response로 받은 데이터가 그냥 text/plain 형식이고 secret키 하나만 와서 여기서는 디코딩을 따로 안해주기로 했음

```swift
func createDataTask(with request: URLRequest, completion: @escaping (Result<Data, Error>) -> Void) {
      let task = urlSession.dataTask(with: request) { data, response, error in
        if let httpResponse = response as? HTTPURLResponse,
           httpResponse.statusCode >= 300 {
            completion(.failure(URLSessionError.responseFailed(code: httpResponse.statusCode)))
            return
        }
        
        if let error = error {
            completion(.failure(URLSessionError.requestFailed(description: error.localizedDescription)))
            return
        }
        
        guard let data = data else {
            completion(.failure(URLSessionError.invaildData))
            return
        }

        completion(.success(data))
    }
    task.resume()
}
```

7. 메서드 호출부분에서 String(decoding:) 사용해서 JSON데이터를 String으로 변환함

```swift
private func getProductSecret() {
    guard let productID = productID else { return }
    apiManager.checkProductSecret(id: productID) { result in
        switch result {
        case .success(let data):
            self.secret = String(decoding: data, as: UTF8.self)
        case .failure(_):
            DispatchQueue.main.async {
                self.navigationItem.rightBarButtonItem = nil
            }
        }
    }
}
```

---

## **DEL2.3. 상품 삭제**

```swift
{{api_host}}/api/products/{{product_id}}/{{product_secret}}
```

상품 정보를 삭제합니다.
상품의 소유자는 판매자이기에 판매자를 식별할수 있는
`identifier` 를 요구합니다.
상품의 삭제를 위해서는 `identifier` 와 해당 상품의 `secret` 이 일치하여야합니다.

## **Request**

[application/json](https://www.notion.so/db1e073dcbea49acb89301e9f50994a1)

## **Response**

[application/json](https://www.notion.so/bc78c023f03c4462aea66a52a35770ea)

### **IMAGES**

[제목 없음](https://www.notion.so/b48a0e5582ff4f419e0e7ca82b1329be)

### **HEADERS**

**identifier**

80c47530-58bb-11ec-bf7f-d188f1cd5f22

Example Request

```swift
import Foundation
#if canImport(FoundationNetworking)
import FoundationNetworking
#endif

var semaphore = DispatchSemaphore (value: 0)

var request = URLRequest(url: URL(string: "{{api_host}}/api/products/{{product_id}}/{{product_secret}}")!,timeoutInterval: Double.infinity)
request.addValue("80c47530-58bb-11ec-bf7f-d188f1cd5f22", forHTTPHeaderField: "identifier")

request.httpMethod = "DELETE"

let task = URLSession.shared.dataTask(with: request) { data, response, error in 
  guard let data = data else {
    print(String(describing: error))
    semaphore.signal()
    return
  }
  print(String(data: data, encoding: .utf8)!)
  semaphore.signal()
}

task.resume()
semaphore.wait()
```

# 프로젝트 코드

## **DEL2.3. 상품 삭제**

```swift
func deleteProduct(id: Int, secret: String, completion: @escaping (Result<ProductDetail, Error>) -> Void) {
    guard let url = URLManager.deleteProduct(id: id, secret: secret).url else { return }
    var request = URLRequest(url: url, method: .delete)
    request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")
    createDataTaskWithDecoding(with: request, completion: completion)
}
```

1. 상품 secret 조회에서 얻은 상품 secret으로 URL 생성하기

```swift
enum URLManager {
    
    static let apiHost: String = "https://market-training.yagom-academy.kr/"
        //..
    case deleteProduct(id: Int, secret: String)
    
    var url: URL? {
        switch self {
                //..
        case .deleteProduct(let id, let secret):
            return URL(string: URLManager.apiHost + "api/products/" + "\(id)/" + "\(secret)")
        case .checkProductSecret(let id):
            return URL(string: URLManager.apiHost + "api/products/" + "\(id)/" + "secret")
        }
    }
    
}
```

2. URLRequest delete로 생성하기
3. 명세에 나온것처럼 identifier header에 추가하기

```swift
request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")
```

4. 어떤 상품이 삭제되었는지 상품 이름을 프린트하기위해서 데이터통신의 결과를 디코딩까지하는 메서드 사용하여 상품 삭제

```swift
func createDataTaskWithDecoding<T: Decodable>(with request: URLRequest, completion: @escaping (Result<T, Error>) -> Void) {
    let task = urlSession.dataTask(with: request) { data, response, error in
        if let httpResponse = response as? HTTPURLResponse,
           httpResponse.statusCode >= 300 {
            completion(.failure(URLSessionError.responseFailed(code: httpResponse.statusCode)))
            return
        }
        
        if let error = error {
            completion(.failure(URLSessionError.requestFailed(description: error.localizedDescription)))
            return
        }
        
        guard let data = data else {
            completion(.failure(URLSessionError.invaildData))
            return
        }
        guard let decodedData = JSONParser.decodeData(of: data, type: T.self) else {
            completion(.failure(JSONError.dataDecodeFailed))
            return
        }
        completion(.success(decodedData))
    }
    task.resume()
```

5. 메서드 호출부분에서 삭제성공시 상품이름 + 삭제완료 프린트하고, 노티피케이션으로 메인페이지와 상품상세페이지를 업데이트하는 post를 보내서 뷰를 업데이트함

```swift
private func deleteCurrentProduct() {
    guard let productID = productID, let secret = secret else { return }
    apiManager.deleteProduct(id: productID, secret: secret) { result in
        switch result {
        case .success(let products):
            DispatchQueue.main.async {
                print("\(products.name) 삭제 완료")
                NotificationCenter.default.post(name: NSNotification.Name("UpdateView"), object: nil)
                self.navigationController?.popViewController(animated: false)
            }
        case .failure(let error):
            print(error)
        }
    }
}
```
---
# PATCH
## **PATCH 2.1. 상품 수정**

## 기능 명세

```swift
{{api_host}}/api/products/{{product_id}}
```

> 상품 정보를 수정합니다.
> 
> 
> 상품 이미지 정보는 `3.2 상품이미지 추가` `3.3 상품이미지 교체` `3.5 상품이미지 삭제` 를 통해 가능합니다. → 상품 이미지는 여기서는 변경 못하도록함
> 
> 상품의 소유자는 판매자이기에 판매자를 식별할수 있는
> 
> `identifier` 를 요구합니다.
> 

---

## **Request**

[application/json](https://www.notion.so/2106acc4857242a481bdf9fbdd25ccdb)

## **Response**

[application/json](https://www.notion.so/b082f18be40f43ca84cbdc9b249c1fbf)

### **IMAGES**

[제목 없음](https://www.notion.so/83dc8a480bb44a6f88c8b455a00e7e6a)

### **HEADERS**

**identifier**

cd706a3e-66db-11ec-9626-796401f2341a

**BODYraw**

```swift
{
    "secret": "password",
    "discounted_price": 100
}
```

---

# 프로젝트 코드

## **PATCH 2.1. 상품 수정**

```swift
func editProduct(id: Int, product: NewProductInformation, completion: @escaping (Result<Data, Error>) -> Void) {
    guard let url = URLManager.editOrCheckProductDetail(id: id).url else { return }
    var request = URLRequest(url: url, method: .patch)
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")
    request.httpBody = JSONParser.encodeToData(with: product)
    createDataTask(with: request, completion: completion)
}
```

1. `application/json` 타입으로 request를 보내야하는데 여기서 필수요소가
    1. identifier → header
    2. product_id → path
    3. secret → body(params)
    
    이 세가지만 있으면 patch가 된다는것이다.
    
2. id는 URL에, identifier는 header에 추가하고

또 json타입 보낼거하고 header에 추가해주고 

```swift
  request.setValue("application/json", forHTTPHeaderField: "Content-Type")
  request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")
```

3. POST할때 사용했던 NewProductInformation 타입을 인코딩해서 body를 생성

여기 body에 수정하고싶은 데이터를 담고, 명세에 필수라고 되어있었던 secret도 담아 보낸다

```swift
struct NewProductInformation: Codable {
    let name: String
    let descriptions: String
    let price: Double
    let discountedPrice: Double
    let currency: Currency
    let stock: Int
    let secret: String
    
    init(name: String, descriptions: String, price: Double, discountedPrice: Double = 0, currency: Currency, stock: Int = 0, secret: String = "EE5ud*rBT9Nu38_d") {
         self.name = name
         self.descriptions = descriptions
         self.price = price
         self.discountedPrice = discountedPrice
         self.currency = currency
         self.stock = stock
         self.secret = secret
     }
}
```

```swift
static func encodeToData<T: Encodable>(with modelData: T) -> Data? {
    let encoder = JSONEncoder()
    
    var data: Data?
    do {
        data = try encoder.encode(modelData)
    } catch let error as NSError {
        NSLog("Error in read(from:ofType:) domain= \(error.domain), description= \(error.localizedDescription)")
    }
    
    return data
}
```

4. 메서드 호출부

상품 id와 NewProductInformation 타입인 상품 정보를 받아서 네트워크 통신을 통해 수정을 하고 

성공시 노티피케이션을 통해서 메인뷰와 상세보기뷰를 업데이트한다. 

```swift
private func patchEditedProduct(with information: NewProductInformation) {
    guard let id = product?.id else { return }
    apiManager.editProduct(id: id, product: information) { result in
        switch result {
        case .success(_):
            NotificationCenter.default.post(name: NSNotification.Name("UpdateView"), object: nil)
            DispatchQueue.main.async {
                self.dismiss(animated: true, completion: nil)
            }
        case .failure(let error):
            print(error.localizedDescription)
        }
    }
}
```
