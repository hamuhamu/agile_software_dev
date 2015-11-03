# 第9章 オープン・クローズドの原則（OCP）

「いかなるシステムも存在する限り変化する。長く使われ続けるようなシステムを開発するときは、そのことを認識していなければならない」

Bertrand Meyerが1988年にOCPを生み出した。

## 9.1 オープン・クローズドの原則（OCP:Open-Closed Principle）

*ソフトウェアの構成要素（クラス、モジュール、関数など）は拡張に対して開いて（オープン：Open）いて、修正に対して閉じて（クローズド：Closed）いなければならない。*

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


