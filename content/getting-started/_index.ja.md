+++
title = "事始め"
date = 2019-10-29T17:42:44+01:00
weight = -10
description = "Quick start with Gorgonia"
chapter = true
+++

## Gorgoniaの入手

Gorgoniaはgo-get可能でありgo modulesをサポートしています。
ライブラリとその依存物を取得するには単純に以下を実行します。

```bash
$ go get gorgonia.org/gorgonia
```

## 簡単な計算をする為の初めてのコード

配管が正常かどうかを確認する簡単なプログラムを作成します:

```go
package main

import (
        "fmt"
        "log"

        "gorgonia.org/gorgonia"
)

func main() {
        g := gorgonia.NewGraph()

        var x, y, z *gorgonia.Node
        var err error

        // define the expression
        x = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("x"))
        y = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithName("y"))
        if z, err = gorgonia.Add(x, y); err != nil {
                log.Fatal(err)
        }

        // create a VM to run the program on
        machine := gorgonia.NewTapeMachine(g)
        defer machine.Close()

        // set initial values then run
        gorgonia.Let(x, 2.0)
        gorgonia.Let(y, 2.5)
        if err = machine.RunAll(); err != nil {
                log.Fatal(err)
        }

        fmt.Printf("%v", z.Value())
}
```

プログラムを実行するとこの結果が出力されるはずです： `4.5`

もし上記の出力の代わりに次のエラーメッセージが表示された場合は：

```go
panic: Something in this program imports go4.org/unsafe/assume-no-moving-gc to declare that it assumes a non-moving garbage collector, but your version of go4.org/unsafe/assume-no-moving-gc hasn't been updated to assert that it's safe against the go1.19 runtime. If you want to risk it, run with environment variable ASSUME_NO_MOVING_GC_UNSAFE_RISK_IT_WITH=go1.19 set. Notably, if go1.19 adds a moving garbage collector, this program is unsafe to use.
```

シェル上で次のコマンドを実行してください。

```bash
$ export ASSUME_NO_MOVING_GC_UNSAFE_RISK_IT_WITH=go1.19
```

そして、Go プログラムを再度実行してください。

詳細については[Hello Worldチュートリアル](/ja/tutorials/hello-world)を参照してください。

