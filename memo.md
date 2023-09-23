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

