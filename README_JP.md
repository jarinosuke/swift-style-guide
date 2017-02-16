Swift コーディング規約

このドキュメントは、以下に挙げる目標を達成できる方法を促進するための試みとして作成されたものです。（大まかな優先度順となっています）

 1. より厳密で、プログラマが誤解する可能性が少ないこと
 1. 意図が明確であること
 1. 冗長さが排除されていること
 1. 美学についての議論が少ないこと

もし提案があれば、[ガイドライン](CONTRIBUTING.md)を読み、プルリクエストを送ってください。:zap:

また、日本語版のこのドキュメントでは Swift 言語仕様翻訳において[詳解Swift](http://www.amazon.co.jp/dp/4797380497)を参考としています。

----

#### 空白

 * スペースではなく、タブを使う
 * ファイル終端は改行する
 * コードをロジック毎に分割するために、空白行を惜しみなく使う
 * 行末に空白を残さない
   * 空白行でのインデント調整もしない

#### 可能な限り`var`宣言よりも`let`宣言を使う

可能な限り（どちらか迷った時にも）`var foo = …`より`let foo = …`を使いましょう。`var`は本当に使わないといけない時にだけ使うようにしましょう。（具体的には、あなたがその値が変わり得ることを*知っている*ときや、`weak`プロパティ修飾子を使っている時などです）

_理由:_ 二つのキーワードの意図と意味は明瞭ですが、*デフォルトでlet*を使うことは、より安全でより明確なコードになります。

`let`宣言は、その変数の値が変わらないことを保証すると同時に、それを*プログラマに明確に伝えます*。そのため、その後に続くコードにおいて、その変数の用途を推測しやすくなります。

コードを論理的に理解するのがより簡単になります。値が決して変わらないと考えているにもかかわらず`var`を使うと、本当に値が変わらないかどうかを手動でチェックしなければいけません。

この方法に従うと結果的に、`var`宣言が使われているのを発見した時は必ず、その値が変わり得ると推測したり、その理由を考えることができます。

#### Return と break を早期に行う

実行を継続するには基準を満たす必要があるような場合は、なるべく早期にコードブロックから抜け出すようにしましょう。

その際、以下のようにする代わりに:
```swift
if n.isNumber {
    // Use n here
} else {
    return
}
```

以下のように書きましょう:
```swift
guard n.isNumber else {
    return
}
// Use n here
```

If文でも同じことが可能ですが、`guard`を使うことが推奨されます。`guard`文では`return`や`break`、`continue`が無いとコンパイルエラーとなるため、コードブロックから抜け出すことが保証されるからです。

#### オプショナル型の開示指定は避ける

`FooType?`もしくは`FooType!`型の変数`foo`がある場合、そこにある`foo!`を得ようとして`foo`に対して開示指定を行うのは可能な限り避けないといけません。

代わりに、以下のようにしましょう:

```swift
if let foo = foo {
    // 開示された`foo`の値を使う
} else {
    // 必要であれば、ここでオプショナルがnilの場合の処理を行う
}
```

他の方法として、Swift のオプショナルチェーンを以下のように使った方が良い場合もあります。

```swift
// `foo`がnilでない場合は関数を呼ぶ、`foo`がnilの場合は関数呼び出し自体を無視する
foo?.callSomethingIfFooIsNotNil()
```

_理由:_ 明示的な `if let`によるオプショナル束縛構文は、結果的に安全なコードとなるからです。 開示指定はランタイムでのクラッシュを生み出してしまう可能性があります。

#### 暗黙的開示オプショナル型の使用を避ける

もし`foo`がnilになる場合がある場合、可能な限り`let foo: FooType!`ではなく`let foo: FooType?`を使うべきです。（一般に、`!`の代わりに`?`を使うことができると覚えておきましょう）

_理由:_ 明示的なオプショナル型は安全なコードを生み出すからです。暗黙的開示オプショナル型はランタイム時のクラッシュを生み出す可能性を持っています。

#### 読み取り専用のプロパティと添字付けのgetterは暗黙的にする

可能な限り、読み取り専用のプロパティと添字付けでは`get`キーワードを省きましょう。

以下のように書きましょう:

```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

… 以下は避けましょう:

```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

_理由:_ 一つ目の方が意図と意味が明確だからです。またコードも少なく済みます。

#### トップレベルの定義には常にアクセス制御を明記する

トップレベルの関数、型、変数には、常にアクセス制御指定子を明記するべきです。

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

しかし、トップレベルより下の定義については、必要に応じてアクセス制御を暗黙的にできます。

```swift
internal struct TheFez {
	var owner: Person = Joshaber()
}
```

_理由:_ トップレベルの定義が`internal`であることが適切である場合は滅多にないでしょう。そして、アクセス制御が明記されていることによって、深く考えた結果そのようにしたということが明らかになります。定義において、同じアクセス制御を繰り返すことはただ冗長なだけなので、デフォルト値が通常合理的です。

#### 型の指定では、常にコロンを識別子の後ろに繋げる

識別子に型を指定する時は、常に識別子のすぐ後ろにコロンを置き、空白を一つあけて型名を書きましょう。

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

_理由:_ 型指定子は、_識別子_に対してなんらかの意味づけをしようとしているので、識別子の近くに定義するべきです。

同じように、ディクショナリの型定義をするときも、常にキーの型のすぐ後にコロンをおいて、空白の後に値の型を指定します。

```swift
let capitals: [Country: City] = [sweden: stockholm]
```

#### 明示的な`self`参照は必要な時だけ

`self`の持つプロパティやメソッドへアクセスする時、デフォルトでは`self`への参照は省きましょう。

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

明示的に`self`を入れるのは言語によって必要と判断された場合だけにしましょう-たとえばクロージャ内、もしくはパラメータ名の衝突がある場合などです。

```swift
extension History {
	init(events: [Event]) {
		self.events = events
	}

	var whenVictorious: () -> () {
		return {
			self.rewrite()
		}
	}
}
```

_理由:_ これによってクロージャにおける`self`のキャプチャリングが目立つようになります。そしてその他の場所における冗長さが無くなります。

#### クラスより構造体を使うようにする

クラスでしか提供できない機能（同一性やデイニシャライザなど）を必要としない限り、クラスではなく構造体で実装しましょう。

継承は通常、クラスを使う主な良い理由には_なりません_。なぜなら、多様性はプロトコルによって実現可能であり、実装の再利用はコンポジションによって実現可能であるからです。

例として以下のクラス構造をあげます。

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

上記は以下のようにプロトコルと構造体で書き換えられます。

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * Float(vehicle.numberOfWheels)
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

_理由:_ 値型はシンプルで説明しやすく、`let`キーワードの挙動通りに動作してくれるからです。

#### デフォルトでクラスを`final`にする

クラスは初めは`final`にしておき、継承の正当な必要性が確認された場合にサブクラス化を許容する目的でのみ変更するべきです。さらにその場合、クラス内のそれぞれのクラスも、可能な限り同じルールに則り、同じように`final`を適用します。

_理由:_ 一般的に、継承よりもコンポジションを使うことが望ましいので、継承を選択しようとするときにその決定についてよく考える余地があることを示すことができるかもしれません。

#### 可能な限り型パラメータは省く

パラメータ化された型のメソッドは、引数の型がレシーバの型と同一の場合は型パラメータを省くことができます。たとえば:

```swift
struct Composite<T> {
	…
	func compose(other: Composite<T>) -> Composite<T> {
		return Composite<T>(self, other)
	}
}
```

上記は以下のように書くことができます:

```swift
struct Composite<T> {
	…
	func compose(other: Composite) -> Composite {
		return Composite(self, other)
	}
}
```

_理由:_ 冗長な型パラメータを省くことで意図が明確になり、その一方で、返り値の型が違う型パラメータを取る場合を明確に示すことができるからです。

#### オペレータ定義では空白を使う

オペレータを定義するときは、オペレータの前後に空白を使いましょう。  
以下のようではなく:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

以下のように書きましょう:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

_理由:_ オペレータは記号でできており、それらが型や値のパラメータのリストに使われる記号とくっついていると読みにくくなるからです。
空白を入れることでそれらをより明確に分けることができます。

#### 翻訳

* [中文版](https://github.com/Artwalk/swift-style-guide/blob/master/README_CN.md)
* [日本語版](README_JP.md)
* [한국어판](https://github.com/minsOne/swift-style-guide/blob/master/README_KR.md)
* [Versión en Español](https://github.com/antoniosejas/swift-style-guide/blob/spanish/README-ES.md)
* [Versão em Português do Brasil](https://github.com/fernandocastor/swift-style-guide/blob/master/README-PTBR.md)
* [فارسی](https://github.com/mohpor/swift-style-guide/blob/Persian/README-FA.md)
