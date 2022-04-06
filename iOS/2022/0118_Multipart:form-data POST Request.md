# POST Request
### 기능 명세
Request에 필수적으로 보낼 요소 세가지 : identifier, params, images
1. identifier는 header에
2. params는 body의 name에
   - params - Request의 Body가 될 부분이다.
name, descriptions, price, currency, secret가 필수로 요구되고 안쓰면 에러남
나머지 discounted_price와 stock는 보내도 되고 안보내도됨
3. images는 body의 name에
```swift
//1
request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "**identifier**")
```

```swift
//2
let parameters = [
            "**params**": """
        {
            "name": "아메리카노",
            "descriptions": "아이스 아메리카노",
            "price": "4100",
            "currency": "KRW",
            "stock": 306,
            "secret": "EE5ud*rBT9Nu38_d"
        }
    """
          ]
//....

if let parameters = params {
    for (key, value) in parameters {
        body.append("--\(boundary + lineBreak)") //boundary 시작이라고 알려줘야함
        body.append("Content-Disposition: form-data; name=\"\(key)\"\(lineBreak + lineBreak)")
        body.append("\(value + lineBreak)")
    }
}
```

```swift
//3
body.append("Content-Disposition: form-data; name=\"**images**\"; filename=\"\(1223456663).jpeg\"\(lineBreak)")
```

### 순서
1. post를 할 URL 생성해서 URLRequest 생성 
2. request를 구분지을 boundary 생성
3. request에 header 추가: Content-Type, identifier
4. request body에 들어갈 params 준비
   - JSONObject 타입(Data 타입)이어야함   
5. image 정보를 담을 Media 타입 생성 
6. 5의 Media타입을 이용하여 Request Body에 들어갈 images 준비
7. params, images, boundary를 이용해서 request body 생성
8. 완성된 request를 가지고 URLSession.shared.dataTask의 파라미터로 전달



### 코드 전문 

```swift
struct Media {
    let key: String
//    let fileName: String
    let data: Data
//    let mimeType: String
    
    init?(withImage image: UIImage, forKey key: String) {
        self.key = key
//        self.mimeType = "image/jpeg"
//        self.fileName = "\(arc4random()).jpeg"
        
        //data를 못가져오면 애초에 이니셜라이즈 하기도싫어서 실패가능으로 함
        guard let data = image.jpegData(compressionQuality: 0.7) else { return nil }
        self.data = data
    }
}
```

```swift
extension Data {
    //자신의 data 내용을 바꾸는거라 mutating 필요
    mutating func append(_ string: String) {
        if let data = string.data(using: .utf8) {
            append(data)
        }
    }
}
```

```swift
@IBAction func POSTRequest(_ sender: Any) {
    let parameters = [
        "params": """
    {
        "name": "아메리카노",
        "descriptions": "아이스 아메리카노",
        "price": "4100",
        "currency": "KRW",
        "stock": 306,
        "secret": "EE5ud*rBT9Nu38_d"
    }
"""
      ]
    guard let mediaImage = Media(withImage: UIImage(named: "testImage")!, forKey: "images") else {
        print("No Image")
        return }

    guard let url = URL(string: "https://market-training.yagom-academy.kr/api/products") else { return }
    var request = URLRequest(url: url)
    request.httpMethod = "POST"

    let boundary = generateBoundary()
    request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
    request.addValue("819efbc3-71fc-11ec-abfa-dd40b1881f4c", forHTTPHeaderField: "identifier")

    let dataBody = createDataBody(withParameters: parameters, media: [mediaImage], boundary: boundary)
    request.httpBody = dataBody

    let session = URLSession.shared.dataTask(with: request) { data, response, error in
        if let response = response {
            print(response)
        }
        if let data = data {
            do {
                let json = try JSONSerialization.jsonObject(with: data, options: [])
                print(json)
            } catch {
                print(error)
                print("haha")
            }
        }
    }
    session.resume()
    
}

//httpBody에 들어갈 Boundary 만들기
func generateBoundary() -> String {
    return "Boundary-\(NSUUID().uuidString)"
}

//httpBody에 들어갈 parameter, media 만들기
//media 에 이미지 몇개들어갈수도있음, 없을수도있음
func createDataBody(withParameters params: Parameters?, media: [Media]?, boundary: String) -> Data {
    let lineBreak = "\r\n"
    var body = Data()

    //parameter
    if let parameters = params {
        for (key, value) in parameters { //key는 "params"임
            body.append("--\(boundary + lineBreak)") //boundary 시작이라고 알려줘야함
            body.append("Content-Disposition: form-data; name=\"\(key)\"\(lineBreak + lineBreak)")
            body.append("\(value + lineBreak)")
        }
    }

    //media
    if let media = media {
        for photo in media {
            body.append("--\(boundary + lineBreak)")
            body.append("Content-Disposition: form-data; name=\"\(photo.key)\"; filename=\"\(1223456663).jpeg\"\(lineBreak)")
            body.append("Content-Type: image/jpeg, image/jpg, image/png" + "\(lineBreak + lineBreak)")
            body.append(photo.data)
            body.append(lineBreak)
        }
    }
    body.append("--\(boundary)--\(lineBreak)") //boundary끝났다고 알려줘야함

    return body
}
```

# Reference
https://www.youtube.com/watch?v=8GH0yMPvQFU
