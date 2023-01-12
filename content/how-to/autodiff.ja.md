---
title: "勾配(微分)の計算方法"
date: 2019-10-29T20:07:07+01:00
draft: false
---

## ゴール
以下の簡単な式を検討します:

$$ f(x,y,z) = ( x + y ) \times z \tag{1}$$

この記事では、Gorgoniaが上記式の勾配 $\nabla f$ を偏微分で評価する方法を示します。

$$ \nabla f = \left[\frac{\partial f}{\partial x}, \frac{\partial f}{\partial y}, \frac{\partial f}{\partial z} \right] $$

### 説明

合成関数の[連鎖律（チェインルール）](https://www.khanacademy.org/math/ap-calculus-ab/ab-differentiation-2-new/ab-3-1a/a/chain-rule-review)を用いて、下図の各ステップの導関数値を計算できます：

{{<mermaid align="left">}}
graph LR;
    x -->|$x=-2$<br>$\partial f/\partial x = -4$| add
    y -->|$y=5$<br>$\partial f/\partial y = -4$| add
    add(+) -->|$q=3$<br>$\partial f/\partial q = -4$| mul
    z -->|$z=-4$<br>$\partial f/\partial z = 3$| mul
    mul(*) -->|$f=-12$<br>$1$| f
{{< /mermaid >}}


{{% notice info %}}
勾配計算のより詳しい説明は、スタンフォード大学 [cs231n コース内の記事](http://cs231n.github.io/optimization-2/) を参照してください。
{{% /notice %}}

上記の勾配計算を[計算グラフ (exprgraph) ](/ja/reference/exprgraph) として記述し、Gorgonia が勾配計算をどのように行えばよいかを明確にします。

勾配計算が完了すると、グラフの各ノードは[２つの値](/ja/reference/dualvalue)を保持します。これらの値は、x に代入された値と、x について偏微分した値の２つです。

例として、ノード x について定義します：

```go
var x *gorgonia.Node
```

Gorgonia が計算グラフを評価し終えたあとに、`x` に代入された値と、偏導関数 $\frac{\partial f}{\partial x}$ の結果の値を次のように呼び出し可能です：

```go
xValue := x.Value()    // -2
dfdx, _ := x.Grad()    // -4, please check for errors in proper code
```

どのように実装するか具体的に見ていきましょう。

## 立式

まず最初に、式を表現する[計算グラフ (exprgraph)](/ja/reference/exprgraph) を作成します。

{{% notice info %}}
計算グラフ作成の詳しい過程は、チュートリアル [こんにちは世界](/ja/tutorials/hello-world/) を参照してくだい。
{{% /notice %}}

```go
g := gorgonia.NewGraph()

var x, y, z *gorgonia.Node
var err error

// define the expression
x = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("x"))
y = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("y"))
z = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("z"))
q, err := gorgonia.Add(x, y)
if err != nil {
    log.Fatal(err)
}
result, err := gorgonia.Mul(z, q)
if err != nil {
    log.Fatal(err)
}
```

各ノードに値を代入します：

```go
gorgonia.Let(x, -2.0)
gorgonia.Let(y, 5.0)
gorgonia.Let(z, -4.0)
```

### 微分実行

勾配計算を実行する２種類の自動微分方法があります：

* [LispMachine](/ja/reference/lispmachine) を用いた[自動微分 (automatic differentiation)](https://en.wikipedia.org/wiki/Automatic_differentiation)
* Gorgonia が提供する [数式微分 (symbolic differentiation)](https://en.wikipedia.org/wiki/Computer_algebra)


#### 自動微分

自動微分は [LispMachine](/ja/reference/lispmachine) を用いた場合のみ可能です。
デフォルトでは、lispmachine は前進法 (forward mode) と後進法 (backwards mode) に対応しています。

そのため、計算結果を得るには RunAll() メソッドを呼び出すだけです。
```go
m := gorgonia.NewLispMachine(g)
defer m.Close()
if err = m.RunAll(); err != nil {
    log.fatal(err)
}
```

結果の値と各ノードの導関数値を取り出せます：

```go
fmt.Printf("x=%v;y=%v;z=%v\n", x.Value(), y.Value(), z.Value())
fmt.Printf("f(x,y,z) = %v\n", result.Value())

if xgrad, err := x.Grad(); err == nil {
    fmt.Printf("df/dx: %v\n", xgrad)
}
if ygrad, err := y.Grad(); err == nil {
    fmt.Printf("df/dy: %v\n", ygrad)
}
if xgrad, err := z.Grad(); err == nil {
    fmt.Printf("df/dx: %v\n", xgrad)
}
```


#### 数式微分

もう一つの選択肢は、数式微分です。
数式微分は、計算グラフに新しいノードを追加することで実行されます。新しいノードは、引数として渡されたノードの導関数値を保持しています。

計算グラフに新しいノードを追加するために [Grad()](https://godoc.org/gorgonia.org/gorgonia#Grad) 関数を使用します。

Grad 関数は、コスト関数 (式 $1$) のスカラノード (a scalar cost node) と、入力値の複数ノード (a list of with-regards-to (WRTs)) を引数としてとり、導関数値を結果として返します。

下記のコードを参照してください：
```go
var grads Nodes
if grads, err = Grad(result,z, x, y); err != nil {
    log.Fatal(err)
}
```

コードは、`z`, `x`, `y` のノードをもとに (with regards to) 偏導関数値を計算するという意味です。



`grads` は `[]*gorgonia.Node` の配列であり、Grad() へ WRTs 引数として渡した順番と同じ並びで格納されています：

* `grads[0]` = $\frac{\partial f}{\partial z}$
* `grads[1]` = $\frac{\partial f}{\partial x}$
* `grads[2]` = $\frac{\partial f}{\partial y}$

導関数値は [TapeMachine](/ja/reference/tapemachine) と [LispMachine](/ja/reference/lispmachine) の結果と等しいです。しかしながら TapeMachine での実行速度のほうがより早いです。

```go
machine := gorgonia.NewTapeMachine(g)
defer machine.Close()
if err = machine.RunAll(); err != nil {
        log.Fatal(err)
}

fmt.Printf("result: %v\n", result.Value())
if zgrad, err := z.Grad(); err == nil {
        fmt.Printf("dz/dx: %v | %v\n", zgrad, grads[0].Value())
}

if xgrad, err := x.Grad(); err == nil {
        fmt.Printf("dz/dx: %v | %v\n", xgrad, grads[1].Value())
}

if ygrad, err := y.Grad(); err == nil {
        fmt.Printf("dz/dy: %v | %v\n", ygrad, grads[2].Value())
}
```

各導関数値には２つの取り出し方法があります：

1. `.Grad()` メソッドによる取り出し。例示したコードでの `x` の導関数値の取り出しは、`x.Grad()` を使用
2. `.Value()` メソッドによる取り出し。例示したコードでは、`grad` 配列のメソッドとして実行します。`grads[1].Value()` を実行することで `x` の導関数値が取り出せる

２種類の方法がある理由は、それぞれ適した場面があるからです。grad に格納されているノードから値を取り出すほうが意味があるときに grad のノードを使用します（例えば、二階導関数を計算する場面）。しかし、導関数値を高速に計算したい場面は、`.Grad()` メソッドが最も適しています。最終的には使用者の好みです。

## コード全体 (自動微分)

```go
func main() {
	g := gorgonia.NewGraph()

	var x, y, z *gorgonia.Node
	var err error

	// define the expression
	x = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("x"))
	y = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("y"))
	z = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("z"))
	q, err := gorgonia.Add(x, y)
	if err != nil {
		log.Fatal(err)
	}
	result, err := gorgonia.Mul(z, q)
	if err != nil {
		log.Fatal(err)
	}

	// set initial values then run
	gorgonia.Let(x, -2.0)
	gorgonia.Let(y, 5.0)
	gorgonia.Let(z, -4.0)

	// by default, lispmachine performs forward mode and backwards mode execution
	m := gorgonia.NewLispMachine(g)
	defer m.Close()
	if err = m.RunAll(); err != nil {
		log.fatal(err)
	}

	fmt.Printf("x=%v;y=%v;z=%v\n", x.Value(), y.Value(), z.Value())
	fmt.Printf("f(x,y,z)=(x+y)*z\n")
	fmt.Printf("f(x,y,z) = %v\n", result.Value())

	if xgrad, err := x.Grad(); err == nil {
		fmt.Printf("df/dx: %v\n", xgrad)
	}

	if ygrad, err := y.Grad(); err == nil {
		fmt.Printf("df/dy: %v\n", ygrad)
	}
	if xgrad, err := z.Grad(); err == nil {
		fmt.Printf("df/dz: %v\n", xgrad)
	}
}
```

上記コードは次の出力を得ます：

```text
$ go run main.go
x=-2;y=5;z=-4
f(x,y,z)=(x+y)*z
f(x,y,z) = -12
df/dx: -4
df/dy: -4
df/dz: 3
```

## コード全体 (数式微分)


```go
func main() {
	g := gorgonia.NewGraph()

	var x, y, z *gorgonia.Node
	var err error

	// define the expression
	x = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("x"))
	y = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("y"))
	z = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("z"))
	q, err := gorgonia.Add(x, y)
	if err != nil {
		log.Fatal(err)
	}
	result, err := gorgonia.Mul(z, q)
	if err != nil {
		log.Fatal(err)
	}

	if grads, err = Grad(result,z, x, y); err != nil {
		log.Fatal(err)
	}

	// set initial values then run
	gorgonia.Let(x, -2.0)
	gorgonia.Let(y, 5.0)
	gorgonia.Let(z, -4.0)

	machine := gorgonia.NewTapeMachine(g)
	defer machine.Close()
	if err = machine.RunAll(); err != nil {
		log.Fatal(err)
	}

	fmt.Printf("x=%v;y=%v;z=%v\n", x.Value(), y.Value(), z.Value())
	fmt.Printf("f(x,y,z)=(x+y)*z\n")
	fmt.Printf("f(x,y,z) = %v\n", result.Value())

	if zgrad, err := z.Grad(); err == nil {
		fmt.Printf("dz/dx: %v | %v\n", zgrad, grads[0].Value())
	}

	if xgrad, err := x.Grad(); err == nil {
		fmt.Printf("dz/dx: %v | %v\n", xgrad, grads[1].Value())
	}

	if ygrad, err := y.Grad(); err == nil {
		fmt.Printf("dz/dy: %v | %v\n", ygrad, grads[2].Value())
	}
}
```

上記コードは次の出力を得ます：

```text
$ go run main.go
x=-2;y=5;z=-4
f(x,y,z)=(x+y)*z
f(x,y,z) = -12
df/dx: -4 | -4
df/dy: -4 | -4
df/dz: 3 | 3
```
