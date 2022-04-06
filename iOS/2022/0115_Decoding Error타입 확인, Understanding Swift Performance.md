# 학습 내용
- 디코딩 에러타입 확인하기
- Understanding Swift Performance 활동학습

# 디코딩 에러타입 확인하기
잘 돌아가던 프로젝트가 갑자기 decoding error가 나면서 되지 않아서 무슨 에러인지 알아보기위해 에러의 종류를 찍어보았다.
Int타입인줄 알았던 price정보가 사실은 decimal이어서 double타입으로 모델을 바꿔줬더니 제대로 작동하였다. 

```swift
static func decodeData<T: Decodable>(of JSON: Data, type: T.Type) -> T? {
        let decoder = JSONDecoder()
        var decodedData: T?
        do {
            decodedData = try decoder.decode(type, from: JSON)
        } catch DecodingError.keyNotFound(let key, let context) {
            Swift.print("could not find key \(key) in JSON: \(context.debugDescription)")
        } catch DecodingError.valueNotFound(let type, let context) {
            Swift.print("could not find type \(type) in JSON: \(context.debugDescription)")
        } catch DecodingError.typeMismatch(let type, let context) {
            Swift.print("type mismatch for type \(type) in JSON: \(context.debugDescription)")
        } catch DecodingError.dataCorrupted(let context) {
            Swift.print("data found to be corrupted in JSON: \(context.debugDescription)")
        } catch let error as NSError {
            NSLog("Error in read(from:ofType:) domain= \(error.domain), description= \(error.localizedDescription)")
        }
        
        return decodedData
    }
```

# Understanding Swift Performance

## Abstraction

추상화

일반화된 부분중에 중요한 것을 추려내는 것 == 타입을 설계할때 올바르게 설계하자.

1. Allocation
2. Reference Counting
3. Method Dispatch

## Allocation

스택과 힙 

- 구조체에서는 [스택만 사용] `값`을 그대로 스택에 가지고있고
- 클래스에서는 [스택과 힙 둘다사용] 스택에 힙 영역을 가리키고있는 `주소값`을 가지게된다.

스택 영역을 사용하는 것이 성능상 유리하다.

힙 영역을 최대한 덜 사용하는 방법으로 가자

String(힙영역 사용)을 대체할 수 있는 방법

→ 구조체로 묶어주기 

cache 의 key가 String으로 되어있었는데 Attributes 구조체로 바꿔짐

*문자열의 공간을 미리 확보 불가능

INT는 64비트로 크기가 정해져있으니 메모리에 그 공간 미리 할당가능인데,,
String이란 변수안에는 문자열이 10글자일지.. 100글자일지.. 알수없어서 힙을 사용하게됨 

Q. let과 같이 상수로 문자열을 사용해도 여전히 내부적으로 힙을 사용하게되나?
→ 네. 컴파일 타임에 메모리공간을 예측 불가능해서요 

## Reference Counting

ref count하는 메모리 공간이 필요하다 ㅎ
근데 이걸 스택만 사용하는걸로 바꾸면 걍 복사하고 꿑이니 
`copy on right`필요 없으면 복사를 안함 .. 개별적으로 수정을 하려고 하는 경우에만 복사가 되어 성능상 좋다

### 구조체 안에 참조타입

구조체 안에 참조타입이 많으면 많을수록 성능이 저하된다...
클래스를 사용하는것보다 더 많은 성능저하가 일어남 
→ 저 참조타입들을 전부다 대체해줘야함 ㅎㅎ

해결방안?
String extension이었던 mimeType 를 값타입인 열거형으로 바꿔주면됨
여기선 이제 URL만 참조타입임 

## Method Dispatch

- Static : 컴파일 타임에 이미 메서드의 위치를 알아서 성능이 좋음
- Dynamic : 런타임에 메서드의 위치?를 찾아가야해서 성능이 좋지않음

### Static

컴파일러는 `인라이닝 최적화`를 함 ... 
컴파일 시점에 실행해야할 메서드를 미리 복사해서 붙여넣기가 해놓음
런타임에 코드를 점프 점프하지않고 바로 실행해야할 메서드를 복붙해놓음

### Dynamic

근데.. class의 inheritance 상속을 사용하게되면 어떤 문제가 생기느냐.
구조체에서는 Drawable 사용시 상속이 일어나지 않기때문에 걍 무조건 찾아가면되는데
클래스에서는 상속받은 애들중 누구인지 모르니 컴파일타임에 찾아갈수없다는게 단점임.... 

어떤 Drawable 호출할지모르니 런타임에서 결정이되게됨
코드를 미리 땡겨오는 인라이닝을 할 수 없음

따라서 클래스에 final 표시를 붙여줌 
→ Final Class 는 Struct처럼 Static Dispatch를 하게된다.
*상속을 하지 않겠다는거니깐 하위 클래스가 없기때문에 실행중에 찾아가지않고 컴파일 타임에 찾아갈수있음

## 클로저의 캡쳐리스트

클로저가 만들어지는 시점에 캡쳐를하고싶은 값을 리스트로 나타내줄수있음
클로저 안의 변수들은 **모두 스택영역의 값이 캡쳐**가됨 
근데 변수들이 `참조`타입인지 `값`타입인지에따라 `주소값`이 캡쳐될지 `값`이 캡쳐될지 달라지는 것임

- **indexPath**는 `값`타입이어서 (참조타입이 아니기때문에) 값만 캡쳐하게됨
- **tableView, cell** 은 (스택영역에 주소값이있는 `참조`타입이니) 인스턴스(주소값)만 캡쳐하고, 코드가 실행되는 시점에 결과값을 계산하게됨 
    
참조타입 캡쳐시 RC도 올리기때문에 순환참조 방지하려면,, weak나 unowned붙여주기
나중에 클로저가 실행될때도 주소가 존재해야하기때문에 RC를 올려서 안전하게 만드는것임 





