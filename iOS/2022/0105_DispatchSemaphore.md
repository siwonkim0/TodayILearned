# 학습 내용
- DispatchSemaphore

## DispatchSemaphore
Semaphore는 signal()과 wait() 를 통해 값을 증가/감소 시키면서 특정 영역에 접근 가능한 작업의 수를 제한시킨다.

dataTask의 네트워크 통신은 비동기식이기 떄문에 프로그램이 네트워킹하는동안 기다려주지않고 다른 프로그램 구문을 실행시켜버리기때문에 semaphore을 걸지않으면 apiStatus가 변경되지 않은채로 출력이됨. 그래서 URLSession의 작업이 끝날때까지 기다리기위해 semaphore.wait()를 쓰고, 작업이 끝나면 semaphore.signal() 로 풀어준다. 

```swift
class APIManager {
    
    let apiHost: String = "https://market-training.yagom-academy.kr/"
    var apiStatus: String?
    let semaphore = DispatchSemaphore(value: 0)
    
    func healthChecker() {
        let url: String = apiHost + "healthChecker"
        var request = URLRequest(url: URL(string: url)!,timeoutInterval: Double.infinity)
        request.httpMethod = "GET"
        
        let task = URLSession.shared.dataTask(with: request) { data, response, error in
            guard let data = data else {
                self.semaphore.signal()
                print(String(describing: error))
                return
            }
            self.semaphore.signal()
            self.apiStatus = String(data: data, encoding: .utf8)!
                        //self.semaphore.signal() 여기서도 동작함 
        }
        task.resume()
        semaphore.wait()
    }
    
}
```
