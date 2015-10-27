# 8 単一責任の原則(SRP: Signle Responsibility Principle)

6章のボウリングのスコアを計算するプログラムにおいて、Gameクラス(@see P96)は以下の2つの役割を果たしていた。
- 現在のフレームの状態記録
- スコアの計算

```
# Game.java(Fat ver)
public class Game
{
}
#...(略)
```

途中からGameクラスがフレームの状態を記録することにして、Scorerクラスがスコアの計算を担当することにした

```
# Game.java
public class Game
{
	public int score();
	public void add(int pins);
	public function int scoreForFrame(int theFrame);

	private Scorer itsScorer = new Scorer();
}
#Scorer.java
public class Scorer
{
	public void addThrow(int pins);
	public int scoreForFrame(int theFrame);
	public int scoreForFrame(int theFrame);

}
```

こうして2つの役割にわけるのはなぜか？役割が複数あると変更理由がたくさんでてきてしまい、私用要求によってクラスの役割がどのように変化したのか埋もれてしまう。(役割をわけておくと変更内容が浮き彫りになる)
また複数の役割を背負っていると、ある役割の変更によって予想もしない形で壊れてしまうことがある。(`もろい`設計)

* Rectangleの例(before)
下記のようなユースケースがある

* Client.1 (ComputationalGeometryApplication 図形の演算を行う)
* Client.2 (Graphical Application スクリーンに図形を描画)

```
<?php
class Rectangle
{
    public function draw(); //描画
    public function calcurateArea(); //面積の計算
}
```

描画の修正をすると、面積の計算部分も修正内容をうける(再ビルド・再テスト・再ロード(?))のは辛い。
そこでRectangle(draw)とGeometricRectangle(calculateArea)に分ける
こうすることで片方のアプリケーションの変更内容をもう片方がうけなくなってbetter。


### 8.1.1 役割とは何か?
単一責任の原則では、「役割(責任) = 変更理由」と定義されている。クラスを変クオするのに2つ以上の理由がある場合、クラスには2つ以上の役割があることになる。
しかし上記を見極めるのは難しい。

Modem([参考画像](http://ybb.softbank.jp/common/old/support/connect/adsl/modem/img/t3gp_pct1.gif))の例で考える。次はModemの役割を端的にあらわしている.

```Modem.java
interface Modem
{
    public void dial(String pnp);
    public void hangup();
    public void send(char c);
    public char recv();
}

```
これは2つの役割があるらしい。接続の管理(dial, hangup)とデータ通信(send, recv)。

### 8.1.2 結合している役割の分離

この2つが分離されるかどうかは、今後どのように変更されるか次第らしい。今後接続を管理する関数が影響を受けるような変更がされる場合、この設計は`硬さ`を露呈する。
sendとrecvを呼び出すクラスを不必要に再ビルド・再ロードしなければいけなくなるから。
Modem InterfaceはDataChannel(send, recv)とConnection(dial, hangup)に分けたほうがよさそうである。

この2つの役割が同時に変更されるようなケースではこれらを分離する必要はない。
`変更の理由が変更の理由たるのは、実際に変更の理由が生じた場合だけである`らしい。(仕様変更されていないクライアントが、コードの修正をしていないよね？みたいな意味と解釈)

### 8.1.3 永続性のあるシステムと単一責任の原則(SRP)
典型的な単一責任の原則違反の例を示す。
Employeeがビジネスロジックと永続化ロジック(DB)を包含している。

図8-4 EmployeeがPersisitenceSubsystemに依存している
```
<?php

class Employee
{
    public function storeSalary($employee)
    {
        PersisitenceSubsystem::save($employee->id, $employee->calculatePay());
    }

    public function calculatePay()
    {
        return $this->ochingin * workHours;
    }
}
```

Facadeパターンでなおす場合(間違ってたらいってくださいｗ)
```
<?php
class Employee
{
    public function calculatePay()
    {
         //something
    }
}

class OchinginRegisterFacade
{
    public function storeSalarary($employee)
    {
        PersistenceSubsystem::save($employee->calculatePay());
    }
}
```

* Proxyパターンでなおす場合(間違ってたらいってくださいｗ)
```
<?php

class Repository
{
    public function storeSalary($calculatePay)
    {
        PersistenceSubsystem::save($calculatePay);
    }
}

class Employee
{
    public function __construct($repository)
    {
        $this->_proxy = $repository;
    }
    public function calculatePay()
    {
        //something
    }
    public function saveSalary()
    {
        $this->_proxy->storeSalarary($this->calculatePay());
    }
}
```


## 8.2 結論
最もシンプルだが、正しく適用するのが最も難しい。
