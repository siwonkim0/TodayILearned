# 학습내용
- View Life Cycle
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba7868fa.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba7868fa.png)
view의 visibility 가 변하면, 이 네 함수들이 자동으로 호출된다. 

**viewWillAppear(*:)
viewDidAppear(*:)
viewWillDisappear(*:)
viewDidDisappear(*:)**

항상 사용시 override 키워드 + super 키워드로 UIViewController에 정의된 메서드를 불러오기

## View Will Appear and View Did Appear

### View Will Appear

뷰가 화면에 보이기 전에 불려지고 뷰가 화면에 보일때마다 불리는 메서드

언제 이 메서드 사용?

- 네트워크 요청 시작하기
- 뷰 업데이트하기 : status bar, navigation bar, table views...
- 스크린 방향에 adjust하기

### View Did Appear

뷰가 화면에 보여지고 나서 불려짐 

뷰가 불려질때마다 실행되지만, 몇초정도 걸리는 작업을 여기서 하면 좋다.

## View Will Disappear and View Did Disappear

### View Will Disappear

사용자가 [ 백버튼 사용, 탭 전환, 모달창을 켜거나 꺼서 ]스크린에서 벗어날 때 실행되는 메서드

언제 사용?

- 수정사항 저장
- 키보드 숨기기
- 네트워크 요청 취소하기

### View Did Disappear

새로운 뷰로 navigate 한 이후 불려짐

이 메서드 실행시 뷰가 사라진게 확실함

언제 사용?

- 오디오 재생 중지
- notification observer 제거하기

실험

뷰컨1 과 뷰컨2에 라이프사이클 메서드들 정의해놓고 화면 전환시 어떤 순서로 불러와지는지 보기

- 코드
    
    ```swift
    class ViewController: UIViewController {
    
        override func viewDidLoad() {
            super.viewDidLoad()
            print("ViewController - View Did Load")
        }
        
        override func viewWillAppear(_ animated: Bool) {
            super.viewWillAppear(animated)
            print("ViewController - View Will Appear")
        }
        
        override func viewDidAppear(_ animated: Bool) {
            super.viewDidAppear(animated)
            print("ViewController - View Did Appear")
        }
        
        override func viewWillDisappear(_ animated: Bool) {
            super.viewWillDisappear(animated)
            print("ViewController - View Will Disappear")
        }
        
        override func viewDidDisappear(_ animated: Bool) {
            super.viewDidDisappear(animated)
            print("ViewController - View Did Disappear")
        }
    }
    ```
    
    ```swift
    class SecondViewController: UIViewController {
    
        override func viewDidLoad() {
            super.viewDidLoad()
            print("SecondViewController - View Did Load")
        }
        
        override func viewWillAppear(_ animated: Bool) {
            super.viewWillAppear(animated)
            print("SecondViewController - View Will Appear")
        }
        
        override func viewDidAppear(_ animated: Bool) {
            super.viewDidAppear(animated)
            print("SecondViewController - View Did Appear")
        }
        
        override func viewWillDisappear(_ animated: Bool) {
            super.viewWillDisappear(animated)
            print("SecondViewController - View Will Disappear")
        }
        
        override func viewDidDisappear(_ animated: Bool) {
            super.viewDidDisappear(animated)
            print("SecondViewController - View Did Disappear")
        }
    
    }
    ```
    
- 뷰컨1에서 뷰컨2로 화면전환했을 때 실행결과
    
    뷰컨2의 will appear가 실행되고 나서 뷰컨1이 사라지고 뷰컨2가 보여진다
    
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba7868fc.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba7868fc.png)
    
- 다시 뷰컨2에서 뷰컨1로 화면전환했을 떄
    
    뷰컨1의 will appear가 실행되고 나서 뷰컨2가 사라지고 뷰컨 1이 보여진다
    
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba7868fe.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba7868fe.png)
    

뷰컨의 뷰가 view hierarchy에 추가되고 제거되는 방법

1. UIKit는 뷰가 로드됐는지 먼저 확인, 만약 로드 안됐으면 로드하고 viewDidLoad() 실행
2. view hierarchy에 추가하기 전에 viewWillAppear() 실행
3. 이제 더이상 화면에 디스플레이 안될 뷰들을 제거하고 viewWillDisappear랑 didDisappear 실행
4. 마침내 새로운 뷰를 화면에 디스플레이하고 viewDidAppear() 실행

---

만국박람회 프로젝트에 적용 

첫번째 화면에는 네비게이션 바가 없고, 두번쨰 화면에서부터 보이게 하고싶었음 

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba786900.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb31c7e4081120ba786900.png)

첫번째 뷰컨

```swift
override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        self.navigationController?.setNavigationBarHidden(true, animated: true)
        print("NavigationBar OFF")
    }
```

두번째 뷰컨 

```swift
override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        self.navigationController?.setNavigationBarHidden(false, animated: true)
        print("NavigationBar ON")
    }
```

피드백 받고 수정한 사항

기존에는 첫번째 뷰컨에서 네비게이션바를 숨기고, 두번째 뷰컨에서 다시 표시했지만,, 

첫번째 뷰컨만이 네비게이션 바를 숨기는 책임을 다하도록 숨기고 표시하는 것으로 수정했다. 


![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb3235e4081120ba786904.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/6152fa94ccd9ef11a51aee7f/61bb3235e4081120ba786904.png)
첫번째 뷰컨에서만 동작

```jsx
override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        self.navigationController?.setNavigationBarHidden(true, animated: true)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        self.navigationController?.setNavigationBarHidden(false, animated: true)
    }
```
