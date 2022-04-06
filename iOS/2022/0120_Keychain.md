# 학습 내용
- Keychain
   - Adding a Password to the Keychain : 키체인에 비밀번호 등록
   - Searching for Keychain items: 비밀번호 찾기
   - Updating and Deleting Keychain items: 비밀번호 업데이트, 삭제

어케 동작하냐?
키체인에게 질문(query)을 날리면 답을 해주는 방식

꼭 attribute가 없어도 됨 ... 아이디, 서버주소 등 
iOS는 키체인이 전체적으로 하나고, MacOS는 앱마다 각각의 키체인을 가짐
iOS는 키체인을 수정이나 생성이 불가능한데...
MacOS는 개발자가 직접 키체인을 만들수가있음 
→ iOS가 보안이 훨씬 엄격하다는 이야기

키체인에는 하나의 비번만 담아놓을 수 있음
하나가 add로 등록이되어있으면 그걸 지우기 전까지는 아무리 다른것을 add해도 안들어감
그래서 삭제하고 다시 추가하든가, 아니면 아예 수정하는 방법도 있음 

[Apple Developer Documentation](https://developer.apple.com/documentation/security/keychain_services/keychain_items/using_the_keychain_to_manage_user_secrets)


## 키체인에 비밀번호 등록
- 정보를 담을 구조체 만들기
- 에러타입 만들기
- 쿼리 만들어서 정보담기
- 쿼리를 키체인에 더하기
    - `SecItemAdd(_:_:)` 사용
    

```swift
struct Credentials {
    var username: String
    var password: String
}

enum KeychainError: Error {
    case noPassword
    case unexpectedPasswordData
    case unhandledError(status: OSStatus)
}

func add() {
    let credentials = Credentials(username: "jane", password: "1234")
    let account = credentials.username
    let password = credentials.password.data(using: String.Encoding.utf8)!
    let query: [String: Any] = [kSecClass as String: kSecClassGenericPassword,
                                kSecAttrAccount as String: account,
                                kSecValueData as String: password]
    let status = SecItemAdd(query as CFDictionary, nil)
    guard status == errSecSuccess else { return }
    print("success")
}
```

## 키체인에서 비밀번호 찾기
→ 유저가 앱을 사용하다가 다른걸 하고돌아오거나 아니면 시간이 좀 지났을때 다시 재인증을 해야하는데 매번 유저한테 다시 로그인하라고하면 너무 번거로우니깐 키체인에서 알아서 찾아서 재인증해줌

- 쿼리 만들기
    - 이때 kSecReturnAttributes 항목을 꼭 써줘야함 .. 생략시 오류남
1에서는 유저네임 없어도 비밀번호가 등록이됐는데 왜 꼭 있어야하는지는 모르겠다,, 
![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220123/61ed21b4ce8df07969da36d8.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220123/61ed21b4ce8df07969da36d8.png)

- 쿼리를 이용해서 search하기
    `SecItemCopyMatching(_:_:)` 사용 
    
- 결과를 추출하기
    - kSecAttrAccount 로 attribute 추출
    - kSecValueData로 password data 추출
    
    ```swift
    func search() -> String? {
        let query: [String: Any] = [kSecClass as String: kSecClassGenericPassword,
                                    kSecMatchLimit as String: kSecMatchLimitOne,
                                    kSecReturnAttributes as String: true,
                                    kSecReturnData as String: true]
        var item: CFTypeRef?
        let status = SecItemCopyMatching(query as CFDictionary, &item)
        guard status != errSecItemNotFound else { print("errSecItemNotFound")
            return nil }
        guard status == errSecSuccess else { print("err")
            return nil }
        print(query)
        guard let existingItem = item as? [String : Any],
            let passwordData = existingItem[kSecValueData as String] as? Data,
            let password = String(data: passwordData, encoding: String.Encoding.utf8),
            let account = existingItem[kSecAttrAccount as String] as? String
        else {
            return nil
        }
        return password
    }
    ```
    
## 키체인 수정, 삭제하기

수정)

2에서 만든 search함수를 사용해서 키체인에서 등록된 비밀번호를 찾아온 후 
사용자가 지금 입력한 비번과 맞는지 비교한 후 에 맞으면 지금 입력한 비번으로 update하기 

- 쿼리 만들고
- 업데이트하는 함수인 `SecItemUpdate()` 사용

```swift
@IBAction func changePassword(_ sender: Any) {
        guard let previousPasswordFromKeyChain = search() else { return }
        guard let inputPreviousPassword = previousPassword.text else { return }
        guard previousPasswordFromKeyChain == inputPreviousPassword else {
            print("기존 비밀번호가 맞지 않습니다.")
            alert()
            return
        }
        
        do {
            try updateItemOnKeyChain()
            dismiss(animated: true, completion: nil)
        } catch {
            print(error.localizedDescription)
        }
    }
```

```swift
func updateItemOnKeyChain() throws {
        guard let newPassword = newPassword.text else { return }
        let credentials = Credentials(username: "jane", password: newPassword)
        let query: [String: Any] = [kSecClass as String: kSecClassGenericPassword]
//        let account = credentials.username
        let password = credentials.password.data(using: String.Encoding.utf8)!
        let attributes: [String: Any] = [kSecValueData as String: password]
        
        let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)
        guard status != errSecItemNotFound else { throw KeychainError.noPassword }
        guard status == errSecSuccess else { throw KeychainError.unhandledError(status: status) }
    }
```

삭제)

수정에서 만든 쿼리를 이용하여 SecItemDelete()함수로 삭제하기만 하면 된다 개쉬움
