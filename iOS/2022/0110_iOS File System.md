## SandBox

각각의 역할을 하는 파일들을 어떤 영역에 위치시키는가에 대한 내용

iOS는 SandBox 구조를 가지고 있음

- 샌드박스 구조가 **아닌** 경우 모든 사용자 데이터나 모든 시스템 자원에 제한 없이 접근 가능하다.  ex) 안드로이드.
- 샌드박스 구조인경우...
    
    앱에다가 사용할 수 있는 영역을 정해주고 넌 이 영역 안에서만 해야해 (파일 읽고쓰기, 시스템 자원에 접근/활용하기.. 에 대해서도)
    
    → 앱에서는 앱 사용자 데이터, 앱 시스템 자원에 대해서만 접근가능함 
    
    → 그 외 사용자 데이터나 시스템 자원은 접근 불가능함 
    
    ex) 사진첩 - 앱에서 사진첩에 있는 사진을 함부로 읽거나 쓰거나 변경할수없음 권한이 있어야된다.
    

### App의 Sandbox내에는

- Bundle Container : 앱을 실행할 수 있는 파일 모음
- Data Container: Documents/Library/tmp
- iCloud Container

## Data Container

이 중에서 오늘은 Data Container 영역을 볼 것이다.

크게 세가지로 나눌수있음 : Documents, Library, tmp

![스크린샷 2022-01-10 오후 11.32.36.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/168d67b1-436d-4d78-8d2e-4dcae4b9dd90/스크린샷_2022-01-10_오후_11.32.36.png)

## Documents

**사용자에게 노출되는 유일한 영역** !!

- 사용자 생성 콘텐츠 저장
- 사용자가 수정, 추가, 삭제 가능
- iCloud, iTunes 백업
- Inbox

## Library

**사용자에게 직접노출 없음**

- 하위 디렉터리 마음대로 생성 가능
- iCloud, iTunes 백업
- 앱을 운용하기위한 앱이 만든 앱 전용 데이터들

![파인더 - 이동 - option 누르면 라이브러리 생김 ](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5ef5e52-5b30-4da4-9aea-7bd89bbc529d/스크린샷_2022-01-10_오후_11.38.21.png)

파인더 - 이동 - option 누르면 라이브러리 생김 

### Library - Caches

- 앱이 실행되는 중이 아닌 경우에 디스크 용량 부족시 운영체제가 필요 없어지면 청소
- 반복 사용되는 임시 데이터 저장
- iCloud, iTunes 백업제외

### Library - Application Support

- 애플리케이션 생성 데이터 저장
- iCloud, iTunes 백업

### Library - Application Support

- 애플리케이션 설정 데이터 저장
- 직접구정은 권장되지 않음
- iCloud, iTunes 백업

## tmp

- 재사용되지 않는 임시 데이터 저장
- 시스템이 주기적으로 청소
- iCloud, iTunes 백업 제외
- 큰 파일을 다운로드받다가 중간에 끊겨버리면 거기서 끊기도록
- 다운완료시 app support나 document로 옮겨주는 것

# 활동학습

### FileManager

```swift
// 경로 찾기
let fileManager = FileManager.default
let appSupportURL = fileManager.urls(for: .applicationSupportDirectory, in: .userDomainMask).first!
let documentsURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
let tmpURL = fileManager.temporaryDirectory

// 파일 생성
let fileAppSupportURL = appSupportURL.appendingPathComponent("abc.txt")
let fileDocumentsURL = documentsURL.appendingPathComponent("abc.txt")
let filetmpURL = tmpURL.appendingPathComponent("abc.txt")

// 파일에 내용 쓰기
let myTextString = NSString(string: "HELLO WORLD")
try? myTextString.write(to: fileAppSupportURL, atomically: true, encoding: String.Encoding.utf8.rawValue)
try? myTextString.write(to: fileDocumentsURL, atomically: true, encoding: String.Encoding.utf8.rawValue)
try? myTextString.write(to: filetmpURL, atomically: true, encoding: String.Encoding.utf8.rawValue)

// 내용 읽기
var text = try String(contentsOf: fileAppSupportURL, encoding: .utf8)
var text2 = try String(contentsOf: fileDocumentsURL, encoding: .utf8)

// 수정하기
text += " 수정1"
text2 += " 수정2"
try text.write(to: fileAppSupportURL, atomically: true, encoding: .utf8)
try text2.write(to: fileDocumentsURL, atomically: true, encoding: .utf8)

// 삭제하기
//try fileManager.removeItem(atPath: fileAppSupportURL.path)
try fileManager.removeItem(at: fileAppSupportURL)
try fileManager.removeItem(at: fileDocumentsURL)
try fileManager.removeItem(at: filetmpURL)
```

### **[#](https://camp.yagom-academy.kr/board/6163034ad5194a7819ddeddd/articles/61db91f4e4081120ba788bfd#application-supportlibrary-document-tmp) Application Support(Library) / Document / tmp**

- 동영상 플레이어의 동영상 파일 : Application Support(Library) / Document유저가 파일로 볼 수 있을지 여부에 따라 나뉠 것 같음 (불가 / 가능)
- 웹 서버에서 받아온 임시 데이터 : tmp
- 사용자가 작성한 임시 메모 : Application Support(Library)

→ 사용자가 작성했는데 저장하기 전에 임시로 저장하는 것 ,, 

- 메모장 앱의 코어데이터 저장소 : Application Support(Library)
- 내 일기를 외부로 내보내기 위한 백업파일 : Document공유가능, user-generated
- 채팅 앱의 대화내역 : Application Support(Library)유저가 직접 저장하려는 의도를 갖지 않으면 백업되지 않음필요한 경우 Document로 옮겨서 추출
- 어젯밤 술취해 남겨둔 음성메모 : Application Support(Library)유저가 직접 저장하려는 의도를 갖지 않으면 백업되지 않음필요한 경우 Document로 옮겨서 추출
- 사진편집을 위해 사진첩에서 가져온 사진 : Document/Inbox외부 개체에 의해 오픈되도록 요청받은 파일에 접근할 때 사용편집App(외부) <- 사진첩App

Document - 사용자에게 파일로써 접근이 가능한 공간

App Support - 사용자에게 보여지지 않아야함
