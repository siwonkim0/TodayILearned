# Content compression resistance priority

줄어들어야하는 상황에서 안줄어들려는 힘 
두개의 뷰가 있다면, 그 중 저항하는 힘이 약한 뷰가 줄어든다
저항하는 힘이 세
면 안줄어듦

예시를 보자  
<img width="553" alt="스크린샷 2022-04-07 오후 12 05 00" src="https://user-images.githubusercontent.com/60725934/162112143-617e6ff4-257a-4181-bb20-5d3e3059e039.png">

저 `날짜 레이블` 2020.12.19 보다 `오른쪽 레이블`이 먼저 줄어들게 하기 위해서는

날짜의 content compression resistance priority를 옆 레이블보다 크게주면 됨 

스토리보드 상으로는 : 날짜 752 / 오른쪽 750 

# Content hugging priority

늘어나야하는 상황에서 안늘어나려는 힘

두개의 뷰 중 안늘어나려는 힘이 약한 뷰가 먼저 늘어난다

### 트러블슈팅

문제 : 내용의 첫줄이 짧을 때 날짜와 내용 미리보기 사이에 간격이 발생하는 문제가 생겼다.

원인 : 날짜 레이블과 내용 레이블의 content hugging priority가 같아서 생긴 문제

해결 : 날짜 레이블의 content hugging priority를 높여주었다.

```swift
label.setContentHuggingPriority(.defaultHigh, for: .horizontal)
```
<img width="590" alt="스크린샷 2022-04-07 오후 12 05 46" src="https://user-images.githubusercontent.com/60725934/162112203-961f4823-c928-4d57-b0d4-9fcf947a9eab.png">

# 오토레이아웃 코드로 구현하기

- 스택뷰를 안썼을때
<img width="558" alt="스크린샷 2022-04-07 오후 12 06 03" src="https://user-images.githubusercontent.com/60725934/162112229-d7e58b08-581c-4410-a5c4-cb1b6af8be81.png">

```swift
func setConstraints() {
    self.contentView.addSubview(titleLabel)
    self.contentView.addSubview(dateLabel)
    self.contentView.addSubview(bodyLabel)
    
    titleLabel.translatesAutoresizingMaskIntoConstraints = false
    dateLabel.translatesAutoresizingMaskIntoConstraints = false
    bodyLabel.translatesAutoresizingMaskIntoConstraints = false

    NSLayoutConstraint.activate([
        titleLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 5),
        titleLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: 5),
        titleLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 5),
        titleLabel.bottomAnchor.constraint(equalTo: dateLabel.topAnchor, constant: -5),
        dateLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
        dateLabel.trailingAnchor.constraint(equalTo: bodyLabel.leadingAnchor, constant: -5),
        bodyLabel.topAnchor.constraint(equalTo: dateLabel.topAnchor),
        bodyLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: 5)
    ])
    
    dateLabel.setContentCompressionResistancePriority(.defaultHigh, for: .horizontal)
    bodyLabel.setContentCompressionResistancePriority(.defaultLow, for: .horizontal)
}
```

- 주의점
1. 저 content compression resistance priority는  NSLayoutConstraint.activate 밖에서 실행해야함 ,, 모르고 걍 넣었다가 오류떠서 한참 헤맸네 
2. defaultHigh가 기본값이기때문에 dateLabel을 bodyLabel 보다 ResistancePriority를 크게 주려고했을때 bodyLabel의 ResistancePriority를 낮춰줬었는데, 이거보다 .init(1000)로 상수를 직접 할당하는 방법도 있다.

```swift
dateLabel.setContentCompressionResistancePriority(.init(1000), for: .horizontal)
```

- 스택뷰를 활용했을때
<img width="556" alt="스크린샷 2022-04-07 오후 12 06 25" src="https://user-images.githubusercontent.com/60725934/162112269-1bbf8c37-489b-4ed1-b853-67c0bfee7027.png">

스택뷰를 쓰지 않고 구현했을때는 모든 레이블 사이의 간격을 신경써줘야했으나, 스택뷰로 하니 가장 상위의 스택뷰와 contentView간의 간격 말고는 신경쓰지 않아도 되어서 훨씬 수월했다.

구현 순서는

1. override init , required init 
2. 각 레이블과 두개의 스택뷰 모두 프로퍼티로 선언한 후 클로저로 각각의 설정을 해준다
    1. translatesAutoresizingMaskIntoConstraints = false
    2. 폰트설정(accesibility 고려)
        1. 폰트 사이즈: .preferredFont(forTextStyle:)
        2. adjustsFontForContentSizeCategory = true
    3. 스택뷰의 경우 axis와 spacing 설정
    4. **setContentCompressionResistancePriority**, **setContentHuggingPriority 설정**
3. 뷰에 요소들을 추가해준다
    1. 가장 상위의 스택뷰를 contentview에 addSubView하고
    2. 제목 레이블과 하위 스택뷰(날짜, 내용)를 각각 상위의 스택뷰에 addArrangedSubView한다.

```swift
class MemoListCell: UITableViewCell {
    static let identifier = "MemoListCell"
    
    private let titleLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.font = .preferredFont(forTextStyle: .title2)
        label.adjustsFontForContentSizeCategory = true
        return label
    }()
    
    private let dateLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        **label.setContentCompressionResistancePriority(.init(1000), for: .horizontal)
        label.setContentHuggingPriority(.defaultHigh, for: .horizontal)**
        label.font = .preferredFont(forTextStyle: .body)
        label.adjustsFontForContentSizeCategory = true
        return label
    }()
    
    private let bodyLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.font = .preferredFont(forTextStyle: .subheadline)
        label.textColor = .systemGray
        label.adjustsFontForContentSizeCategory = true
        return label
    }()
    
    private let insideStackView: UIStackView = {
        let stackView = UIStackView()
        stackView.axis = .horizontal
        stackView.translatesAutoresizingMaskIntoConstraints = false
        stackView.spacing = 8
        return stackView
    }()
    
    private let outsideStackView: UIStackView = {
        let stackView = UIStackView()
        stackView.axis = .vertical
        stackView.translatesAutoresizingMaskIntoConstraints = false
        stackView.spacing = 5
        return stackView
    }()
    
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setUpViews()
        setUpConstraints()
        
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }

        //tableview datasource에서 불릴 메서드
        func configure(with item: MemoListInfo) {
        titleLabel.text = item.title
        bodyLabel.text = item.body
        dateLabel.text = item.lastModified
    }
    
    private func setUpViews() {
        contentView.addSubview(outsideStackView)
        outsideStackView.addArrangedSubview(titleLabel)
        outsideStackView.addArrangedSubview(insideStackView)
        insideStackView.addArrangedSubview(dateLabel)
        insideStackView.addArrangedSubview(bodyLabel)
    }
    
    private func setUpConstraints() {
        NSLayoutConstraint.activate([
            outsideStackView.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 12),
            outsideStackView.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -8),
            outsideStackView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            outsideStackView.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -4)
        ])
    }
}
```

# Reference

[https://ontheswift.tistory.com/21](https://ontheswift.tistory.com/21)
