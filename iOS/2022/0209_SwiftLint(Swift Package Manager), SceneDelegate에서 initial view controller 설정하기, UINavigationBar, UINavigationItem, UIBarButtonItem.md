# 학습 내용
- Swift Package Manager로 SwiftLint 프로젝트에 적용
- SceneDelegate에서 initial view controller 설정하기
- UINavigationBar, UINavigationItem, UIBarButtonItem

# swift package manager로 swiftLint 적용하기

cocoapods 를 통해 하는 방법도 있는데.. cocoapods는 조금 더 복잡하고 xcworkspace가 생겨서 패키지매니저가 나은 것 같다

적용 순서

1. Targets → Build Phases → Run Script에 아래 코드를 복붙해준다
Run Script 없으면 위에 + 버튼 누르고 New Run Script Phase
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62034bce9276e54c1b072969.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62034bce9276e54c1b072969.png)

```swift
if test -d "/opt/homebrew/bin/"; then
  PATH="/opt/homebrew/bin/:${PATH}"
fi

export PATH

if which swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi

${PODS_ROOT}/SwiftLint/swiftlint
```

1. `.swiftlint.yml`파일 생성하여 제외시킬 파일이나 규칙을 명시해준다.

`.`을 파일명 앞에 붙이면 숨김파일이라는 뜻이다

`cmd + shift + .` : 숨김파일보기 단축키

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62034bcd9276e54c1b072967.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62034bcd9276e54c1b072967.png)

```swift
included:
  - CloudNotes # 프로젝트 이름
excluded: # 린트 과정에서 무시할 파일 경로. `included`보다 우선순위 높음
  - CloudNotes/AppDelegate.swift
  - CloudNotes/SceneDelegate.swift
identifier_name:
    min_length:
    - 2  # only min_length
    error: 4 # only error
    excluded: # 제외할 문자열 목록 사용
    - id
    - URL
    - url
    - GlobalAPIKey
disabled_rules: # 실행에서 제외할 룰 식별자들
  - colon
  - control_statement
  - trailing_whitespace
  - vertical_parameter_alignment
opt_in_rules: # 일부 룰은 옵트 인 형태로 제공
  - empty_count
  - conditional_returns_on_newline
```

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62034bcd9276e54c1b072965.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62034bcd9276e54c1b072965.png)

# Reference

[https://tngusmiso.tistory.com/59](https://tngusmiso.tistory.com/59)


# SceneDelegate에서 initial view controller 설정하기

1. SceneDelegate의 scene메서드 안에서 scene을 활용해 UIWindow를 생성한 후에 
2. UIWindow의 rootViewController를 제일 처음 보여지게 하고싶은 뷰컨트롤러로 설정한다
3. window?.makeKeyAndVisible() 으로 window를 key window로 만들어서 화면에 보여지게 한다

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b2b9276e54c1b072987.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b2b9276e54c1b072987.png)
    

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windoewScene = (scene as? UIWindowScene) else {
            return
        }
        window = UIWindow(windowScene: windoewScene)
        let splitVC = SplitViewController(style: .doubleColumn)
        window?.rootViewController = splitVC
        window?.makeKeyAndVisible()
    }
}
```

# **UINavigationBar,** UINavigationItem, UIBarButtonItem

## n**avigationBar**

네비게이션 컨트롤러꺼

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b949276e54c1b072990.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b949276e54c1b072990.png)

navigationBar 는 보통 네비게이션 컨트롤러가 가지고있음 (근데 네비게이션 컨트롤러 없이도 사용가능)

네비게이션 컨트롤러는 navigationBar의 모습을 configure하고

뷰컨트롤러는 navigationItem 프로퍼티를 통해 navigationBar의 컨텐츠를 configure한다.

## navigationItem

뷰컨꺼

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b959276e54c1b072994.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b959276e54c1b072994.png)

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b8f9276e54c1b07298c.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b8f9276e54c1b07298c.png)

navigationItem은 뷰컨의 부모(네비게이션 컨트롤러)가 가지고있는 navigation bar를 대신하는 인스턴스임 그래서 뷰컨은 navigationItem 프로퍼티를 가지고 있다

뷰컨에서 이 프로퍼티가 처음 접근되는 그 순간 UINavigationItem 객체가 생성이 된다고한다.

따라서 뷰컨을 화면에 띄울때 네비게이션컨트롤러를 쓰지 않는다면 이 프로퍼티에 절대 접근하면 안된다. 

### 이해가 안되는 부분

근데.. 뷰컨의 뷰를 생성할때 네비게이션 아이템에 바버튼 아이템을 생성하지 말라는게 무슨 소리지

네비게이션 스택에 두개의 뷰컨이 push되었을때 가장 위의 뷰컨이 보여지고 네비게이션 바에 백버튼을 띄우기 위해서 뒤 뷰컨의 네비게이션 아이템인 title이 가져와진다는 말은 이해가 가는데..

그니깐 뷰컨의 뷰랑 네비게이션 아이템은 독립적으로 가져와지므로 같이 생성하지 말라는건가...

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b959276e54c1b072996.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035b959276e54c1b072996.png)

## UIBarButtonItem

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035c759276e54c1b072999.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220209/62035c759276e54c1b072999.png)

뷰컨이 가지고있는 UINavigationItem 인스턴스는 backBarButtonItem, leftBarButtonItem 등 프로퍼티를 가지고 있고 그 프로퍼티들의 타입이 바로 UIBarButtonItem이다

따라서 뷰컨에서 이렇게 네비게이션 아이템의 오른쪽 바버튼아이템을 설정해줄 수 있다.
```swift
private func setUpNavigationItem() {
    navigationItem.title = "메모"
    navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: nil)
}
```
