# Swift 開発メモ
個人的に初めて知ったことや、覚えておきたいと思ったことをまとめていきます。  
この資料を参考にする場合は自己責任でお願いします。

- [可読性](#可読性)
- [アクセス修飾子](#アクセス修飾子)
- [定数と変数](#定数と変数)
- [非同期処理](#非同期処理)
- [クロージャ](#クロージャ)
- [分岐処理](#分岐処理)
- [メソッド](#メソッド)
- [StackView](#stackview)
- [Storyboard](#storyboard)
- [Animation](#animation)
- [UIPickerView](#uipickerview)
- [UIDatePicker](#uidatepicker)

## 可読性
`isEmpty` を使用して、なおかつ `!` で真偽値を反転すると、最終的に **文字列が格納されているか** を判定したいという意味がわかりにくくなる。そこで、以下の改善後のように明示的に **どうであってほしいか** を判定式に盛り込むことで可読性を上げることができる。また、複数の判定が関係する場合は同じ形式に揃えることも検討すべきである。
```swift
// 改善前
if !(data.apple.isEmpty) && !(data.swift.isEmpty) && data.user == .akkey {
    print("done")
}
// 改善後
if data.apple.isEmpty == false && data.swift.isEmpty == false && data.isAkkey == true {
    print("done")
}
```

## アクセス修飾子
アクセス修飾子 | 挙動
----- | -----
`open` | モジュール外からもアクセスできる
`public` | モジュール外からもアクセスできる、サブクラス化されない、 override できない
`internal` | モジュール内に限りアクセスできる
`fileprivate` | ファイル内に限りアクセスできる
`private` | 宣言した中に限りアクセスできる、 extension 内からもアクセス可能

\* モジュール : import を使って取り組むプログラム群  
参考：[\[Swift 3.0\] アクセス修飾子にopenとfileprivateが追加された話](https://dev.classmethod.jp/smartphone/iphone/swift3_scoped_access_level/)

---

set のみ private にして、 get は public にしたい場合がある。これを実現する方法は複数あるが、 Swift であれば比較的簡単に実現できるので、比較して以下にします。 `private(set)` とすることで setter のみを private にすることができる。逆に、 get を private にして set を public にすることは一般的にすべきでない。
```swift
// 一般的な方法
private var _data: String = ""
public var data: String {
    return _data
}
_data = "AKKEY"

// 修正後
public private(set) var data: String = ""
data = "AKKEY"
```

## 定数と変数
変数定義の方法を複数紹介する。
```swift
// ポインタアドレスの固定なので可能
// ただし、 Array や Dictionary はアドレス変更が必要なので不可
let data = User()
data.value = 8

// setter と getter
var value: Int {
    get { return 0 }
    set { _ = newValue }
}

// getter
let value: Int {
    return 0
}

// didSet
var value: Int = 0 {
    didSet {
        self.update()
    }
}

// 使うときに初期化される
lazy var data: User = {
    return User()
}()
```

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

## メソッド
引数の値に初期値を設定することができる。初期値を設定している引数に関しては、値を渡さずにメソッドを呼ぶことができる。
```swift
func createData(name: String, number: Int = 0, plus: String = "nil") -> String {
    return "\(name) | \(number) | \(plus)"
}

createData(name: "AKIO") // "AKIO | 0 | nil"
createData(name: "AKIO", number: 8) // "AKIO | 8 | nil"
createData(name: "AKIO", number: 8, plus: "OK") // "AKIO | 8 | OK"
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

## UIPickerView
UITextField に UIPickerView での変更をリアルタイムで反映させる表示方法を簡単に実現することができる。**UITextField** の inputView に UIPickerView を設定することで実現可能である。
```swift
private var textField: UITextField?
private var picker: UIPickerView?

override func viewDidLoad() {
    picker = UIPickerView()
    picker?.dataSource = self
    picker?.delegate = self

    textField = UITextField()
    // UITextField の inputView に UIPickerView を設定
    // inputAccessoryView で KeyboardAccessary も設定可能
    textField?.inputView = picker

    // 初期位置の設定
    picker.selectRow(data[1], inComponent: 0, animated: true)
}

func numberOfComponents(in pickerView: UIPickerView) -> Int {
    return 1
}

func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
    return data.count
}

func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
    return data[row].text
}

func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
    guard let picker = self.picker else { return }
    // リアルタイム更新
    textField.text = self.pickerView(picker, titleForRow: row, forComponent: 0)
}
```

## UIDatePicker
UITextField に UIDatePicker での変更をリアルタイムで反映させる表示方法を簡単に実現することができる。**UITextField** の inputView に UIDatePicker を設定することで実現可能である。
``` swift
private var textField: UITextField?
private var picker: UIDatePicker?

override func viewDidLoad() {
    picker = UIDatePicker()
    // 各種設定
    picker?.datePickerMode = .date
    picker?.calendar = Calendar(identifier: .gregorian)
    // picker の値が変動したときに呼ばれる関数の設定
    picker?.addTarget(self, action: #selector(change), for: .valueChanged)
    // UITextField の inputView に UIDatePicker を設定
    // inputAccessoryView で KeyboardAccessary も設定可能
    textField?.inputView = picker

    // 初期位置の設定
    picker.setDate(day, animated: true)
}

@objc private func change() {
  textField.text = picker.date
}
```
