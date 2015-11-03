# 第9章 オープン・クローズドの原則（OCP）

「いかなるシステムも存在する限り変化する。長く使われ続けるようなシステムを開発するときは、そのことを認識していなければならない」

Bertrand Meyerが1988年にOCPを生み出した。

## 9.1 オープン・クローズドの原則（OCP:Open-Closed Principle）

**ソフトウェアの構成要素（クラス、モジュール、関数など）は拡張に対して開いて（オープン：Open）いて、修正に対して閉じて（クローズド：Closed）いなければならない。**

OCPはあとで似たような変更がもっと出た時に、コードを修正しないで済ませるにはどうすればよいかを教えてくれる。

OCP使ってリファクタリングすると、新しいコードを追加するといった方法で対処出来る。既存コードに修正を加えない。

## 9.2 オープン・クローズドの原則（OCP）の概要

1. 拡張に対して開かれている（オープン：Open）
モジュールの振る舞いを拡張できるという意味。
1. 修正に対して閉じている
モジュールの振る舞いを拡張しても、そのモジュールのソースコードにはまったく影響を受けない。

矛盾していると思うだろ？

## 9.3 鍵は「抽象」にあり

宣言が固定されていても、特定の実装に結合していないメソッドを「抽象」を使って表現出来る。特定の実装に結合していないメソッドはその派生クラスで実装される。

モジュール設計において、修正に対してコードを閉じることが出来る。コードを修正しなくても、派生クラスを新たに追加すればよい。

図9-1はOCPでない例。具体的に実装されてしまい、別のServerオブジェクトを利用する際にClientクラスを変更する必要がある。

図9-2はOCPの例。ClientInterfaceクラスは抽象クラスで、いくつかの抽象メソッドを持っている。ClientクラスはClientInterfaceクラスから派生しているので、新しいServerクラスを利用したくなったら、それに対応したClientInterfaceの派生クラスを追加すれば良い。Clientクラスそのものの変更は不必要。

ClientInterfaceは抽象メソッドを持っているだけで、実装はその派生クラスに任せている。

AbstractServer（抽象サーバー）とせずにClientInterface（クライアントインタフェース）と名づけている理由。「抽象クラスはそれを実際に実装するクラスとの関係よりも、それを利用するクラスとの関係の方がずっと密接」だから。

図9-3は別の方法。Policyクラスは「方針」を具体的に実装したPublic関数を持っている。Clientとほぼ同じだが、方針関数が利用する抽象インターフェースはPolicyクラス自身の一部という点で異なる。こういった関数の実態はPolicyの派生クラスで自由に実装出来るので、Policyで利用する振る舞いを拡張したり修正したければ、Policyの派生クラスを新たに作り、その派生クラスでそういった拡張や周背に対応した実装をすればよい。

最初のやつはStrategyパターン、2番目がTemplate Methodパターン。OCPの典型的なパターン。汎用的な機能と具体的な実装を明確に分ける。

## 9.4 図形描画アプリケーションの例

悪評高い「Shape（図形）」。ポリモーフィズム（多様性）のメカニズムに利用するところをOCPの説明で使う。

標準的なGUI操作で円と資格を描くアプリケーションを作るとする。

### 9.4.1 オープン・クローズドの原則（OCP）に従わない例

C言語を使った手続き型の手法はOCPに違反する。その例。

```c
--shape.h------------------------------------
enum ShapeType {circle, square};

struct Shape
{
   ShapeType itsType;
};

--circle.h------------------------------------
struct Circle
{
   ShapeType itsType;
   double itsRadisu;
   Point itsCenter;
}

void DrawCircle(struct Circle*);

--square.h------------------------------------
struce Square
{
   ShapeType itsType;
   double itsSide;
   Point itsTopLeft;
};

void DrawSquare(struce Square*);

--drawAllShapes.cc------------------------------------
typedef struct Shape *ShapePointer;

void DrawAllShapes(ShapePointer list[], int n)
{
   int i;
   for (i=0, i<n; i++)
   {
      struct Shape* s = list[i];
      switch (s->itsType)
      {
      case square:
         DrawSquare((struct Square*)s);
      break;
      
      case circle:
         DrawCircle((struct Circle*)s);
        break;
      }
   }
}
```

新しい種類の図形に対して閉じていない。三角形を足した時に、このコードを修正する必要がある。

実用的にはDrawAllShapes関数にはswitch文を使うだろう。switch文のcase項目が同じでも処理内容が違ってくる。図形をドラッグしたり、変形させたりする関数がありうる。新しい図形を1つ追加すると、そういったswitch文の部分を探しだして、そこに関数を追加しないといけない。

実際はもっと複雑なものになるだろう。

新しい図形を追加すると、ShapeTypeに追加する必要がある。他のすべての図形はShapeTypeに依存しているので、全ての図形を再コンパイルしないといけない（列挙型を保持する変数のサイズが変わる可能性がある）。

ソースコードを変更するだけでなく、構造体Shapeを使う全てのモジュールのバイナリファイルも変更する必要がある。新しい図形を気軽に追加出来ない。

**悪い設計**

9-1の設計は「硬い」「もろい」「移植性がない」。

### 9.4.2 オープン・クローズドの原則（OCP）に従った例

9.2はOCPに準じている。

```java
public abstract class Shape {
   public abstract void Draw();
}

public abstract class Square extends Shape {
   double itsSide;
   Point itsTopLeft;
}

public abstract class Circle extends Shape {
   double itsRadious;
   Point itsCenter;
}

public class DrawingTool {
   public void DrawAllShapes(List<Shape> shapeList){
      for (Shape shape : shapeList) {
         shape.Draw();
      }
   }
}
```

新しい図形を描画出来るようにしたい時はShapeクラスの派生クラスを新たに追加すれば良い。他のモジュールにも **まったく影響を与えない**。

「もろさ」がない。

「硬さ」もない。

「移植性」がある。

**既存のコードを変更するのではなく、新しいコードを追加すること** で変更に対応出来る。

### 9.4.3 落とし穴

先の例では話がうますぎる。 **四角（Square）を描く前にすべての円（Circle）を描画しなければならない** という仕様変更に閉じていない。

### 9.4.4 「先を見越した構造」と「自然な構造」

仕様変更が起きることを先に見越していれば、それに対応した「抽象」が出来たはず。でも出来ていない。つまり不適切だった。図形の形よりも描画の順番の方が重要なシステムにおいては、Shapeを使ったモデルは自然 **ではない** ということなのだ。

**すべてのケースに適用出来る自然なモデルなど存在しないのだ！**

戦略的に閉じるしかない。今後どういう種類の変更が起きるのかを想定して、「抽象」する。

ある程度の経験に基づくもの。ユーザや業界の理解。

正しく推測できれば勝ち、間違った推測をすれば負け。ほとんど負ける。

OCPに準ずることが高く付きすぎる場合もある。設計には時間がかかる。OCPの適用は起こる可能性の高い変更に限定すべき。

結論頑張るしかない。あとは **実際に変更が起きるのを待つしかない！**

### 9.4.5 「仕掛け」を仕込む

前世紀には「仕掛け」を仕込んでおけばおｋだった。

使ってもいない仕掛けのせいでむしろ複雑になっていった。実際に望ましい抽象が必要になった時点でシステムに仕込むべき。

**私を一度欺く者には恥あれ・・・**

「私を一度欺く者には恥あれ。私を二度欺くことあらば私に恥あれ」。ソフトウェアに **不必要な複雑さ** を導入しないために、 **一度** は騙されてみる。最初は変更が起きないことを前提にコードを書いておく。その結果変更が起きたら、今後それと **同じような種類の** 変更があった場合に身を守れるように抽象を導入する。

**意識的に変化を促す**

できるだけ早期にかつ頻繁に変更が起こって欲しい。

意識的に変化を促すのが懸命。2章の方法を利用できる。

* テストファースト
* 短いサイクルで開発
* 大きな基幹ソフトの導入を考える前に、今必要な機能を優先して開発し、実装した機能を頻繁に出資者に提示する
* 最も重要な機能から優先的に開発する
* 早期かつ頻繁にソフトウェアをリリースする。

### 9.4.6 明示的に閉じるために抽象を使う

先ほどの例の仕様変更に耐えたい。

DrawAllShapesメソッドを閉じたい。閉じるという行為は「抽象」で出来る。「順番の抽象」ができればよい。

Precedes（優先する）という名前の抽象メソッドをShapeに定義して順序付けを実現。

```java
public abstract class Shape {
   abstract void Draw();
   abstract boolean Preceeds(Shape shape);
}
```

```java
public class ShapeComparator implements Comparator<Shape> {
   public int compare(Shape shape1, Shape shape2) {
      if (shape1.Preceeds(shape2)) {
         return -1;
      } else {
         return 1;
      }
   }
}

public class DrawingTool {
   void DrawAllShapes(List<Shape> shapeList){
      List<Shape> orderedList = DrawingOrderSort(shapeList);
      for (Shape shape : orderedList) {
         shape.Draw();
      }
   }
   
   List<Shape> DrawingOrderSort(List<Shape> shapeList) {
      List<Shape> orderedList = new ArrayList<Shape>(shapeList);
      Collections.sort(orderedList, new ShapeComparator());
      return orderedList;
   }
}
```

これだと不完全。個々のShapeオブジェクトはPrecedesメソッドをオーバーライドして、順序付けを指定しなければならない。9-5をみる。

```java
public abstract class Circle extends Shape {
   double itsRadious;
   Point itsCenter;
   boolean Preceeds(Shape shape) {
      if (shape instanceof Square) {
         return true;
      } else {
         return false;
      }
   }
}
```

PredeedsメソッドはOCPに準拠していない。新しいShapeの派生クラスを作る行為に対してこのメソッドを閉じる方法はない。新しいShapeの派生クラスが作られるたびに、Precedes()を変更しないといけない。

新しいShapeの派生クラスが作られなければ問題にはならない。頻繁に起こる際には問題になってくる。

### 9.4.7 「テーブル駆動型」のアプローチを使って閉じる方法

テーブル駆動型のアプローチを利用して実現出来る。

```java
public abstract class Shape {
   abstract void Draw();
   
   abstract String getType();
   
   private String typeOrderTable[] = { "square", "circle" };
   
   boolean Preceeds(Shape shape) {
      String thisType = this.getType();
      String argType = shape.getType();
      int thisOrd = -1;
      int argOrd = -1;
      int ord = 0;
      
      for (String tableEntry : typeOrderTable) {
         if (tableEntry.equals(thisType)) {
            thisOrd = ord;
         }
         if (tableEntry.equals(argType)) {
            argOrd = ord;
         }
         if ((0 <= argOrd) && (0 <= thisOrd)) {
            break;
         }
         return thisOrd < argOrd;
      }
   }
}
```

一般的な順序付けの問題に対してDrawAllShapesを閉じることが出来る。新しくShapeの派生型が出来たり、順序付けを変更する行為に対してもShapeの派生型を閉じることが出来る。

この方法の場合、順序付けに対して閉じていないのはテーブル自身だけ。

## 9.5 結論

OCPはオブジェクト指向設計の核心。この原則に従えば、利益（柔軟性、再利用性、保守性）を享受出来る。開発者が担当部分で最も頻繁に変更されるプログラム部分にだけ的を絞って抽象を適用するよう努めるべき。 **早まった「抽象」をしないことも、「抽象」を使うのと同等に重要なことなのだ。**
