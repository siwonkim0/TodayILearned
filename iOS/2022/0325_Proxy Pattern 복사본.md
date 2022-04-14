# Proxy Pattern
비즈니스로직을 수행하는 실제 객체가 들어갈 자리에 
대리자(Proxy)를 대신 투입하여 흐름제어 수행

### 사용 이유
- 흐름 제어
- 원본 객체가 실제로 사용될 때까지 생성 및 초기화를 위한 처리를 미루기 위해
원본 객체 대신 프록시를 노출하여 원본 객체에 대한 접근을 제어
- 큰 이미지나 그래픽같은 큰 데이터가 로딩되는데 시간이 걸리기때문에
처음부터 데이터를 로딩하지 않고, 필요한 순간에 로딩하기 위해 사용
- 원본 객체의 코드를 바꾸지 않고 추가적인 기능을 넣고 싶을 때  ex) 이름 변경 전 권한 체크

### 실 사용 예시
유튜브 재생목록에 마우스를 대면 프리뷰가 재생되는 기능 
예) 유튜브 비디오 프리뷰
처음에 영상 목록 → 이때는 Proxy 객체가 처리
마우스 올렸을 때 프리뷰 → 비로소 영상 객체 생성

### 작동 방식
- 원본 객체와 프록시는 같은 프로토콜을 채택하고
같은 이름의 메서드를 구현
- 프록시는 원본 객체의 인스턴스를 가짐
- 간단한 기능은 프록시가 처리하고
- 주요 기능은 프록시가 원본객체의 메서드를 호출해서 수행
이때 마치 원본객체가 프록시의 delegate 처럼 행동하여
원본객체에 비즈니스 로직이 있고 프록시는 그저 원본객체의 메서드를 호출

```swift
import Foundation

// Subject (Service Interface)
protocol YoutubeService {
    func showPreview()
    func setName(name: String)
}

// Real Service
class YoutubeVideo: YoutubeService {
    private var id: Int
    private var name: String
    
    init(id: Int, name: String) {
        self.id = id
        self.name = name
        
    }
    func showPreview() {
        print("\(name) Preview Play 😀\n")
        
    }
    func setName(name: String) {
        print("\(self.name)에서 \(name)으로 이름 수정 완료\n")
        self.name = name
      
    }
    
}
// Client
class Client {
    var auth: AccessAuth
    
    init(auth: AccessAuth) {
        self.auth = auth
    }
}

enum AccessAuth {
    case owner
    case guest
}

// Proxy
class YoutubeVideoProxy: YoutubeService {
    private var youtubeVideo: YoutubeService?
    private var id: Int
    private var name: String
    private var client: Client
    
    init(id: Int, name: String, client: Client) {
        self.id = id
        self.name = name
        self.client = client
    }
    
    func showTitle() {
        print("\(id), \(name) Youtube Video")
    }
    
    func showPreview() {
        if let youtubeVideo = self.youtubeVideo { youtubeVideo.showPreview()
            
        } else {
            loadVideoInfo()
            self.youtubeVideo?.showPreview()
        }
    }
    
    private func loadVideoInfo() {
        if let _ = self.youtubeVideo {
            print("Cache File Exist!\n")
            
        } else {
            print("load Video...🐧")
            self.youtubeVideo = YoutubeVideo(id: self.id, name: self.name)
        }
    }
    
    func setName(name: String) {
        if client.auth == .guest {
            print("Guest는 이름 수정 불가\n")
        } else if client.auth == .owner {
            self.loadVideoInfo()
            self.name = name
            self.youtubeVideo?.setName(name: name)
        }
    }
}
```
클라이언트가 어떤 일에 대한 요청을 하면, YouTubeVideoProxy가 대신 YoutubeVideo의 request()메서드 호출을 하고 
그 반환 값을 클라이언트에게 전달합니다.

