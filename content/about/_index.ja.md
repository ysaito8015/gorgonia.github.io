+++
title = "Gorgoniaの仕組み"
date = 2019-10-28T11:41:02+01:00
description = "このセクションにはGorgoniaの仕組みを説明することを目的とした記事が含まれています。"
weight = -9
chapter = true
+++

# このセクションについて

Gorgonia は、計算グラフを定義してから実行する仕組みです。数学関数のみを含み、条件分岐（if/then やループ）を持たない一種のプログラミング言語とみなされます。このようにみなすことは、実際にユーザの思考の流れに沿っているはずの有力なパラダイムです。計算グラフは [AST （抽象構文木）](http://en.wikipedia.org/wiki/Abstract_syntax_tree) と同義です。

計算グラフの構築と計算グラフの実行は異なるものであり、ユーザがそれぞれを構築・実行する際には異なる思考の流れであるべきだという考えを、ネットワーク（計算グラフ）記述言語 BrainScript を同梱している Microsoft 社の [CNTK](https://github.com/Microsoft/CNTK) は最もよく例証しているといえます。

Gorgonia の実装は、CNTK と BrainScript のように思考の分離を強調するものではありませんが、それらの思考の流れは少々役立ちます。

## 以降のセクション内容について


このセクションには Gorgonia の仕組みを説明することを目的とした記事が含まれています。

{{% notice info %}}
このセクションの記事は、理解を助けるものであり、また、背景や経緯を説明します。
{{% /notice %}}

{{% children description="true" %}}
