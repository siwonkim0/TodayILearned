## 깃허브 토큰 설정
팀 프로젝트 한 것을 내 repository에 가져와서 잔디를 심을려고 했는데 맥북을 새로 사서 그런지 토큰을 입력하라는 메세지가 나왔다,,
![](https://i.imgur.com/qWx4iED.png)

깃허브 settings - developer settings - personal access tokens에 가서 토큰 생성해줌
![](https://i.imgur.com/xdAVHQP.png)

그런데 이런 방식으로 토큰을 만들고 터미널 비번창에 토큰을 입력했는데도 계속 접근 권한이 없다는 에러가 뜨는 것이다. 
![](https://i.imgur.com/dmHyjA7.png)

알고보니 여기를 체크를 안해서 repository에 접근을 못하는 것이었다... 
![](https://i.imgur.com/JF7iUtN.png)

## 깃허브 Mirror Push

- 잔디심기용 repository를 생성하기
- 잔디를 가져오길 원하는 repository를 클론해오기
```swift
git clone --bare https://github.com/hwangjeha/ios-juice-maker.git
```

- 내 계정에 미리 만들어놓은 잔디심기용 repository에 mirror push 하기
```swift
git push --mirror https://github.com/siwonkim0/jandi.git
```
