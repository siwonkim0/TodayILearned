### DataSource를 하나로 두면 TableView가 어떤건지 어케알지?

데이터소스는 뷰요소인 테이블뷰를 몰라야하는거 아닌가? 
음... 생성자 주입 & 분기처리를 통해 해결함
세개의 테이블뷰 배열을 뷰컨에서 주입받은 후에 
메서드 내에서 각각의 테이블뷰마다 분기처리를 해서 처리를 해준다

[https://stackoverflow.com/questions/33933921/setting-a-uitableview-data-source-and-delegate-in-separate-file-swift](https://stackoverflow.com/questions/33933921/setting-a-uitableview-data-source-and-delegate-in-separate-file-swift)

```swift
class MyTVDataSource: NSObject, UITableViewDataSource{
    private var tableView: [UITableView]
    private var items = ["Item 1","item 2","item 3","Item 4"]

    init(tableView: [UITableView]) {
        self.tableView = tableView
    }
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
                if tableView == tableView[0] {
                        return ~~~
                }
        return items.count

    }
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = ItemCell(style: .default, reuseIdentifier: "Cell")
        cell.label.text = items[indexPath.row]

        return cell
    }

}
```
## 상세보기 화면 재사용

뷰: ProjectDetailView

공통 뷰컨: ProjectDetailViewController

하위 뷰컨 2개: 

- AddProjectDetailViewController
- EditProjectDetailViewController

커스텀 뷰를 하나 만들어서 공통된 뷰컨에 적용하고

그 뷰컨을 

두개의 뷰컨(add 새로운 매모추가, edit 메모 수정)에서 상속받아서 같이 사용하도록 함
## isediting프로퍼티 사용하여 edit<->done버튼 구현하기
edit 버튼 클릭시 수정화면 나오게

DetailView의 viewDidLoad에서 초기 셋팅으로 모든 컴포넌트 수정권한을 꺼놓음

```swift
private func configureNavigationBar() {
        self.navigationItem.title = TitleText.navigationBarTitle
        navigationItem.rightBarButtonItem = editButtonItem
        navigationItem.leftBarButtonItem = UIBarButtonItem(barButtonSystemItem: .cancel, target: self, action: #selector(didTapCancelButton))
        navigationController?.navigationBar.backgroundColor = .systemGray6
    }
```

애플이 제공하는 editButtonItem으로 네비게이션 버튼을 설정한 뒤 editButton의 상태를 트래킹하는 메소드를 오버라이드하여 우리가 행할 액션들을 정의하였음

버튼을 누르면 바뀌는 editing 여부에 따라서 
컴포넌트 수정권한을 끄고 켜는 액션을 호출함 

```swift
override func setEditing(_ editing: Bool, animated: Bool) {
        super.setEditing(editing, animated: animated)
        if editing {
            projectDetailView.setEditingMode(state: true)
        } else {
            projectDetailView.setEditingMode(state: false)
        }
    }
```

[https://developer.apple.com/documentation/uikit/uiviewcontroller/1621498-isediting](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621498-isediting)
