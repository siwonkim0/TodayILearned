# 학습 내용
- CocoaPods
- RxSwift : Observable, Subject 차이 

### CocoaPods로 FireBase, RxSwift/RxCocoa 적용

1. 터미널에서 pod 활성화 
- pod init
- open Podfile
    pod 'Firebase/Analytics'
    pod 'RxSwift', '~> 6.5.0'
    pod 'RxCocoa', '~> 6.5.0'
    
- sudo arch -x86_64 gem install ffi
- arch -x86_64 pod install

2. .gitignore 에서 Pods/ 주석해제하여 CocoaPods는 push 안되게하기
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220301/621e0ca2536b5e1551d119f7.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220301/621e0ca2536b5e1551d119f7.png)

3. 프로젝트 pull받은 다른 팀원은 pod install 만 하면됨 

