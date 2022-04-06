## 학습 내용
- TDD, 유닛테스트 활동학습
- STEP 3 PR

고민했던 부분
: 스테퍼에는 title이 없어서 각각의 스테퍼를 식별할 수 있는 방법이 없는줄 알았다. 그래서 각각의 스테퍼마다 IBAction을 연결해줬지만,,, 비효율적이라고 생각했다.

```swift
    @IBAction func strawberryStepperValueChanged(_ sender: UIStepper) {
        FruitStore.shared.updateStock(fruit: .strawberry, quantity: Int(sender.value))
    }
    
    @IBAction func bananaStepperValueChanged(_ sender: UIStepper) {
        FruitStore.shared.updateStock(fruit: .banana, quantity: Int(sender.value))
    }
    
    @IBAction func pineappleStepperValueChanged(_ sender: UIStepper) {
        FruitStore.shared.updateStock(fruit: .pineapple, quantity: Int(sender.value))
    }
    
    @IBAction func kiwiStepperValueChanged(_ sender: UIStepper) {
        FruitStore.shared.updateStock(fruit: .kiwi, quantity: Int(sender.value))
    }
    
    @IBAction func mangoStepperValueChanged(_ sender: UIStepper) {
        FruitStore.shared.updateStock(fruit: .mango, quantity: Int(sender.value))
    }
```



**해결 방법**
Stepper restorationIdentifier를 지정해줘서 하나의 이벤트로 여러 스테퍼를 컨트롤 할 수 있다.
과일 별로 `Stepper` 이벤트를 따로 만들어주는게 효율적이지 못하다는 생각이 들어서 `Stepper restorationIdentifier`을 지정해 줬습니다. 
`restorationIdentifier`을 딸기, 바나나 등 과일 이름으로 지어줬습니다.
과일 이름으로 지어준 이름은 `Fruit` 타입의 원시값으로 `restorationIdentifier`을 지어주면 활용하기 좋을 것 같다는 생각에 과일 이름으로 지어줬습니다.
```swift
func initializeStepper() {
    strawberryStepper.restorationIdentifier = "딸기"
    bananaStepper.restorationIdentifier = "바나나"
    pineappleStepper.restorationIdentifier = "파인애플"
    kiwiStepper.restorationIdentifier = "키위"
    mangoStepper.restorationIdentifier = "망고"
        
    strawberryStepper.value = Double(FruitStore.shared.showStock(of: .strawberry)) ?? 0.0
    bananaStepper.value = Double(FruitStore.shared.showStock(of: .banana)) ?? 0.0
    pineappleStepper.value = Double(FruitStore.shared.showStock(of: .pineapple)) ?? 0.0
    kiwiStepper.value = Double(FruitStore.shared.showStock(of: .kiwi)) ?? 0.0
    mangoStepper.value = Double(FruitStore.shared.showStock(of: .mango)) ?? 0.0
}
```

```swift
@IBAction func stepperValueChanged(_ sender: UIStepper) {
    guard let stepperIdentifier = sender.restorationIdentifier,
          let fruit = Fruit(rawValue: stepperIdentifier) else {
              return
          }
    let quantity = Int(sender.value)
    FruitStore.shared.updateStock(fruit: fruit, quantity: quantity)
}
```
