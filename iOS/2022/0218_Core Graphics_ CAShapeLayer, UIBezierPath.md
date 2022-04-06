# Core Graphics
### 실험 1

컵, 음료수, 빨대 

1. 일단 커스텀 뷰를 만든 후에 뷰컨에서 addSubView 해주고
2. 커스텀 뷰의 frame을 정해준다.

`self.frame = CGRect(x: 50, y: 250, width: 300, height: 300)`

3. 각 그림마다 다른 레이어를 만들고 frame을 정해준다
4. CAShapeLayer() 로 레이어를 선언해주고
5. UIBezierPath() 로 패스를 만들어서
6. layer.path = cupPath.cgPath 로 레이어의 패스 프로퍼티에 패스를 할당해준다

```swift
class ViewController: UIViewController {
    let cupView = CupView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(cupView)
    }

    func addConstraints() {
        cupView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
        cupView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 30),
        cupView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -30),
        cupView.topAnchor.constraint(equalTo: view.topAnchor, constant: 100),
        cupView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -100)])
    }
}
```

```swift
class CupView: UIView {
    let strawLayer: CAShapeLayer = {
        let layer = CAShapeLayer()
        layer.lineCap = .round
        layer.frame = .init(x: 0, y: 0, width: 300, height: 300)
        
        let strawPath = UIBezierPath()
        layer.strokeColor = UIColor.black.cgColor
        layer.fillColor = UIColor.clear.cgColor
        layer.lineWidth = CGFloat(10)
        
        strawPath.move(to: CGPoint(x: 100, y: 230))
        strawPath.addLine(to: CGPoint(x: 190, y: 30))
        strawPath.addLine(to: CGPoint(x: 250, y: -10))
        
        layer.path = strawPath.cgPath
        return layer
        
    }()
    
    let copiedCupLayer: CAShapeLayer = {
        let layer = CAShapeLayer()
        layer.frame = .init(x: 0, y: 0, width: 300, height: 300)
        layer.strokeColor = UIColor.gray.cgColor
        layer.fillColor = UIColor.blue.cgColor
        layer.lineWidth = CGFloat(10)

        let cupPath = UIBezierPath()
        cupPath.lineCapStyle = .square
        cupPath.move(to: CGPoint(x: 50, y: 50))
        cupPath.addLine(to: CGPoint(x: 70, y: 250))
        cupPath.addQuadCurve(to: CGPoint(x: 200, y: 250), controlPoint: CGPoint(x: 140, y: 290))
        cupPath.addLine(to: CGPoint(x: 220, y: 50))
        
        layer.path = cupPath.cgPath
        return layer
        
    }()
    
    let drinkLayer: CAShapeLayer = {
        let layer = CAShapeLayer()
        layer.frame = .init(x: 0, y: 0, width: 300, height: 300)
        layer.fillColor = UIColor.green.cgColor
        
        let drinkPath = UIBezierPath(roundedRect: CGRect(x: 40, y: 100, width: 190, height: 200), cornerRadius: 0)
        drinkPath.lineWidth = CGFloat(10)
        
        layer.path = drinkPath.cgPath
        return layer
        
    }()
    
    let cupLayer: CAShapeLayer = {
        let layer = CAShapeLayer()
        layer.frame = .init(x: 0, y: 0, width: 300, height: 300)
        layer.strokeColor = UIColor.gray.cgColor
        layer.fillColor = UIColor.clear.cgColor
        layer.lineWidth = CGFloat(10)

        let path = UIBezierPath()
        path.lineCapStyle = .square
        path.move(to: CGPoint(x: 50, y: 50))
        path.addLine(to: CGPoint(x: 70, y: 250))

        path.addQuadCurve(to: CGPoint(x: 200, y: 250), controlPoint: CGPoint(x: 140, y: 290))
        path.addLine(to: CGPoint(x: 220, y: 50))
        
        layer.path = path.cgPath
        return layer
        
    }()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        makeLayer()
        self.frame = CGRect(x: 50, y: 250, width: 300, height: 300)
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        fatalError("init(coder:) has not been implemented")
    }

    func makeLayer() {
        layer.addSublayer(strawLayer)
        layer.addSublayer(copiedCupLayer)
        layer.addSublayer(drinkLayer)
        drinkLayer.mask = copiedCupLayer
        layer.addSublayer(cupLayer)
        
// 애니메이션은 이상하게 동작함 
        let animation = CABasicAnimation(keyPath: "bounds.size.height")
        animation.byValue = -100
        animation.duration = 3
        drinkLayer.add(animation, forKey: "aa")
        cupLayer.masksToBounds = true
    }

}
```
