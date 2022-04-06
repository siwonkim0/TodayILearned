# 학습 내용
- Core Data Manager 만들기
- Core Data 활용하여 테이블 뷰 셀 업데이트하기 


# CoreDataManager
## 기본 세팅

1. container 만들기
2. context 만들기
3. load 하기 -> appdelegate에서 한번만 실행되도록 해준다

```swift
class MemoDataManager {
  static let shared = MemoDataManager(modelName: "CloudNotes")
  var memos = [Memo]()
  let persistentContainer: NSPersistentContainer
  var viewContext: NSManagedObjectContext {
      return persistentContainer.viewContext
  }
  
  private init(modelName: String) {
      persistentContainer = NSPersistentContainer(name: modelName)
  }
  
  func loadPersistentContainer() {
      persistentContainer.loadPersistentStores { description, error in
          guard error == nil else {
              fatalError(error!.localizedDescription)
          }
      }
  }
  
  func saveViewContext() {
      if viewContext.hasChanges {
          do {
              try viewContext.save()
          } catch {
              print("An error occured while saving: \(error.localizedDescription)")
          }
      }
  }
}
```

### 코어데이터 모델(entity)을 이니셜라이즈하려면 반드시 context와 함께 해야한다.

## CRUD
1. 새로운 데이터를 생성하고 싶을 때 

context로 entity를 이니셜라이즈 하고

바꾸고 싶은 프로퍼티의 값을 할당해준 후에

context를 save하면 된다. 

*context를 save하기 전까지는 코어데이터에 반영이 안된다는 사실*

```swift
var newMemo: Memo {
    let newMemo = Memo(context: MemoDataManager.shared.viewContext)
    newMemo.id = UUID()
    newMemo.title = ""
    newMemo.body = ""
    newMemo.lastModified = Date()
    saveViewContext()

    return newMemo
}
```

1. 기존 데이터를 수정하고 싶을 때

코어데이터에 있던 메모를 fetch해와서 (request.predicate 로 필터링 이용)

1처럼 바꾸고 싶은 프로퍼티를 할당한 후 

context save 시에 코어데이터가 수정이 되는 것 !! 

```swift
@discardableResult
func fetch(
    entityName: String = "Memo",
    predicate: NSPredicate? = nil,
    sortDescriptors: [NSSortDescriptor]? = [NSSortDescriptor(key: "lastModified", ascending: false)]
) -> [Memo]? {

    let request: NSFetchRequest<Memo> = Memo.fetchRequest()
    **request.predicate = predicate**
    request.returnsObjectsAsFaults = false
    request.sortDescriptors = sortDescriptors
    guard let newData = try? viewContext.fetch(request) else {
        return nil
    }
    return newData
}

func updateMemo(
    entityName: String = "Memo",
    id: UUID?,
    title: String?,
    body: String?,
    lastModified: Date = Date()
) {
    guard let id = id else {
        return
    }
    let predicate = NSPredicate(format: "id == %@", id.uuidString)
    **guard let memo = fetch(entityName: entityName, predicate: predicate)?.first else {
        return
    }**
    memo.title = title
    memo.body = body
    memo.lastModified = lastModified
    saveViewContext()
}
```

1. 기존 데이터를 삭제하고 싶을 때

2와 마찬가지로 request.predicate 로 필터링 해서 fetch해 가져온 entity를

delete() 메서드로 삭제하고 나서

context를 save 한다. 

```swift
func deleteMemo(id: UUID) {
    let request: NSFetchRequest<Memo> = Memo.fetchRequest()
    let predicate = NSPredicate(format: "id == %@", id.uuidString)
    request.predicate = predicate

    do {
        let memo = try viewContext.fetch(request)
        guard let memoToDelete = memo.first else {
            return
        }
        viewContext.delete(memoToDelete)
        saveViewContext()
    } catch {
        print(error)
    }
}
```

# 텍스트뷰가 수정이 될 때마다 코어데이터에 저장되는 문제
### 수정 전

textView의 text를 쪼개서 Memo 타입으로 만든 currentMemo를 

textView가 수정될때마다 불리는 textViewDidChange 메서드에서 

`Memo(context:)` 객체를 생성함과 동시에 코어데이터의 context에 업데이트하고 있었음 

마지막에 [viewContext.save](http://viewContext.save)() 이거 한번만 불리는 줄 알았더니 여러번 불리고 있었나보다

```swift
extension MemoDetailViewController: UITextViewDelegate {
        weak var delegate: MemoListViewControllerDelegate?
        private var currentMemo: Memo {
        let memoComponents = textView.text.split(
            separator: "\n",
            maxSplits: 1
        ).map(String.init)
        
        let memo = **Memo(context:** MemoDataManager.shared.viewContext)
        memo.title = memoComponents[safe: 0] ?? ""
        memo.body = memoComponents[safe: 1] ?? ""
        memo.lastModified = Date()
        
        return memo
    }

    func textViewDidChange(_ textView: UITextView) {
        delegate?.memoListViewController(updateTableViewCellWith: currentMemo)
    }
}

protocol MemoListViewControllerDelegate: AnyObject {
    func memoListViewController(updateTableViewCellWith memo: Memo)
}

extension MemoListViewController: MemoListViewControllerDelegate {
        func memoListViewController(updateTableViewCellWith memo: Memo) {
            guard let indexPath = tableView.indexPathForSelectedRow else {
                return
            }
            
            let request: NSFetchRequest<Memo> = NSFetchRequest(entityName: "Memo")
                //context의 수가 글자 입력할때마다 늘어남 
            print(try? MemoDataManager.shared.viewContext.count(for: request))
                
            **MemoDataManager.shared.memos[indexPath.row] = memo**
                //메모의 수는 그대로 
            print(MemoDataManager.shared.memos.count)
            tableView.reloadRows(at: [indexPath], with: .none)
            tableView.selectRow(at: indexPath, animated: true, scrollPosition: .none)
            
            do {
                try **MemoDataManager.shared.viewContext.save()**
            } catch {
                print("에러")
            }
        }
}
```

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220217/620d2f669276e54c1b0739ad.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220217/620d2f669276e54c1b0739ad.png)

### 문제점/이유

텍스트뷰가 수정이 될 때마다 코어데이터에 저장이 되어서

당장 수정할때는 정상적으로 하나만 나오지만, context의 수가 글자 수정한 수만큼 늘어나서 

다음 fetch시 여러 셀들이 한꺼번에 나오게 된다. 

(지금 최신 메모가 아래에 가있는 것은 fetch해올때 sort를 아직 안해서 그렇다)

![https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220217/620d2f639276e54c1b0739ab.png](https://s3.ap-northeast-2.amazonaws.com/media.yagom-academy.kr/resources/usr/6152fa94ccd9ef11a51aee7f/20220217/620d2f639276e54c1b0739ab.png)

### 해결 방법

코어데이터의 `Memo(context:)` 객체를 이용하지 않고 그냥 title, body를 전달하여 

해당 indexPath로 Memo 를 가져와서 수정해준 후 

마지막에 viewContext.save() 로 context를 저장해준다.

```swift
extension MemoDetailViewController: UITextViewDelegate {
    func textViewDidChange(_ textView: UITextView) {
        let memoComponents = textView.text.split(
            separator: "\n",
            maxSplits: 1
        ).map(String.init)
        
        let title = memoComponents[safe: 0] ?? ""
        let body = memoComponents[safe: 1] ?? ""
        let lastModified = Date()
        
        delegate?.memoListViewController(updateTableViewCellWith: title, body: body, lastModified: lastModified)
    }
}

extension MemoListViewController: MemoListViewControllerDelegate {
        func memoListViewController(updateTableViewCellWith title: String, body: String, lastModified: Date) {
        guard let indexPath = tableView.indexPathForSelectedRow else {
            return
        }
        
        let memo = MemoDataManager.shared.memos[indexPath.row]
        memo.title = title
        memo.body = body
        memo.lastModified = lastModified
        
        tableView.reloadRows(at: [indexPath], with: .none)
        tableView.selectRow(at: indexPath, animated: true, scrollPosition: .none)
        
        do {
            try MemoDataManager.shared.viewContext.save()
        } catch {
            print("에러")
        }
    }
}
```
