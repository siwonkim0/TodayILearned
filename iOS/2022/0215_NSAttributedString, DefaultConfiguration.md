# NSAttributedString

NSAttributedString 을 이용해서 lastModified와 body를 묶어서 하나의 문자열로 생성함 

→ 한 이유: 커스텀 셀을 만들지 않고 defaultContentConfiguration 으로 셀을 만들기 위해서 (레이아웃 잡는 수고를 덜할 수 있어서)

![스크린샷 2022-02-15 오후 1.28.19.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/291cfec2-34aa-465e-ab78-2c5a9621aa6b/스크린샷_2022-02-15_오후_1.28.19.png)

```swift
struct Memo: Decodable {
    var title: String
    var body: String
    var lastModified: Date
}

extension Memo {
    var subtitle: NSAttributedString {
        
        let dateFormatter = DateFormatter()
        dateFormatter.locale = .current
        dateFormatter.dateStyle = .medium
        
        let dateString = dateFormatter.string(from: lastModified)
        let truncatedBody = body.truncated(limit: 40)
        
        let attributedString = NSMutableAttributedString() // "":빈문자열
        
        attributedString.append(NSAttributedString(
            string: dateString + " ",
            attributes: [.font: UIFont.preferredFont(forTextStyle: .footnote)]
        ))
        
        attributedString.append(NSAttributedString(
            string: truncatedBody,
            attributes: [
                .font: UIFont.preferredFont(forTextStyle: .caption1),
                .foregroundColor: UIColor.secondaryLabel
            ]
        ))
        return attributedString
    }
}
```

NSAttributedString 은 읽기 전용이라서 

빈 문자열을 먼저 생성한 후 append로 문자열들을 추가해주기위해 

NSMutableAttributedString 사용함 

# defaultContentConfiguration

셀의 레이블을 두개만 가지고 있도록 attributedString으로 미리 조작하여 

defaultContentConfiguration 사용하여 셀 생성함 

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: cellIdentifier, for: indexPath)
    
    var configuration = cell.defaultContentConfiguration()
    configuration.text = MemoDataManager.shared.memos[indexPath.row].title
    configuration.secondaryAttributedText = MemoDataManager.shared.memos[indexPath.row].subtitle
    configuration.textProperties.numberOfLines = 1
    configuration.secondaryTextProperties.numberOfLines = 1
    
    cell.contentConfiguration = configuration
    cell.accessoryType = .disclosureIndicator
    return cell
}
```

- textProperties, secondaryTextProperties 의 numberOfLines 사용하여 줄바꿈 설정

# 테이블뷰를 뷰컨트롤러에 추가해줄때

테이블뷰컨트롤러를 쓸때와 다르게 레이아웃을 잡아줘야한다. 

```swift
class MemoListViewController: UIViewController {
        private let tableView = UITableView()
        private func setupTableView() {
            view.addSubview(tableView)
                tableView.register(UITableViewCell.self, forCellReuseIdentifier: cellIdentifier)
            tableView.translatesAutoresizingMaskIntoConstraints = false
        
            NSLayoutConstraint.activate([
                tableView.topAnchor.constraint(equalTo: view.topAnchor),
                tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
                tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
                tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
            ])
        }
}
```
