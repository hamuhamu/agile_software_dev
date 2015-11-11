# アジャイルソフトウェア開発の奥義 chapter10

リスコフの置換原則
Liskov Substitution Principle

ch9: open-closed principle
OCPの要となる概念: 抽象とポリモーフィズム
抽象とポリモーフィズムを実現するメカニズムのひとつ: 継承

何でもかんでも継承していいのか？
適切な継承とは？
OCPに則った継承法はあるのか？

そのへんを考えるのが 10章の目的.
LSP.


# 10.1 LSP

LSP「派生型は，その基本型と置換可能でなければならない」
何を当たり前のことを...と思うかもしれbない

wikipediaさん曰く

> T 型のオブジェクト x に関して真となる属性を q(x) とする。このとき S が T の派生型であれば、S 型のオブジェクト y について q(y) が真となる。

擬似的に定義するとこんな感じ?

```
func f :: T -> X
o_t :: T
o_s :: S

f o_t == f o_s -- LSP成立
```

T型期待するとこにS型も使えるよねーみたいな話

- 前提詰めたい
- 「fにS型が適用できる」なのか「出力が等しい」なのかでだいぶ違うぞ
- 後者だったら結構違反することありそう
- 継承クラスでoverrideしたら大体後者満たさない気がするが..

# 10.2 違反する簡単な例

* コード詳細は本を見てください

```
public class Shape {
	String itsType = "";
	Shape(String type) {
		itsType = type;
	}
}

public class Square extends Shape {
	double itsSide;
	Point itsTopLeft;
    void Draw() {};

	Square() {
		super("square");
	}
}
public class Circle extends Shape {
	double itsRadious;
	Point itsCenter;
	void Draw() {}

	Circle() {
		super("circle");
	}
}

public class DrawingTool {

	void DrawShape(Shape shape) {
		if (shape.itsType.equals("square")) {
			((Square)shape).Draw();
		} else if (shape.itsType.equals("circle")) {
			((Circle)shape).Draw();
		}
	}
}
```

DrawShapeがOCP違反してる
shape増やすたびにif-elseの判定ロジックが増えてしまう
...この話LSP関係ない気がする

問題なのは各ブロック内で `Square`,`Circle` としてキャストしないとDraw関数が使えないということだろう
**SquareをShapeの代わりとして使えないよね！** という話．
ただ，この場合Square/Circleの基底クラスでDraw関数を宣言すればなんとかなっちゃう感ある．

LSPに違反する => OCPに違反してしまう という話のようだ
∵振る舞い違い検出のために条件判定しなくちゃいけないから

# 10.3 長方形と正方形: もっととらえにくい例

```
public class Rectangle {

	void SetWidth(double w) {itsWidth=w;}
	void SetHeight(double h) {itsHeight=h;}
	double GetHeight() {return itsHeight;}
	double GetWidth() {return itsWidth;}

	private Point itsTopLeft;
	private double itsWidth;
	private double itsHeight;
}

public class Square extends Rectangle {

    void SetWidth(double w) {
    	super.SetWidth(w);
    	super.SetHeight(w);
    }

    void SetHeight(double h) {
    	super.SetHeight(h);
    	super.SetWidth(h);
    }
}

```

正方形は，長方形が特定の条件を満たす時の呼び方である
いわば `正方形 ⊆ 長方形`

「正方形は長方形（の一種）である」ので，IS-A関係が成立
IS-Aは継承で表せる

B is one of A のとき， `B extends A` とかけるよねという話

IS-Aの概念はOOPの基本概念である
ただし，短絡的に継承されるとLSPに違反するおそれがある，というのが本節の主題

「Squareは height / width どちらか１つでいいじゃないか！メモリの無駄だ！」
という話は富豪的に解決するとして一旦置いておこう

以下の例を考える.

```
public testRectangle( Rectangle &r)
    r.SetWidth(32);
}
```

これにsquareが渡された時，Square::SetWidthが呼ばれないため，heightが不変．
squareの振る舞いとして正しくない．
(まぁ，rectangleを期待してるんだからheightが変わらないことは当然ではある.その意味でLSP自体は満たしている)

この矛盾を避けるためには，SetWidth/setHeightを overrideされた時そちらを優先するため，virtualで宣言すればok

## 10.3.1 問題の本質

しかし，上記は本質ではない
これはあくまで 「Rectangleとしての挙動」は正しいからだ．

まずいのは，Square では Rectangleと挙動が変わる点がある，ということ
すなわち，SquareをRectangleとみなして使った時に結果が異なってしまうことがある
以下を考えよう

```
public testRectangle( Rectangle &r)
    r.SetWidth(4);
    r.SetHeight(5);
    assertEquals(20.0, r.GetArea());
}
```

上記は Rectangleでは成立するが，Squareでは成立しない
Squareでは h/w 共に5なので area = 25 となる

ここで問題なのは，Rectangleクラスを使う側は，SetHeightによりwidthまで変更されるなど夢にも思わないからである
そりゃそうだ

この関数では 派生クラスSquareを基底クラスRectangleの代わりとして利用できないので，LSPに反していると言える

じゃー何が問題なのか？
クラス設計か？
それとも関数か？

これはそれぞれの立場において行ったことは正しい．
そもそも SquareをRectangleから派生したことに誤りがある．

## 10.3.2 正当性と本来的な性質は別物

「モデルの正当性は，立場によって異なる」ことを忘れてはならない

正当性を主張するのは「最終的に利用する側」である
ユーザが我々の作成物に対しどのような想定をするか？をつかむのが大事

とはいえ，我々はユーザではないので全てを把握するのは不可能
仮に可能だとしても，対応により余計な複雑性が混入する可能性がある

明らかにLSPに違反するものは対応するとしても，その他は「問題が顕在化したら対応」でよいのではなかろうか

## 10.3.3 IS-A関係に付随する「振る舞い」

先の例において，問題はなんだったのか？

一つの解: SqueareはRectangleでは**ない** (=IS-A関係ではない)
なぜなら，Rectangleと違う挙動をするから．明確である．

SquareとRectangleで， setHeight/Width の **振る舞いが変わった** ことが原因といえる．
（Rectangleのときにはhとwが同時に変化するなんて想定してなかったよ！）

ということで，IS-Aに加え，「合理的な仮定」のうえ，振る舞いの同等性を担保する必要があった．という話

ふと
正方形は長方形の，さらにいうと「４角形のとりうる一つの状態」と言えるかもしれんねー
同じパラメータをもつけど，同じオペレーションはできないよね．
あるパラメータがかぶった時に同じに見えるだけだよね: これがIS-Aか？
つまり同じ「モノ」ではないよね

## 10.3.4 契約による設計

合理的な仮定を明確にするテクニック: 契約による設計 Design By Contract. DBC

あるobjにおいて，メソッド前後の事前条件・事後条件を明示する

こんなかんじ

```
Rectangle::SetWidth(double w)
assert((itsWidth == w) && (itsHeight == old.itsHeight))
```

あるメソッドをoverrideする際，

- 事前条件は同等かユルく
- 事後条件は同等かキツく

すればよい．

## 10.3.5 ユニットテストに契約を明記する

契約内容はメソッド内validationではなく，ユニットテストに記述できる，
むしろ振る舞いを知る上では，そちらのほうがよかろう

[memo]
DBCの概念こんな感じ

パラメータの定義域が広くなるか，狭くなるかと捉えれば良さげ
f,gをメソッド，F,Gをメソッドによる各パラメータの写像ととらえて，

f(x) = g(x)
F(x) = y {x:X} {y:Y}
G(x) = y {x:X'} {y:Y'}

において

X' ⊇ X
Y' ⊆ Y

ならよかろう

結論: 副作用って怖い

# 10.4 実例

(力尽きそう

コードは本を見てください

## 10.4.1

集合扱うクラスライブラリ

Unbounded Set: link list
Bounded Set: array

があった
テンプレートクラスで書かれているので，様々なクラスの集合を作れる

いちいちbound/unboundを意識したくなかったので，setを抽象化した
共通インタフェースでアクセスできるようにした
具象クラスの各メソッド内で，bound/unboundのsetを扱うようにするのだろう
実コンテナ自体はhas-aで隠すということかな

上記設計では，どんなobjでも適切にbound/unboundを切り替えて使うことができた

## 10.4.2

扱えるsetとして persistentSetを加えようという話がでた

外部ライブラリ PersistentSetは，PersistentObjectを受け取り保存すすことができる

テンプレートクラスを使っていなかったので，PersistentObject以外を受け取ると正しい挙動ができない

（=入力可能な集合が狭まっている..!

問題なのは，Set自体は様々な型を許容するように書かれていること

実際に動かして，PersistentSetが 「保存できないオブジェクトをうけとる」までエラーが発生しない = デバッグしづらい

## 10.4.3

じゃーどうするねん という話

LSPに違反したままで，「約束事」で解決しようとした

要するに「persistent obj で渡せばいいんじゃろ？」という話なので

persistent setを使う際には，converterを挟め，という話？（自信ない

## 10.4.4

LSPに順ずるにはどうするか？

そもそも PersistentSet と Set は IS-A ではない

ので，継承すること自体が間違い

親子関係ではなく，兄弟関係と考えればよさそう（cf.p157の図

- member container
  - set
    - bounded
    - unbounded
  - persistent set

という感じ

# 10.5

直線 vs 線分

２点をつなぐ線，と考えると挙動が似ている
線分を直線の子クラスとしてよいか？

「ある点が与えられた時，それは自身（直線/線分）上にあるか？」というメソッドで挙動が変わってしまう

この場合，共通部分を切り出して基底クラスに持たせるのが吉
さっきのmember containerもこの形だろう

継承関係のままで，「線分のときは挙動変わるからよろしく」と妥協することもできる
抽象化するコストとのトレードオフ．
ただし，一度「派生クラスで基底クラスと異なる挙動する」を許容してしまうと，以降すべてのクラスで同じことを意識する必要が生まれるのでつらみ


