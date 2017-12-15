# Swift 開発メモ
個人的に初めて知ったことや、覚えておきたいと思ったことをまとめていきます。  
この資料を参考にする場合は自己責任でお願いします。

## 非同期処理
Swift の GCD (Grand Central Dispatch) を用いてマルチスレッド処理の実行を行います。
非同期処理は `async` 系を利用します。明示的に同期処理を実現する場合は `sync` 系を利用します。

---

`Timer` 以外にも実行時間を遅延させる方法がある。以下の例は5秒後にブロック内の処理をするものである。
```swift
DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    // 処理
}
```
参考：[[Swift 3] Swift 3時代のGCDの基本的な使い方](https://dev.classmethod.jp/smartphone/iphone/swift-3-how-to-use-gcd-api-1/)

## クロージャ
非同期処理が完了もしくは失敗した段階で値を返したい場合がある。このような場合はクロージャを用いて実現する。
```swift
class Model {
    func getUserData(success: ((Bool) -> Void)?, failure: ((Bool) -> Void)?) {
        success?(true)
        failure?(false)
    }
}

// 呼び出し側
let model = Model()
model.getUserData(success: { res in
    print(res ? "TRUE" : "FALSE")
}, failure: { res in
    print(res ? "TRUE" : "FALSE")
})
```

---

クロージャ内で `self` を用いる場合は `[weak self]` を利用しなければならない状況がある。以下のように実装を行う。ただし、 `self` を新しい変数として利用することもできる。
```swift
mmm.subscribe({ [weak self] in
    guard let `self` = self else { return }
    self.label.text = ""
})
```

## 分岐処理
`nil` が返ってきたときに返す値を指定することができる。`(model.data.hoge ?? "").isEmpty` といった使い方をすることができる。
```swift
var str: String?
str = nil
print(str ?? "NG") // NG
```

## StackView
コード側から StackView に追加した button を削除するには二段階の remove が必要になる。まず、 StackView から削除対象の button を `removeArrangedSubview` で削除する。次に、削除対象の button に対して `removeFromSuperview` を呼ぶ必要がある。また、 StackView から対象の button を探す場合は `flatMap` を使用する方法がある。
```swift
// buttonStackView に UIButton 型の button が配置されている
buttonStackView.arrangedSubviews.flatMap { $0 as? UIButton }.forEach {buttonView in
    self.buttonStackView.removeArrangedSubview(buttonView)
    buttonView.removeFromSuperview()
}
```

## Storyboard
Storyboard と対になる Class の名前が互いに同じ場合、以下の方法でインスタンス化することができる。
```swift
class Storyboard {
    static func instantiate<T: UIViewController>(_ type: T.Type) -> T {
        let storyboard = UIStoryboard(name: String(describing: type), bundle: Bundle.main)
        return storyboard.instantiateInitialViewController() as! T
    }
}

let vc = Storyboard.instantiate(NextPageViewController.self)
self.navigationController?.pushViewController(vc, animated: true)
```

## Animation
AutoLayout を用いて制約をつけているパーツをアニメーションさせる場合、 `self.view.layoutIfNeeded()` を呼ぶ必要がある。
```swift
@IBOutlet private weak var viewTop: NSLayoutConstraint?
self.view.layoutIfNeeded()
viewTop.constant = 10
UIView.animate(withDuration: 0.5, animations: {
    self.view.layoutIfNeeded()
}, completion: nil)
```
