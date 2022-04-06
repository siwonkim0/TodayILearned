# 1207 학습 내용
- asset catalog에서 데이터 가져오기
- 스크롤뷰 오토레이아웃
- 뷰와 모델 분리하기
- 테이블뷰 동적 데이터

만국박람회 STEP 2 진행


이번 한 주 배운 것 
- Table View
- Decoding 과 JSONDecoder
- 메타 타입
- Scroll View Auto Layout
- 뷰와 모델 분리

## asset 
asset catalog 에서 데이터 가져오는 방법

NSDataAsset 사용

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    guard let data = NSDataAsset(name: "exposition_universelle_1900") else {
        fatalError()
    }
    
}
```




## 스크롤 뷰 오토레이아웃
- Frame Layout 과 Content Layout의 차이

스크롤뷰는 내부에 컨텐츠가 없으면 자신의 크기를 결정할 수 없음

Content Layout Guide: 내부 컨텐츠의 크기 
**Frame Layout Guide**: 스크롤과 관계 없이 **화면을 보여주는** 범위 

Scroll View == **Frame Layout Guide**

- 스크롤뷰 오토레이아웃 적용 단계

1. 스크롤뷰 제약 설정 상위뷰에 0 0 0 0 —> 스크롤뷰의 크기, 위치는 지정이됨 
2. (스택)뷰를 스크롤뷰에 집어넣음
3. Content Layout Guide 에 Top, Bottom Space를 걸어준다 → 스크롤뷰가 얼마만큼의 높이를 가지는지 알기때문에 스크롤이 동작 (최대 얼만큼 올라가고 내려가는지 알고있는 상태)
4. Content Layout Guide 에 leading space를 걸어준다 → 어디서 시작하는지 알려줌 : 위치까지 지정됨
5. Frame Layout Guide 와 equal width를 가진다고 알려줌 → 넓이를 알려줌 (양옆으로 스크롤되지않게)
6. 그리고 나서 Content Layout Guide 에 trailing space를 걸어준다




## 토요 스터디 - 테이블 뷰 동적 데이터
## 실험 1

### 문제점

글씨가 겹쳐지는 현상 

1번 방법인 프로토타입 셀의 요소를 사용하면 커스텀 셀에서 구현해놓은거랑 겹쳐서 이렇게 되는듯

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "testIdentifier", for: indexPath) as? CustomTableViewCell else { return CustomTableViewCell() }
        
        //  1. 프로토타입 셀의 요소를 쓰는 것 -> 오류
//        cell.textLabel?.text = "안녕하세요 " + asset[indexPath.row] + "행 입니다."
//        cell.imageView?.image = UIImage(systemName: "circle")
        
        // 2. 수박 추천 방법) 커스텀 셀에서 만들었던 IBOutlet을 쓰는 것 -> 이거하기 위해 다운캐스팅 해주는 듯
        let camperAsset = jsonParser()
        cell.myLabel.text = camperAsset[indexPath.row].nicName
        cell.pointLabel.text = camperAsset[indexPath.row].yagomPoint.description
        
        // 3. 애플문서에 나온 방법) 커스텀 셀을 안만들고 하는 방법 -> 됨
//        var content = cell.defaultContentConfiguration()
//        content.text = "안녕하세요 " + asset[indexPath.row] + "행 입니다."
//        content.image = UIImage(systemName: "circle")
//
//        cell.contentConfiguration = content
//        cell.showsReorderControl = true
        
        return cell
    }
```

커스텀 셀 구현시에는 2번 방법인 커스텀 셀에서 연결해준 IBOutlet을 갖고오는게 베스트인듯

3번 방법도 애플 문서에나와서 괜찮은 방법인데 수박은 저거 쓰는사람 못봤다고해서,, 걍 2번쓰자.

---

## 실험 2

테이블 뷰 컨트롤러를 사용해줌

데이터를 열거형에 담아서 rawValue로 각 section의 이름을 설정해주었다.

CaseIterable 프로토콜을 채택해서 열거형의 모든 케이스를 담은 배열의 요소를 count해서 섹션의 개수를 결정

```swift
enum FoodType: String, CaseIterable {
    case American
    case Chinese
    case Korean
    case Japanese
}

struct Food {
    let name: String
    var price: Int
}
```

```swift
class FoodTableViewController: UITableViewController {
    var menu: [FoodType: [Food]] = [
        .American: [
            Food(name: "햄버거", price: 5000),
            Food(name: "피자", price: 18000),
            Food(name: "아메리카노", price: 41000)
        ],
        .Chinese: [
            Food(name: "탕수육", price: 18000)
        ],
        .Korean: [
            Food(name: "비빔밥", price: 8000),
            Food(name: "돼지갈비", price: 20000)
        ],
        .Japanese: [
            Food(name: "스시", price: 20000),
            Food(name: "스윙스(돈까스)", price: 5000)
        ]
    ]
    
    override func viewDidLoad() {
        super.viewDidLoad()

    }

**// MARK: - Table view data source**

    override func numberOfSections(in tableView: UITableView) -> Int {
        return FoodType.allCases.count
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        guard let key = menu[FoodType.allCases[section]] else { fatalError() }
        return key.count
    }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "food", for: indexPath)
        var content = cell.defaultContentConfiguration()
        
                guard let key = menu[FoodType.allCases[indexPath.section]] else { fatalError() }
        content.text = key[indexPath.row].name
        content.image = UIImage(systemName: "circle")

        cell.contentConfiguration = content
        cell.showsReorderControl = true
        return cell
    }
    
    override func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
            return FoodType.allCases[section].rawValue
    }
}
```
- Table View
- Decoding 과 JSONDecoder
- 메타 타입
- Scroll View Auto Layout
- 뷰와 모델 분리
