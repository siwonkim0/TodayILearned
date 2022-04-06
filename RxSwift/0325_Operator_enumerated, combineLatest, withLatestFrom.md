# Operator

## enumerated

방출된 요소의 index를 사용하고 싶은 경우에 사용

```swift
example(of: "takeWhile") {
    let disposeBag = DisposeBag()
    
    Observable.of(2,2,4,4,6,6)
        .enumerated() // 인덱스를 사용가능
        .takeWhile({ index, value in // 해당 조건이 false일 때까지 emit
            value % 2 == 0 && index < 3
        })
        .map { $0.element }
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}

// 2, 2, 4 차례대로 출력
```

## combineLatest

여러 Observable을 합쳐서 하나의 Observable을 만드는 것 

타입이 달라도 합칠 수 있음 

## withLatestFrom

첫번째 `Observable`: tap 이벤트

두번째 `Observable`: 이벤트에 대한 처리

두 `Observable`을  `withLatestFrom` 연산자를 이용해 결합하면,

tap 이벤트가 발생 했을 때 필요한 데이터를 발생시키거나 관련된 처리를 하도록 사용가능

로그인 버튼 선택시 usernameTextField에 전달된 이벤트 발생

첫번째 옵저버블과 두번째 옵저버블을 합성하는 것이 아니라, 두번째 옵저버블의 이벤트만 그대로 발생시킴

```swift
let disposeBag = DisposeBag()
    
let loginButton  = PublishSubject<Void>()
let usernameTextField = PublishSubject<String>()
    
loginButton
    .withLatestFrom(usernameTextField)
    .subscribe(onNext: { print($0) })
    .disposed(by:disposeBag)
    
usernameTextField.onNext("tome.kye")
    
loginButton.onNext(())      // "tome.kye" 아이템 방출
    
usernameTextField.onNext("tommy.kye")
    
loginButton.onNext(())      // "tomme.kye" 아이템 방출
```

## map

이벤트 타입을 바꿔줄 때 쓰임

예를 들어, Observable<Int> → Observable<String> 으로 변환,,

## combineLatest, withLatestFrom, map 적용 예시

- combineLatest

observable<String>, observable<String>, observable<Date>타입인

input.projectTitle, input.projectBody, input.projectDate 를 결합해서

observable<String, String, Date> 타입인 projectDetail을 만듦

- withLatestFrom

didTapdoneButton의 tap이벤트 발생시 projectDetail의 이벤트 발생시키도록 함

- map

observable<String, String, Date> 타입을 observable<Project>타입으로 변환함

- compactMap

nil 제거

```swift
func transform(input: Input) -> Output {
    let projectDetail = Observable.combineLatest(input.projectTitle, input.projectBody, input.projectDate)
    
    let create = input.didTapdoneButton.withLatestFrom(projectDetail)
        .map { (title, body, date) in
            return Project(id: UUID(), state: .todo, title: title, body: body, date: date)
        }
        .compactMap { $0 }
        .subscribe(onNext: { [weak self] project in
            self?.useCase?.append(project)
        }).disposed(by: disposeBag)
    
    return Output()
}
```
