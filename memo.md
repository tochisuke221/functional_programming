# 2章 純粋関数
## 純粋関数とは
純粋関数とは、同じ入力からは、必ず、同じ出力が得られる関数で、副作用のないものをいう。
以下3つの特徴がある
- 戻り値は1つだけ
- 引数のみに基づいて戻り値を計算する
- 既存の値を変更しない

## なぜ純粋関数は必要？
- 関数が嘘をつかない
  - 純粋関数は参照透過性を維持するため、驚きが少ない。
  - そのため**バグが少なくなる傾向にある**
  - また頭の中で状態遷移図を描く必要がなくなるので、**推論がしやすくなる**

- テストがしやすい。
  - **テストが簡潔**で理解しやすくなることから、テストをドキュメントの代用として扱える


# 第3章 イミュータブルな値について

## 可変性は危険
- ミュータブルな値を使うことはかなり危険
  - 例えばある関数fが引数aを受けとりaに何か操作を行った場合、実は同じインメモリを参照していて意図しない変更が加わってしまったなどの可能性がある
- 共有ミュータブル状態は、命令型プログラミングの構成要素である。（=関数型にすることでバグを減らせる）
    - プログラムのさまざまな箇所からアクセスできる状態・直接値が変更できる状態・1つの場所に格納されている状態が共有ミュータブル状態
- こうした可変性(可動部分)はプログラムを複雑化させる
- 可変性を回避することは関数プログラミングの要となる

## 可動部分に対処する
- オブジェクト思考のアプローチ
  - カプセル化を使って、変化するデータを保護することになる
  - カプセル化は、ミュータブル名状態を通常はオブジェクト中に隔離する手法のこと。
- 関数型プログラミングのアプローチ
  - 可動部分を限りなく少なくして、理想的には完全に取り除く
  - 例えば、コピーを使って可変性に対抗する
    - 渡されたリストをコピーして、コピーしたリストに対してのみ関数内で操作することでユーザを驚かせないようにする
  - イミュータブル値をベースに構築する
  
## 3.1.5のコーヒーブレイク
```: Main.scala
object Main extends App{
  def addreviate(fullName: String): String = {
    val initial = fullName.substring(0, 1)
    val separator = fullName.indexOf(' ')
    val lastName = fullName.substring(separator + 1)
    
    initial + ". " + lastName
  }

    println(addreviate("Justin taro"))
}
```

- 共有ミュータブル状態に対する純粋関数に対しては、常に渡された値と生成したい値の関係を全て洗い出すことから始める



# 第4章 値としての関数
## 関数のシグネチャは事実をありのままに伝えるべき
- シグネチャが関数の本体について、明らかにしない場合はその関数は嘘をついている。
  - 理想的には、パラメータリストがその説明をすべき
  - 純粋関数の特徴「関数はその引数にのみ基づいて、戻り値を計算する」を守るべき
## 関数を値として引数に渡す
- 関数をイミュータブルな値として引数に渡す
- ビジネス要件が大きく変わった時に、一部を変更するだけに止めることができる
  - 管理のしやすさ
  - テストのしやすさ
- コードの重複が減る(DRYの原則)
- 関数のシグネチャが明らかに、関数自体がドキュメントとなる

## 4.10のコーヒーブレイク

```coffee.java
static List<String> rankedWords(
  Function<String, Integer> wordScore, List<String> words {
    Compare<String> wordComparator = (w1, w2) -> Integer.compare(wordScore.apply(w2), wordScore.apply(w1));

    return word.stream().sorted(wordComparator).collect(Collectors.toList())
  }
)
```


以下の関数を引数で渡す

```
static int scoreWithBonusAndPenalty(String word){
  int base = score(word)
  int bonus = word.contains("c") ? 5 : 0
  int penalty = word.contains("s") ? 7 : 0

  return base + bonus + penalty
}

rankedWords(w -> scoreWithBonusAndPenalty(w), words)
```

↑
まだ改良の余地あり

```
static int bonus(String word){
  return word.contains("c") ? 5 : 0
}

static int penalty(String word){
  return word.contains("s") ? 7 : 0
}

rankedWords(w -> score(w) + bonus(w) - penalty(w), words)
```


## 4．17実習: Scalaで書き直してみる

```
// 文字列を昇順に並び替える
//入力List("scala", "rust", "ada")
//出力List("ada", "rust", "scala")
def len(s: String): Int = s.length
println(List("scala", "rust", "ada").sortBy(len))
    
//"s"が入っている数で昇順で並び替え
def lenOfS(s: String): Int = s.length - s.replaceAll("s", "").length
println(List("rust", "ada").sortBy(len))
    
// Int型の要素からなるリストを降順に並べる
// List(5, 1, 4, 3, 2)
def negative(i: Int): Int = -i
println(List(5, 1, 4, 3, 2).sortBy(negative))
    
//'s'の入った降順で並び替え
def nagativeLenOfS(s: String): Int = s.replaceAll("s", "").length - s.length
println(List("scala", "rust", "ada", "ssss").sortBy(nagativeLenOfS))


def rankedWords(wordScore: String => Int, words: List[String]) = {
  def negativeScore(word: String): Int =- wordScore(word)
  words.sortBy(negativeScore)
}
```

## 宣言型プログラミングを取り入れる
- 上記の実習で実装したrankedWordsは「何を行うか」ではなく「どのように行うか」に気をとられている
- 結果として数行でありながら、わかりづらく、読みづらい。

```
def rankedWords(wordScore: String => Int, words: List[String]) = {
  words.sortBy(wordScore).reverse
}
```

- 上記のようにすれば「何を行うか」が明確になる
- たいてい、宣言型のコードは命令型より簡潔でわかりやすい


## 小さな関数とそれたの役割
- ソフトウェアの設計は、管理しやすくすることを目的にしている。
- そのためには、コードの管理のしやすさ = 簡単に理解できるようにする必要がある
- **関数ごとに１つの小さなビジネス要件を実装することに焦点を合わせる**
- 関数をインラインにして渡す（無名関数）のテクニックが効果的だったりもすが、ワンライナーに限る


## 新しいパラメータを追加するだけでは不十分
- 新しい要件がきた際に、3つ目のパラメータを追加するなどはイマイチ
  - アプローチとしては悪くないが...
  - 簡単な修正ではすむ
  - しかし不十分
  - 私たちが実装するソリューションは他のユーザによって利用されることを踏まえると、引数を増やしても使いやすくはならない
- 繰り返しそのコードを利用する場合、重複を生み出す可能性がある

## 関数は関数を返すことができる
（省略）
P109 ~ P111 参照・まじで感動する

- 関数は関数を返すことができる
- 結局関数は値
- つまり、関数から関数を返すことができ、返された関数を任意の名前で保存する。
- 引数として使ったり、作成したり、他の関数から返すこともできる
- これらの特徴を使ってカリー化を行うことができる

## カリー化
- パラメータが複数の関数をパラメータが１つの関数に変換し、その関数からさらにパラメータが１つの関数を返す手法をカリー化という。
- （複数のパラmーエタを持つ関数を、関数から関数が返される一連のパラメータ１つの関数に変換することをカリー化と呼ぶ）

```
def f(a: A, b: B, c: C): D // 3つの引数を受け取る

def f(a: A): B => C => D //カリー化された関数。
def f(a: A)(b: B)(c: C): D
```

### カリー化実習
```
// 入力: List(5,1,2,4,0)
// ４より大きい値
def lagerThan(n: Int)(i: Int): Boolean => i > n
List(5,1,2,4,0).filter(largerThan(4))

// ５で割り切れる
def divisibleBy(n: Int)(i: Int): Boolean => i % n == 0
List(5,1,2,4,0).filter(divisibleBy(5))

// 文字sを3つ以上含んでいる
de containsS(n: Int)(s: String): Boolean => s.length - s.replaceAll("s", "") > n
List("rust", "ada").filter(containsS(2))

```

## 直積型を使ってイミュータブルデータをモデル化する
- 複数の情報を統合する手段として直積型がある
- 直積型は、イミュータブルであるため高階関数と非常に相性が良い

# 第5章 逐次プログラム

## パイプライン
- 現代のプログラミング言語において最も人気のあるパラーんがパイプライン処理
  - つまり逐次プログラム（ある値を受け取り、段階的に変換し最終的に値を返すロジックのこと）
- 組み立てと再利用可能なパーツから成り立つ。
  - したがって、大きな問題を小さなパーツに分解し、各パーツを個別に解いた後、パイプラインとして連結することで元の問題を解くことができる。

- 命令型のアプローチでは、以下のような問題がる(P138)
  - ミュータブルなリストを使い返す必要がある
  - 条件が増えるごとにループごとにインデントのレベルが深くなる
  - ループの本体で式ではなく文をつかう => テストしづらい

## 5.7 コーヒ-ブレイク

```
val recommendations = friends.flatMap(recommendedBooks)

val authors = recommendations.flatMap(_.authors)
```

## flatMap
- flatMapは非常に特別な関数
- 連結されたmapを入れ子のmapに変換することはできない
  - なぜなら、どのような関数を渡してもmapがListを返すということは保証されないため(ListのListなどがある)
- 一方、flatMapはそれが保証される(絶対Listが返ってくる)

```
List(1,2,3).flatMap(a => List(a * 2).flatMap(b => List(b, b + 19)))
```

## 5.12 実習 入れ子のflatMap

```
List(1).flatMap(x=>
  List(-2, 7).map(
    y => Point(x, y)
  )
)
```

## 入れ子のflatMapを改善
- 入れ子にすると読みにくくなる
- for内包表記を使う

```
for {
  book <- books
  author <- book.authors
  movie <- bookAdaptations(author)
} yield "${movie.title} is good"
```


