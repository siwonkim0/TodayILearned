# 학습 내용 
- 셀 선택 관련 메서드
- 선택된 셀의 indexPath

# 테이블 뷰 셀 선택되도록 하는 방법 : selectRow

## 메모 추가하기 버튼 선택시 추가된 셀이 선택되도록하기

```swift
@objc private func addMemo() {
    let newMemo = Memo(title: "", body: "", lastModified: Date())
    MemoDataManager.shared.memos.insert(newMemo, at: 0)
    tableView.reloadData()
    tableView.selectRow(at: IndexPath(row: 0, section: 0), animated: true, scrollPosition: .top)
    delegate?.memoDetailViewController(showTextViewWith: newMemo)
}
```

## 셀 삭제 기능 + 삭제시 바로 밑 셀 선택 기능

- trailingSwipeActionsConfigurationForRowAt 메서드: 셀을 스와이프하면 삭제 버튼이 보이도록 생성
- 테이블 뷰 데이터소스에서 먼저 데이터를 삭제한 후
- tableView.deleteRows 를 해야함 (반대로 하면 에러 발생)
- tableView.allowsSelectionDuringEditing = true 로 스와이프 하는 동안 삭제 버튼이 선택되어있도록
- 그리고 tableView.selectRow로 선택을 함

→ 그러나 이렇게만 하면 삭제가 완료되었을때 아래 셀이 선택이 잠깐 됐다가 선택이 풀림 

따라서 수정이 끝나면 불리는 **didEndEditingRowAt 메서드에서 다시한번** tableView.selectRow를 해줘야함 

```swift
func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
    let deleteAction = UIContextualAction(style: .destructive, title: "Delete") { _, _, _ in
        MemoDataManager.shared.memos.remove(at: indexPath.row)
        tableView.deleteRows(at: [indexPath], with: .fade)
        let memo = MemoDataManager.shared.memos[indexPath.row]
        self.delegate?.memoDetailViewController(showTextViewWith: memo)
        
        tableView.allowsSelectionDuringEditing = true
        tableView.selectRow(at: indexPath, animated: false, scrollPosition: .top)
    }
    deleteAction.image = UIImage(systemName: "trash")
    return UISwipeActionsConfiguration(actions: [deleteAction])
}

func tableView(_ tableView: UITableView, **didEndEditingRowAt** indexPath: IndexPath?) {
    tableView.selectRow(at: indexPath, animated: false, scrollPosition: .top)
}
```

![didEndEditingRowAt에서 selectRow 안했을때](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1ad367cd-4c50-4c4a-972a-9e6f632eb936/Simulator_Screen_Recording_-_iPad_Pro_(9.7-inch)_-_2022-02-15_at_20.19.44.gif)

didEndEditingRowAt에서 selectRow 안했을때

![didEndEditingRowAt에서 selectRow 했을때](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/238d0486-ba6c-4403-a8eb-09435980853e/Simulator_Screen_Recording_-_iPad_Pro_(9.7-inch)_-_2022-02-15_at_20.21.37.gif)

didEndEditingRowAt에서 selectRow 했을때

왼) 선택 풀림 

오른) 선택 유지 

---

# 선택된 셀의 indexPath 반환 : tableView.indexPathForSelectedRow

기존 코드에서는 list 뷰컨에서 detail 뷰컨으로 갈 때 선택된 셀의 index 정보를 저장하기 위해

특정 셀을 선택시 detail뷰컨에 셀의 index까지 같이 넘겨줬었다.

그러나 현재 선택되어있는 셀의 index를 반환하는 프로퍼티인

tableView.indexPathForSelectedRow를 알게되었다.

- memoListViewController(updateTableViewCellWith memo: Memo) 메서드

따라서 list 뷰컨에서 셀을 선택해서 detail 뷰컨이 나타나 있는 상태에서

detail 뷰컨의 메모 내용을 수정했을 때 list뷰컨의 현재 선택되어있는 셀의 index를 가져와서

전달받은 Memo 타입으로 데이터를 업데이트하고

업데이트된 특정 셀만 reload한다. 

여기에서도 삭제와 마찬가지로 마지막에 selectRow를 한번 더 해줘야 선택이 유지가 된다. 

이 메서드는 detail 뷰컨의 textViewDidChange에 있어서 textView가 수정될때마다 불린다.

```swift
extension MemoDetailViewController: UITextViewDelegate {
    func textViewDidChange(_ textView: UITextView) {
        delegate?.memoListViewController(updateTableViewCellWith: currentMemo)
        private var currentMemo: Memo {
            let memoComponents = textView.text.split(
                separator: "\n",
                maxSplits: 1
            ).map(String.init)
            
            let title = memoComponents[safe: 0] ?? ""
            let body = memoComponents[safe: 1] ?? ""
            let date = Date()
            
            return Memo(title: title, body: body, lastModified: date)
        }
    }
}
```

```swift
extension MemoListViewController: MemoListViewControllerDelegate {
    func memoListViewController(updateTableViewCellWith memo: Memo) {
        guard let indexPath = tableView.indexPathForSelectedRow else {
            return
        }
        MemoDataManager.shared.memos[indexPath.row] = memo
        tableView.reloadRows(at: [indexPath], with: .none)
        tableView.selectRow(at: indexPath, animated: true, scrollPosition: .none)
    }
}
```

# 배열 안전하게 조회

indicies 가 컬렉션타입이 가지고 있는 인덱스의 배열인데

만약 그 안에 지금 조회하는 인덱스가 들어있다면 그대로 반환하고, 

아니라면 nil을 반환하는 서브스크립트 

→ index out of range 에러를 방지함으로써 정말 있는 배열의 인덱스만 조회할 수 있다. 

```swift
extension Collection {
    subscript (safe index: Index) -> Element? {
        return indices.contains(index) ? self[index] : nil
    }
}
```
