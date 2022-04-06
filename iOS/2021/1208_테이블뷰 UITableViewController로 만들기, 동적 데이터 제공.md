# 학습 내용

- predefined UITableViewCell 사용하여 테이블 뷰 커스터마이징하기
- 테이블 뷰에 동적 데이터 제공하기

---

## Table View Controllers

테이블 뷰 만드는 두가지 방법

1. 뷰컨의 뷰에 table view instance 를 추가하기
- 테이블 뷰 말고도 다른 뷰들도 manage 가능
- 테이블 뷰의 위치와 사이즈 조정 가능
- 개발자가 data source 와 delegate objects 를 설정해야하는 의무가 있다

2. **UITableViewController** 사용 → **추천**
- 테이블 뷰가 전체 뷰를 차지하고, 크기를 조정 불가능
- table view controller가 data source 와 delegate 로써 행동한다 (?)

보통의 경우에는 table view controller 사용하는게 베스트다.

### UITableViewCell

- textLabel(`UILabel`)
- detailTextLabel(`UILabel`)
- imageView

### Prototype Cells 설정하기
![](https://i.imgur.com/5QSbd9U.png)


## Cell configuration

iOS 14.0 부터 생김

기존 방식처럼 UITableViewCell에 직접 접근하기보다는
셀이 무엇을 contain해야하는지 묘사해놓으면 알아서 자기 view를 업데이트할 수 있게 함 
→ 셀은 자신의 컨텐츠를 디스플레이하는데 컨트롤을 갖게되고, 우리는 보여주고싶은 데이터에 집중가능(?)

하는 방법)

`defaultContentConfiguration`

을 통해 미리 만들어진 디폴트 스타일을 불러 온 후에 스타일을 커스텀하고 컨텐츠를 할당한 후,

contentConfiguration에 할당하면 설정이 완료된다. 

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    //빈 셀을 넣어주고 하나하나 뺄때마다 새로운 이름표를 붙여서 나온다.
        let cell = tableView.dequeueReusableCell(withIdentifier: "rightdetail", for: indexPath) 
    // Configure the cell...
        var content = cell.defaultContentConfiguration()
    content.text = "Favorites"
    cell.contentConfiguration = content
      return cell
}
```

그럼 이 방법은 뭐지? 직접접근,, 나중에 deprecate 될 방법인듯...?

```swift
  var num = 0
  override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
      let cell = tableView.dequeueReusableCell(withIdentifier: "rightdetail", for: indexPath) //빈 셀을 넣어주고 하나하나 뺄때마다 새로운 이름표를 붙여서 나온다.
      num += 1
      cell.textLabel?.text = "\(num)"
      return cell
  }
```

---

### 데이터 Array

테이블 뷰에서 데이터를 가져올 때는 Array를 써라 →  ViewController의 프로퍼티로 가지고 있어야함 

Array의 장점?

- 순서가 있고
- count라는 프로퍼티를 사용 가능
- 0부터 시작
![](https://i.imgur.com/rcjtYfx.png)


### Cell dequeueing

수백개의 모델 객체를 화면에 보여줄 때 객체 각각을 위해 셀을 초기화한다면 메모리 부족으로 앱이 크래시날 것

→ 테이블뷰는 오로지 화면에 보이는 셀 + @ 를 로드한다

스크롤을 하면서 사라지는 셀의 레이아웃을 새로 보이는 셀에 reuse 함 

reuseIdentifier + 스토리보드에서 prototype cell 선택하기로 셀을 등록해놓을 수가 있고,

테이블뷰는 각각의 reuse identifier에 해당하는 셀 목록들을 관리한다.

그리고 `dequeueReusableCell(withIdentifier:for:)` 메서드로 reuse identifier에 해당하는 셀 을 꺼내온다. 

## Table View Protocols

- UITableViewDataSource
- UITableViewDelegate

data source 와 delegate는 거의 같은 객체이다.. (UITableViewController 의 서브클래스)

### Table View data source

UITableViewDataSource 프로토콜 채택

테이블 뷰 객체에 필요한 데이터를 제공해준다.

필요한 최소한의 정보?

- (섹션의 개수)
- 행의 개수
- 각 행에 뭐가 들어가는지 (셀의 내용)


```swift
optional func numberOfSections(in tableView: UITableView) -> Int

func tableView(_ tableView: UITableView, numberOfRowsInSection
   section: Int) -> Int

func tableView(_ tableView: UITableView, cellForRowAt
   indexPath: IndexPath) -> UITableViewCell
```

→ 테이블 뷰가 데이터를 로딩할 때 이 메서드들을 여러번 호출하면서 정보를 요청한다..

사용자가 스크롤을 하면 테이블뷰는 data source에 곧 보여질 행에 대한 정보를 계속 요청한다.

data source는 요청받은 정보에 대한 답을 할 의무가 있음 

1. `numberOfSections(in:)`

옵셔널임→ 만약 이 메서드 구현 안하면 걍 section = 1 (1로 지정할 것이라도 생략하는 것보다명확하게 정의하는게 낫다)

몇개의 섹션을 보여줄지 지정해줄수 있는 메서드

```swift
override func numberOfSections(in tableView: UITableView) -> Int {
    print("numberOfSections")
    return 1
}
```

2. `tableView(_:numberOfRowsInSection:)`

한 섹션에 대해 몇개의 row를 보여줄지 지정가능

파라미터로 section 도 있어서 section이 많아지면 각 세션에 몇개의 셀을 넣을지 이런것도 지정가능한듯

(필요없을땐 그냥 무시가능)

```swift
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    print("numberOfRowsInSection")
    return emojis.count //런타임에 사용자가 이모지 배열에 뭔가를 추가하면 바로 업데이트됨
}
```

3. `tableView(_:numberOfRowsInSection:)`

위 두 1, 2 메서드가 실행되면 테이블뷰는 이제 화면에 보여줄 섹션과 행이 몇갠지 알 수 있음

따라서 index path[섹션, 행] 를 제공해줄 수 있게 됨  

여기서 구현해야할 일

: 셀을 꺼내와서 설정을 해준 후 화면에 보여질 수 있도록 리턴하기

1)dequeue cell

```swift
//빈 셀을 넣어주고 하나하나 뺄때마다 새로운 이름표를 붙여서 나온다.
let cell = tableView.dequeueReusableCell(withIdentifier: "EmojiCell", for: indexPath) 
```

2)Fetch the model object to be displayed. 화면에 보여질 모델객체 가져오기

indexPath를 사용해서 array index 를 만들 수 있음

지금은 섹션이 하나라서 섹션 무시하고 row만 가져와도 된다.

```swift
let emoji = emojis[indexPath.row] //모델 객체를 가져옴
```

3)configure cell : 뷰 세팅 ex) label, image view...

```swift
var content = cell.defaultContentConfiguration()
content.text = "\(emoji.symbol) - \(emoji.name)"
content.secondaryText = emoji.description

cell.contentConfiguration = content
```

4)return cell to display 

```swift
return cell
```
실행 화면
![](https://i.imgur.com/l35leiJ.png)


전체코드

```swift
struct Emoji {
    var symbol: String
    var name: String
    var description: String
    var usage: String
}
```

```swift
class EmojiTableViewController: UITableViewController {
    var emojis: [Emoji] = [
       Emoji(symbol: "😀", name: "Grinning Face",
       description: "A typical smiley face.", usage: "happiness"),
       Emoji(symbol: "😕", name: "Confused Face",
       description: "A confused, puzzled face.", usage: "unsure what to think, displeasure" ),
       Emoji(symbol: "😍", name: "Heart Eyes",
       description: "A smiley face with hearts for eyes.",
       usage: "love of something; attractive"),
       Emoji(symbol: "🧑‍💻", name: "Developer",
       description: "A person working on a MacBook (probably using Xcodㄷ to write iOS apps in Swift).", usage: "apps, software, programming"),
       Emoji(symbol: "🐢", name: "Turtle", description:
       "A cute turtle.", usage: "something slow"),
       Emoji(symbol: "🐘", name: "Elephant", description:
       "A gray elephant.", usage: "good memory"),
       Emoji(symbol: "🍝", name: "Spaghetti",
       description: "A plate of spaghetti.", usage: "spaghetti"),
       Emoji(symbol: "🎲", name: "Die", description: "A single die.", usage: "taking a risk, chance; game"),
       Emoji(symbol: "⛺️", name: "Tent", description: "A small tent.", usage: "camping"),
       Emoji(symbol: "📚", name: "Stack of Books",
       description: "Three colored books stacked on each other.",
       usage: "homework, studying"),
       Emoji(symbol: "💔", name: "Broken Heart",
       description: "A red, broken heart.", usage: "extreme sadness"), Emoji(symbol: "💤", name: "Snore",
       description:
       "Three blue \'z\'s.", usage: "tired, sleepiness"),
       Emoji(symbol: "🏁", name: "Checkered Flag",
       description: "A black-and-white checkered flag.", usage:
       "completion")
    ]

    override func viewDidLoad() {
        super.viewDidLoad()

        // Uncomment the following line to preserve selection between presentations
        // self.clearsSelectionOnViewWillAppear = false

        // Uncomment the following line to display an Edit button in the navigation bar for this view controller.
        // self.navigationItem.rightBarButtonItem = self.editButtonItem
    }

    // MARK: - Table view data source

    override func numberOfSections(in tableView: UITableView) -> Int {
        print("numberOfSections")
        return 1
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        print("numberOfRowsInSection")
        return emojis.count //런타임에 사용자가 이모지 배열에 뭔가를 추가하면 바로 업데이트됨
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "EmojiCell", for: indexPath) //빈 셀을 넣어주고 하나하나 뺄때마다 새로운 이름표를 붙여서 나온다.
        let emoji = emojis[indexPath.row] //모델 객체를 가져옴
        
        var content = cell.defaultContentConfiguration()
        content.text = "\(emoji.symbol) - \(emoji.name)"
        content.secondaryText = emoji.description
        
        cell.contentConfiguration = content
        
        return cell
    }
}
```
