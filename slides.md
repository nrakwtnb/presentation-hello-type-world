---
theme: seriph
highlighter: prism
#highlighter: shiki
lineNumbers: false
background: /cover_background.jpg
---

# Print "Hello type world !"

<br><br>

### ver 2023.4.11

<br>

## N. Watanabe.

### @内輪向けゆるふわ会(4/9)

---

# (Rough) Introduction

普段何気なく行っているプログラミング。
その背景となる理論は、計算機科学と呼ばれる分野の中で発展してきているが、
その起源はどこに求められるだろうか？

最も賛同を得られると思われるのは、Turing機械であると思われる。
しかしTuring機械が提案された時代は、「計算」という行為の定義が真面目に再考された時代である。
実はその時代から既に型という概念は存在していた。

この計算および型というキーワードを軸に、その周囲の発展について本発表では簡単に紹介していく。

---

# Disclaimer ?

- 発表者は、型理論の専門家ではないため、
ところどころ怪しい記述がありますが、私がそう認識している箇所では「と思います」などと表現している事に注意してください。

- またコードスニペットは疑似コード（TypeScriptとRust、一部Pythonを混ぜた様な感じ）であり、特定のプログラミング言語を念頭に置くものではありません。

- 分かりにくい箇所や間違い等ありましたらご指摘ください。

- 余力があればZennなどに投稿予定。

---

# About types

そもそも型とは何か、について述べていく。

型は、値・オブジェクト・関数・変数などに割り当てられる、その性質を記述するタグあるいは分類ラベルである。

なぜそのようなラベルが必要か？あるいはどのように役立つのか？
もちろん様々な側面が存在するが、最も重要な点・答えはやはり

<center>

<b>(主に静的型付け言語において)プログラムの実行前に不整合性 (=エラー)を検出可能<br>
という軽量形式検証としての役割</b>

</center>

だと思われる。

一般にプログラムは複雑であり、また実行する事は高コストであるため、
プログラム中のあらゆる所に粗い目印をつけていき/おき、その目印が一貫したものか、を確認する事で
おかしな事 (=動作が未定義の状態に陥る事)が実行時に起きない事を保証するのである。
これは一般に **型安全性 (type safety)** とも呼ばれており、モダンな型付き言語では(おおよそ)保証されている。

---

# Atomic Type

通常、多くのプラミング言語には、数値や真偽値、文字といったそれ以上分解できない値が定義されている。
このような値に対応する型も、通常、これ以上分解できない基本的な単位をなしている。
このような型を
**原子型 (atomic type)**
（あるいは **基本型 (base type)** や **プリミティブ型 (primitive type)** などとも）
と呼ぶ。

後で見るように、基本的に、他の型はこの原子型を用いて定義される。多くのプログラミング言語で導入されている代表的な例を以下に挙げる：

- $\mathbf{Int}$: 整数型（ここでは、上限や下限などは気にしない）
    - $0, 1, 2, -1, -2, \ldots$
- $\mathbf{Float}$: 浮動小数点型
    - $1.0, -1.375, 16777215.984375, \ldots$
    - 多くの言語で、上限や下限、精度など毎に異なる型が定義されているが、ここでは気にしない
- $\mathbf{Char}$: 文字型
    - ASCIIコードの文字 $a, b, c, \ldots$ に加え、各種ユニコード文字 $\textrm{あ}, \textrm{金}, \textrm{α}, \ldots$
- $\mathbf{Bool}$: 真偽値型
    - $\mathrm{True}$ と $\mathrm{False}$

---

# Compound type

先に挙げた基本的な値を組み合わせる事で、各種データ構造を作り出すことができる。
それに対応して、基本型を組み合わせた型が構成される。
これを **複合型 (compound type)** と呼ぶ。

- タプル (Tuple)型：$\mathbf{A}_1 \times \mathbf{A}_2 \times \cdots \times \mathbf{A}_n = \prod_{i=1}^{n} \mathbf{A}_i$
    - 特に $n=2$ の時 ($\mathbf{A} \times \mathbf{B} = \mathbf{Pair}[\mathbf{A}, \mathbf{B}]$)、対または積 (Pair, Product) と呼ぶ
- Variant型：$\mathbf{A}_1 + \mathbf{A}_2 + \cdots + \mathbf{A}_n = \sum_{i=1}^{n} \mathbf{A}_i$
    - 順序付きUnion、あるいはtagged Unionとも呼ばれる
    - 特に $n=2$ の時 ($\mathbf{A} + \mathbf{B} = \mathbf{Either}[\mathbf{A}, \mathbf{B}]$)、Either型などと呼ばれる事もある
- リスト（可変配列）： $\mathbf{List}[\mathbf{A}] = [\mathbf{A}]$ と表される
    - 形式的には、$\mathbf{List}[\mathbf{A}] = \sum_{n \in \mathbb{N}} \mathbf{A}^{n}$ とも表現できる
- レコード（固定キーの辞書、連想配列、マップなど）：$\{ k_1: \mathbf{A}_1, k_2: \mathbf{A}_2, \ldots, k_n: \mathbf{A}_n \}$
    - $k_i$ はキーを表し、典型的には $\mathbf{String}$ 型を指定する
    - ここでは簡単のため、キーは予め固定されたケースを考える
    - 理論的に扱う場合は、順序を入れる事もある

---

# Function type

多くのプログラミング言語において、最も基本的かつ重要な言語仕様として、関数が挙げられる。
関数に対応する型はその名の通り **関数型 (arrow type / function type)** と呼ばれる。
特に、型 $\mathbf{X}$ の引数を取り、型 $\mathbf{Y}$ の返り値を返す関数の方は、$\rightarrow$ を用いて

$$\mathbf{X} \rightarrow \mathbf{Y}$$

と表せる。
他変数の場合は、例えば3変数関数については

$$
(\mathbf{X}_1 \times \mathbf{X}_2 \times \mathbf{X}_3) \rightarrow \mathbf{Y}
$$

のように表せる。あるいはこれをCurry化した

$$
\mathbf{X}_1 \rightarrow \mathbf{X}_2 \rightarrow \mathbf{X}_3 \rightarrow \mathbf{Y}
=
\mathbf{X}_1 \rightarrow (\mathbf{X}_2 \rightarrow (\mathbf{X}_3 \rightarrow \mathbf{Y} ))
$$

も頻繁に用いられる。

ここで先の原子型を用いた具体例を挙げておく：

- $\mathbf{Int} \to \mathbf{Float}$: $\mathbf{Int}$ 型の引数を受け取り、$\mathbf{Float}$ 型の値を返す
- $\mathbf{Int} \times \mathbf{Char} \to \mathbf{Bool}$: $\mathbf{Int}$ 型と $\mathbf{Char}$ 型の引数を受け取り、$\mathbf{Bool}$ 型の値を返す

---

# Type operator (Type costructor)

先程見たように複雑なデータ構造の型は、原子型を用いて構成される。例えばリストは、型 $\mathbf{A}$ を与えると、リスト型を $\mathbf{List}[\mathbf{A}]$ を返すとみなせる。
また関数型も、（単引数関数の場合）入出力の型の対 $(\mathbf{X}$, $\mathbf{Y})$ を与え、新しい型 $\mathbf{X} \rightarrow \mathbf{Y}$ が定義される、と見なす事ができる。

特にこれらを何度も組み合わせる事で幾らでも複雑な型、例えば、

$$
\{ a: \mathbf{List}[\mathbf{A} + \mathbf{B}] \times \mathbf{A} \}
\times
(\mathbf{B}+\mathbf{C} \rightarrow \mathbf{A}) \rightarrow (\{ s: \mathbf{A} + \mathbf{B} \} \rightarrow \mathbf{D})
$$

のような型を構成する事ができる。このように、値や関数を組み合わせて複雑な式を構築していくのに対応して、
型の世界でも、複雑な型が作られていく。

このような、既存の型から新しい型を生成する操作を
**型演算子 (type operator)** あるいは **型構築子 (type constructor)**
と呼ぶ。これは型の世界における「関数」（ **抽象化** とも言う）に他ならない。

以降、型と対比して、値や関数を全て引っくるめて本発表では **オブジェクト** と呼ぶ事にする。(型理論・形式文法においては、 **項** と呼ばれる)

また、定数が$0$引数関数と同一視されたように、通常の型は$0$引数型演算子と同一視できる（＝通常の型は型演算子の一種）事に注意する。

---

# Kind

型演算子は型世界の関数であるが、オブジェクトの世界と同じく何でも渡して良いわけではない。
例えば、
$\mathbf{List}[\mathbf{Pair}]$
や
$\mathbf{Either} \rightarrow \mathbf{Int}$
のような型は意味をなさないため、禁止する必要がある。
そのためには、型の世界にも、『型』を導入すればよい。
型演算子の『型』を **カインド(kind)** と呼ぶ。

#### Proper type

対応するオブジェクトを持つ事が許される通常の型 $\mathbf{Int}$ や $\mathbf{List}[\mathbf{Char}]$ などは、
$\ast = \mathbf{Type}$
のカインドを持つとし、これを

$$\mathbf{Int}: \ast \;, \quad\mathbf{List}[\mathbf{Char}]: \ast$$

と表す(文献によっては、 $\mathbf{Int}:: \ast \rightarrow \ast$ とも)。この通常の型は、 **真の型 (proper type)** と呼ばれる。

#### Single-variable type operator

$\mathbf{List}$ は型を一つ受け取り、新たな型を一つ返す。このような型演算子はカインド $\ast \rightarrow \ast$ を持ち、これを

$$
\mathbf{List}: \ast \rightarrow \ast
$$

のように表現する。他にも $\mathbf{Optional} = \mathbf{Maybe}$ などがこれに該当する。

---

#### Multi-variables type operators

一般に $n$個の真の型を受け取って、新しい型を返すような型演算子のカインドは、$n=2$ であり、

$$
\underbrace{
\ast \rightarrow \ast \rightarrow \ast \rightarrow \cdots \rightarrow \ast
}_{n+1 \textrm{個の} \ast}
$$

として与えられる。例えば、対や関数を作る型演算子は

$$
\mathbf{Pair}: \ast \rightarrow \ast \rightarrow \ast
\;,\quad
\mathbf{Func}: \ast \rightarrow \ast \rightarrow \ast
$$

である。これらを組み合わせたより複雑な型演算子、例えば、型 $X$ と $Y$ を引数に取り、それらの直和型 $X + Y$ のリストを返すものを、関数定義風に書くと、以下のようになる：

```typescript
function List_of_sum(X: Type, Y: Type) -> Type {
    [X+Y]
}
```

特にこの時、

$$
\mathrm{List\_of\_sum}: \ast \rightarrow \ast \rightarrow \ast
$$

である。

---

#### Higher-order type operators

型演算子に渡せるのは、必ずしも真の型に限らない。型演算子も渡せる。ただし抽象度が非常に高いので、おそらくメタプログラミングレベルで有益であると思われる。
無理矢理例を作るのであれば

```typescript
function state_and_evolution(M: Type -> Type) -> Type {
    (M[Int], M[Int] -> M[Int])
}
```

と定義すると、これは整数から作られる何らかの型を状態に持ち、それ自身に作用するメソッドを持つようなオブジェクトを表現する事ができる型である。
この時、 state_and_evolution 型演算子は、カインド $(\ast \rightarrow \ast) \rightarrow \ast$ を有する。

さらにカインド $(\ast \rightarrow \ast) \rightarrow \ast$ を持つ型演算子を渡せるような型演算子を考える事もできる、という事を（無限に）繰り返していく事も原理的には可能である。
ただし実用上は、型演算子の引数として、真の型のみを許容するように制限したケースがほとんどである。

いずれにせよ、型演算子が織りなす世界が、型の真の姿である事が分かった(関数がオブジェクトの一種であり、第一級の市民権を得ているのと同じ構図)。
これを踏まえ、以降、**型演算子（真の型含む）の事を「型」と改めて呼ぶ**事にする。

---

# Object world v.s. type world ?

ここまで述べた事を以下のように整理する：

- オブジェクトの住む世界（地上）
    - 真の型（その集まりを $\ast$とかく）：オブジェクトが住む世界のラベル
    - 関数：オブジェクト（関数含む）を入出力に持つ
- 型の住む世界（天空）
    - カインド（その集まりを $\square$ という記法で表す）：型が住む世界のラベル
    - 型演算子：型（真の型、型演算子）を入出力に持つ

このように、型の住む世界にも「関数」が自然に存在する事が判った。
オブジェクトの世界は、複雑で泥臭い地上の世界（定義される関数や計算も複雑）であるのに対し、
型の世界は、（情報を粗くしている分）単純で小綺麗な天上の世界（定義される型演算子や計算も単純）であると言える。
しかしここで次の疑問が思い浮かぶ：

<center>

**オブジェクトの世界（真の型で制御）と型の世界（カインドで制御）は断絶しているのか？**

</center>

以降は、この世界の壁に大きな"穴"を開けてつないでいく事を考える。

---

# Polymorphic types

まずは、天上の型の世界（単純・明快）から地上のオブジェクトの世界（複雑・混沌）に降りていく方向、すなわち、型を受け取って、オブジェクトを返すような「関数」を考える。
例えば、

```typescript
function head_for_T(T: Type) -> (List[T] -> T) {
    function head(x: list[T]) -> T {
        get_index(x, 0)
    }
}
```

の内側で定義されているような、リストを与えると先頭の要素を返す関数 headは、リストが許容している型 $\mathbf{T}$ の性質に依らず、定義される。
そしてこれを生成する関数 head_for_T は、型 $\mathbf{T}$ を受け取り、オブジェクト head関数を返すため、この例に該当する。

これは **(パラメトリック)多相型 (parametrics polymorphism)** あるいは **総称型・全称型(generics)** と呼ばれる。
特にパラメトリック多相型は、型 $\mathbf{S}$ を与えた時、型 $F(\mathbf{S})$ のようにかけるとすると、
$\forall \mathbf{T}. F(\mathbf{T})$
のように表示される。

例えば、
多相恒等関数 id であれば、その型は
$\mathrm{id}: \forall \mathbf{T}. \mathbf{T} \rightarrow \mathbf{T}$
で与えられるし、先の head_for_T関数であれば、
$\mathrm{head\_for\_T}: \forall \mathbf{T}. \mathrm{List}[\mathbf{T}] \rightarrow \mathbf{T}$
と表される。

---

# Dependent types

断絶したオブジェクトの世界（地上）と、型の世界（天上）に対し、天から舞い降りる事を許すのがパラメトリック多相であったが、
逆に地上から昇天するようなパスを考える事もできる。これにより、天上の世界は、比較的守られた世界から一転、混沌とした世界へと変貌する。

これはオブジェクトを渡し、型を返すような「関数」を考える事に相当する。
言い換えると、型がオブジェクトに依存しているような状況を許す事を意味する。よって、このような型は
**依存型(dependent type)** と呼ばれる。
特に、型 $\mathbf{A}$ のオブジェクト $a$ に依存する型は、

$$
\Pi_{a: \mathbf{A}}. \mathbf{B}(a) = \Pi {a: \mathbf{A}}. \mathbf{B}_a
\qquad
\mathbf{B}_a \textrm{は$a$でパラメトライズされる型}
$$

のように書かれる。(気持ちとしては、型の族を表す。)

ただし、2023年の現代的なプログラミング言語においても、依存型を"直接"サポートするものは数少なく (有名どころはAgdaおよびIdris)、(私が知る限り) 実践的なものはほとんどない。
それは依存型が活躍できる場面が希少だからではなく (むしろ多相型よりありふれている)、
型検査コストが跳ね上がるためであると考えられる。
この事について理解するために、依存型の最も基本的な例である、長さ情報を持つベクタ型をまずは取り上げる。

---

# 例：固定長ベクタ型

まず $\mathbf{Vec}(n, \mathbf{T})$ を型 $\mathbf{T}$ を許容する長さ $n$ のベクタを表す型とする。これは $\underbrace{\mathbf{T} \times \mathbf{T} \times \cdots \times \mathbf{T}}_{n}$ と等値 (=所属する要素の1対1対応が作れる意) である。
この時、$\mathbf{T} = \mathbf{Int}$ と固定し、オブジェクト $n$ が属する自然数の型 $\mathbf{Nat}$ を用いて、
整数ベクタ型を

$$
\mathbf{Vec}_{\mathbf{Int}} = \Pi_{n: \mathbf{Nat}}. \mathbf{Vec}(n, \mathbf{Int})
$$

と表記する。

このカインドは $\mathbf{Nat} \rightarrow \ast$ に対応し、長さ $k$ を引数として与えると、その長さのベクタ型を返す：

$$
\mathbf{Vec}_{\mathbf{Int}}(k) = \mathbf{Vec}(k, \mathbf{Int})
\;.
$$

なお、 $\mathbf{Vec}_{\mathbf{Int}}$ が生成する型全体は $\mathbf{List}[\mathbf{Int}]$ と等値でもある。

ちなみに整数型を一般の型 $\mathbf{T}$ に戻すと、ベクタ型は型演算子かつ依存型の例 (型とオブジェクトを引数により、型を返す)となっている。

---

# 固定長ベクタ型を用いた型検出

固定長ベクタ型に対して、例えば先頭以外の要素を取得する tail 関数を考える。

これは、長さ $n+1$ のベクタを受け取り、長さ $n$ のベクタを返すため、その型は

$$
\Pi_{n: \mathbf{Nat}}. (\mathbf{Vec}(n+1, \mathbf{Int}) \rightarrow \mathbf{Vec}(n, \mathbf{Int}))
$$

で与えられる。すなわち $\mathrm{tail}(n):\mathbf{Vec}(n+1, \mathbf{Int}) \rightarrow \mathbf{Vec}(n, \mathbf{Int})$ であり、これは **長さ $n+1$ の整数ベクタ型に対してのみ** 作用できる。

ここで重要な点として、
長さ $0$ の空ベクタに対し、tail関数は(一般的に)未定義（エラーを引き起こしうる）である。
このエラーは従来の $\mathbf{List}[\mathbf{Int}]$ の粒度での型検査では検出できない。

しかしながら、長さ情報を持つ依存型 $\mathbf{Vec}_{\mathbf{Int}}$ 型を用いると、head関数の入力は長さ $0$ のベクタを許容していない事が型検査によって検出できるため、エラーを未然に防ぎうる。言い換えると、依存型は、オブジェクトによってパラメトライズ・分割される細かい情報を与える事で、型システムの表現力を高めている。

一方で、長さの情報を静的検査の時点で計算し、追跡する必要があるため、型検査コストが増える事が分かる。特に、型の長さが事前分からないような動的に変化するケース（データの読み書きやAPIの呼び出し、乱数に応じたオブジェクトの生成）への対応が難しい問題も存在する。

---

# その他の例 (1)

以下、もう少しだけ非自明な例 (ただしあまり適切とは言い難い)を考えていく。

#### データベースのスキーマを表す型

カラム名毎にデータ型が異なるので、これを依存型で表現してみる。

- $\mathbf{Column}$: あるDBに含まれるカラム名(文字列)のなす型
- $\mathbf{TypeSchema} = \Pi_{c: \mathbf{Column}} \mathbf{Data}_c$: データベースの型情報を表す型
    - $\mathbf{TypeSchema}(c)$: カラム $c$ が格納するデータの型
- ここではデータベースとしたが、結局はインタフェースの定まったデータを格納するオブジェクト（キーの固定されたレコード型）に他ならない

---

# その他の例 (2)

#### テンプレートリテラル・フォーマッタを表す型

`Today's date is (%d).` といったテンプレートリテラルも依存型の一種である。

- $\mathbf{Template}$: テンプレートを表す型 (不正なテンプレートを含むが、文字列の型と考えて良い)
- $\mathbf{TemplateLiteral} = \Pi_{t: \mathbf{Template}} \mathbf{Format}_t$: テンプレートリテラルを表す型
    - $\mathbf{TemplateLiteral}(t) = \mathbf{Format}_t$: テンプレート $t$ が生成するフォーマット関数のなす型
    - 例：（テンプレート、フォーマット関数の型）
        - (`My name is (%s).`, $\mathbf{String} \rightarrow \mathbf{String}$)
        - (`My birthday is (%d)/(%d).`, $\mathbf{Int} \rightarrow \mathbf{Int} \rightarrow \mathbf{String}$)
- テンプレートを結合した場合に、フォーマット関数がどう変化するか、なども求められる
- ただし型検査時にテンプレートの解析が必要になるため、型検査のコストが増大する

---

# Summary at this stage

これまでに見てきた事を整理したものが以下の表である：

<div align="center">

<style>
.table th {
    font-weight: bold;
    text-align: center;
    line-height: 1;
}
</style>

<table class="table" style="width:100%; text-align: center;">
<tr>
  <th>

名称

  </th><th>

入力

  </th><th>

出力

  </th><th>

例

  </th><th>

型・カインド

  </th>
</tr>



<tr><td rowspan=3>

関数

</td><td rowspan=3>

オブジェクト

</td><td rowspan=3>

オブジェクト

</td><td>

入力が$0$か否かを判定

</td><td>

$\mathbf{Int} \rightarrow \mathbf{Bool}$

</td></tr><tr><td>

文字列を指定回数だけ反復させる

</td><td>

$\mathbf{Int} \rightarrow \mathbf{String} \rightarrow \mathbf{String}$


</td></tr><tr><td>

数値の変換が、特定の条件を満たすか検査

</td><td>

$(\mathbf{Int} \rightarrow \mathbf{Int}) \rightarrow \mathbf{Bool}$

</td></tr>


<tr><td rowspan=3>

型演算子

</td><td rowspan=3>

型

</td><td rowspan=3>

型

</td><td>

$\mathbf{List}$

</td><td>

$\ast \rightarrow \ast \rightarrow \ast$

</td></tr><tr><td>

$\mathbf{Func}$

</td><td>

$\ast \rightarrow \ast$

</td></tr><tr><td>

($\ast \to \ast$の)型演算子に型を代入して返す

</td><td>

$(\ast \rightarrow \ast) \rightarrow \ast \rightarrow \ast$

</td></tr>




<tr><td rowspan=3>

パラメトリック多相型

</td><td rowspan=3>

型

</td><td rowspan=3>

オブジェクト

</td><td>

$\mathrm{head\_for\_T}$

</td><td>

$\forall \mathbf{T}. \mathbf{List}[\mathbf{T}] \rightarrow \mathbf{T}$

</td></tr><tr><td>

$\mathrm{reverse\_for\_T}, \mathrm{sort\_for\_T}$

</td><td>

$\forall \mathbf{T}. \mathbf{List}[\mathbf{T}] \rightarrow \mathbf{List}[\mathbf{T}]$

</td></tr><tr><td>

等値性判定

</td><td>

$\forall \mathbf{T}. \mathbf{T} \times \mathbf{T} \rightarrow \mathbf{Bool}$

</td></tr>



<tr><td rowspan=3>

依存型

</td><td rowspan=3>

オブジェクト

</td><td rowspan=3>

型

</td><td>

$\Pi_{n: \mathbf{Nat}} \mathbf{Vec}_n$

</td><td>

$\mathbf{Nat} \rightarrow \ast$

</td></tr><tr><td>

$\mathbf{TemplateLiteral}$

</td><td>

$\mathbf{String} \rightarrow \ast$

</td></tr><tr><td>

$\mathbf{TypeSchema}$

</td><td>

$\mathbf{Column} \rightarrow \ast$

</td></tr>

</table>

</div>

---

# 単純型付き$\lambda$計算 $\lambda_\to$

時間の都合上詳細は省略するが、
変数および関数操作(作成と適用)からなる計算体系 (=プログラミング言語の仕様)を **$\lambda$計算** と呼ぶ。
これは、いわゆる、最も純粋な関数型プログラミング言語に他ならず、全てのもの (数値・真偽値などのプリミティブ、加減乗算やビットシフトなどの演算、条件分岐・再帰などの言語機能) は$\lambda$式と呼ばれる関数のみで記述する。
それにも関わらず、実はTuring完全であるぐらい表現力が高い事が知られている。

これに適切な型が付くべし、と制約を加えたものが **単純型付き$\lambda$計算 $\lambda_\to$** と呼ばれる計算体系である。
実はこの制約により再帰が禁止となり、Turing完全ではなくなるものの（代わりに計算が必ず停止する事が保証される）、依然として豊富な表現力 (=複雑な計算関数を実現する能力)を誇る。

---

# Pythonによる実装例 (型無し$\lambda$計算)

2や3、5といった数値、それらの足し算を全て(匿名)関数の定義(抽象化)と関数を並べて代入(関数適用)で表現している。
特に five は5のlambda式表現、five_ はtwoとthreeをplusに渡した返り値、とした時、関数としては全く等価である事が確認できる。
具体的には、両方とも2変数関数であり、引数に何を渡しても同じ返り値 (例外発生含む)を返すため、fiveとfive_は全く同じ関数を表す(Pythonの内部抽象表現は異なる)と言える。

```python
>>> two = lambda s: lambda z: s(s(z)) # 2を表現する
>>> three = lambda s: lambda z: s(s(s(z))) # 3を表現する
>>> five = lambda s: lambda z: s(s(s(s(s(z))))) # 5を表現する
>>> plus = lambda m: lambda n: lambda s: lambda z: m(s)(n(s)(z)) # 足し算を表現する
>>> five_ = plus(three)(two)

>>> five(lambda x: x+1)(0)
5
>>> five_(lambda x: x+1)(0)
5
>>> five(lambda x: x+[None])([])
[None, None, None, None, None]
>>> five_(lambda x: x+[None])([])
[None, None, None, None, None]
```

<div style="font-size: 7pt;">

※Pythonはweakな戦略(lambda式の中身を関数呼び出し前に評価しない)であるため、引数に適当な値を渡さないと等価性を確認できないが、Strong戦略の場合は、関数定義としても一致する。

</div>

---

#### 型を付与

Pythonは動的型付けの言語であるが、どういう型が必要か推論する事はできる。
先の例で言うと

```python
>>> five = lambda s: lambda z: s(s(s(s(s(z))))) # 5を表現する
```

に対し、適当な型 $\mathbf{X}$ を用いて、 sの型は $\mathbf{X} \rightarrow \mathbf{X}$、zの型は $\mathbf{X}$ であり、
fiveの型は $(\mathbf{X} \rightarrow \mathbf{X}) \rightarrow \mathbf{X} \rightarrow \mathbf{X}$
となる。特に

```python
>>> five(lambda x: x+1)(0)
5
```

は $\mathbf{X} = \mathbf{int}$ 型で、`lambda x: x+1` は型を満たせば `lambda x: x**3` などでも良い。

```python
>>> five(lambda x: x+[None])([])
[None, None, None, None, None]
```

は $\mathbf{X} = \mathbf{list}[\mathbf{Any}]$ 型であり、やはり型を満たせば `[]` を `[[1, 2], "a"]` など適当に置き換えても、先のfiveとfive_の等価性は成立する。

このように、$\mathrm{five}: (\mathbf{Int} \rightarrow \mathbf{Int}) \rightarrow \mathbf{Int} \rightarrow \mathbf{Int}$ と関数・変数に型を付与したものが、$\lambda_\to$ である。

---

# $\lambda$-cube

<div class="grid grid-cols-[70%,30%] gap-4"><div>

先の単純型付き$\lambda$計算 $\lambda_\to$ に対し、これまで見てきた3つの機能を付与する/しない事で、計 $2^3 = 8$ つの計算体系が手に入る。これを以下の表にまとめた。またこれを図示したものが右図であり、 (Barendregtの)**$\lambda$-cube** と呼ばれている。

</div><div class="center">

<center>

<img style="width: 45%; padding-bottom: 0px; padding-top: 0px;" src="/lambda-cube.png">

</center>

<div class="less" style="font-size: 6pt; line-height: 1;">

From Fig1. of Barendregt, *Introduction to generalized type systems* (1991)

</div>

</div></div>


<div align="center">

<style>
th {
    font-weight: bold;
    text-align: center;
    line-height: 1;
}
</style>

<table class="table" style="width: 100%; text-align: center;">
<tr>
  <th style="width: 20%;">

計算体系

  </th><th style="width: 18%;">

パラメトリック多相

  </th><th style="width: 18%;">

型演算子

  </th><th style="width: 18%;">

依存型

  </th><th style="width: 26%;">

名称

  </th>
</tr>



<tr><td>

$\lambda_{\rightarrow}$

</td><td>

</td><td>

</td><td>

</td><td>

単純型付き$\lambda$計算

</td></tr>



<tr><td>

$\lambda_{2}$

</td><td>

$\checkmark$

</td><td>

</td><td>

</td><td>

System $F$

</td></tr>



<tr><td>

$\lambda_{\underline{\omega}}$

</td><td>

</td><td>

$\checkmark$

</td><td>

</td><td>

</td></tr>



<tr><td>

$\lambda_{\omega}$

</td><td>

$\checkmark$

</td><td>

$\checkmark$

</td><td>

</td><td>

system $F_{\omega}$

</td></tr>



<tr><td>

$\lambda_{P}$

</td><td>

</td><td>

</td><td>

$\checkmark$

</td><td>

Logical Framework

</td></tr>



<tr><td>

$\lambda_{P2}$

</td><td>

$\checkmark$

</td><td>

</td><td>

$\checkmark$

</td><td>

</td></tr>



<tr><td>

$\lambda_{P\underline{\omega}}$

</td><td>

</td><td>

$\checkmark$

</td><td>

$\checkmark$

</td><td>

</td></tr>



<tr><td>

$\lambda C = \lambda_{P \omega}$

</td><td>

$\checkmark$

</td><td>

$\checkmark$

</td><td>

$\checkmark$

</td><td>

Calculus of Constructions (CoC)

</td></tr>

</table>

</div>

---

# Apperance of Coq

先に見た各計算体系は、あくまで数学的なモデルに過ぎない。
しかしながら、これらをベースにしたプログラミング言語は幾つか存在する。

特に、最も型に対する表現力の高い $\lambda C$ をベースとし、幾つかの文法機能（Inductive type・再帰など）を加えた実装として、Coq (正確にはGallina)と呼ばれる言語が存在する。
このCoqはプログラミング言語としても使用できるが、特筆すべきは **対話型定理証明(支援)器 (interactive theorem prover)** としての機能を提供している事にある。

主に2つの応用先がある：

- 数学の定理を形式化（誤りのない、数学の証明として記述）
    - 四色定理（任意の地図を4色で塗り分けられる）、Kepler予想（面心立方格子が最密充填）
- 関数が仕様を満たす事を保証
    - CompCert（信頼性の高いCコンパイラ）、OpennSSLの脆弱性発見

なぜ先の計算体系が証明につながるのか、この背景にある考え方について、簡単に説明する。

<footer>

See https://staff.aist.go.jp/reynald.affeldt/ssrcoq/coq-tutorial.pdf for details.

</footer>

---

# 判断の解釈

この鍵となるのは、
$x: \mathbf{X}$
という記号 (判断と呼ぶ)の解釈方法である。
これまでは、オブジェクト $x$ の型が $\mathbf{X}$ である事を意味していた。
しかしながら、それ以外にも様々な解釈が考えられる：

- $\mathbf{X}$ は型であり、 $x$ はそれに属するオブジェクト(関数)である（計算体系におけるこれまでの解釈）
- $\mathbf{X}$ は命題であり、 $x$ はそれに対する証明である（論理学における解釈）
- $\mathbf{X}$ は問題であり、 $x$ はそれに対する解決策である
- $\mathbf{X}$ は集合であり、 $x$ はそれに属する要素である（集合論）
- $\mathbf{X}$ は空間であり、 $x$ はそれに属する点である（HoTT）

特に今回関連するのが、2番目の

<div class="center">

**$\mathbf{X}$ は命題であり、 $x$ はそれに対する証明である**

</div>

という見方である。これを信じるならば、計算体系において、関数は計算とほぼ同義なので、計算と証明が対応づく事が示唆される。
そしてそれが幻想ではなく、各種記号(型演算子)などに対してもきちんと対応関係がつく事を今から眺めていく。

---

# $\mathbf{A} \rightarrow \mathbf{B}$ の解釈と規則

<div class="grid grid-cols-2 gap-4" style="font-size: 12pt;"><div>

#### 計算体系

- 解釈 (関数)：型 $\mathbf{A}$ を受け取り、型 $\mathbf{B}$ を返す関数の型
- 導入規則 ($\lambda$抽象)
    - 型 $\mathbf{A}$ のオブジェクト $a$ が与えられた時、それに依存しうるオブジェクト $t_a$ の型が $\mathbf{B}$ であったとすると、関数 $f(a) = t_a$ の型は $\mathbf{A} \rightarrow \mathbf{B}$ で与えられる
- 除去規則 (関数適用)
    - 型 $\mathbf{A}$ のオブジェクト $a$ と、型 $\mathbf{A} \rightarrow \mathbf{B}$ の関数 $f$ があったら、型 $\mathbf{B}$ のオブジェクト $f(a)$ が構成できる

</div><div>

#### 論理体系

- 解釈 (含意)：命題 $\mathbf{A}$ は、命題 $\mathbf{B}$ を示唆する
- 導入規則
    - 命題 $\mathbf{A}$ が仮定として与えられた時、命題 $\mathbf{B}$ の証明 $f_A$ が構築できたとすると、これは命題 $\mathbf{A} \rightarrow \mathbf{B}$ の証明を与える
- 除去規則 (modus ponens)
    - 命題 $\mathbf{A}$ の証明 $a$ と、命題 $\mathbf{A} \rightarrow \mathbf{B}$ の証明 $f$ があったら、命題 $\mathbf{B}$ の証明 $a f$ が構成できる

</div></div>

<br>

#### 備考

計算体系における関数の合成は、論理体系では三段論法に対応する。

$$
\mathrm{inputs}\; (\mathbf{A} \rightarrow \mathbf{B})
\rightarrow
(\mathbf{B} \rightarrow \mathbf{C})
\;\;\Longrightarrow\;\;
\mathrm{outputs}\;
(\mathbf{A} \rightarrow \mathbf{C})
\; .
$$

---

# $\mathbf{A} \times \mathbf{B}$ の解釈と規則

<div class="grid grid-cols-2 gap-4" style="font-size: 12pt;"><div>

#### 計算体系

- 解釈 (対)：型 $\mathbf{A}$ と型 $\mathbf{B}$ の対
- 導入規則 (対の構成)
    - 型 $\mathbf{A}$ のオブジェクト $a$ および型 $\mathbf{B}$ のオブジェクト $b$ から、型 $\mathbf{A} \times \mathbf{B}$ のオブジェクト $(a,b)$ を構成できる
- 除去規則 (射影)
    - 型 $\mathbf{A} \times \mathbf{B}$ のオブジェクト $p$ が与えられたら、片方の型 $\mathbf{A}$ ($\mathbf{B}$) のオブジェクト $p.1$ ($p.2$) を取り出せる

</div><div>

#### 論理体系

- 解釈 (連言)：命題 $\mathbf{A}$ かつ命題 $\mathbf{B}$ が同時に成立
    - $\times$ は $\wedge$ とも書かれる
- 導入規則
    - 命題 $\mathbf{A}$ の証明 $a$ および命題 $\mathbf{B}$ の証明 $b$ から、命題 $\mathbf{A} \times \mathbf{B}$ の証明 $(a,b)$ を構成できる
- 除去規則
    - 命題 $\mathbf{A} \times \mathbf{B}$ の証明 $p$ が与えられたら、片方の命題 $\mathbf{A}$ ($\mathbf{B}$) の証明 $p.1$ ($p.2$) を取り出せる

</div></div>

<br>

#### 備考

$n$ 個の積に拡張も容易である。特に計算体系における結合法則なども明らかと言える：

$$
\mathbf{A} \times \mathbf{B} \times \mathbf{C}
\;\;\Longleftrightarrow\;\;
(\mathbf{A} \times \mathbf{B}) \times \mathbf{C}
\;\;\Longleftrightarrow\;\;
\mathbf{A} \times (\mathbf{B} \times \mathbf{C})
\; .
$$

---

# $\mathbf{A} + \mathbf{B}$ の解釈と規則

<div class="grid grid-cols-2 gap-4" style="font-size: 12pt;"><div>

#### 計算体系

- 解釈 (tagged Union)：1つ目の型 $\mathbf{A}$ か2つ目の型 $\mathbf{B}$ かどちらか
- 導入規則 (tagged Unionの構成)
    - 型 $\mathbf{A}$ (あるいは$\mathbf{B}$) のオブジェクト $a$ (あるいは$b$) を型 $\mathbf{A} + \mathbf{B}$ のオブジェクトとして埋め込める（解釈できる）
- 除去規則 (case文)
    - 型 $\mathbf{A} + \mathbf{B}$ のオブジェクト $c$ 、型 $\mathbf{A} \rightarrow \mathbf{C}$ の関数 $f$ および型 $\mathbf{B} \rightarrow \mathbf{C}$ の関数 $g$ が与えられたら、場合分けによって型 $\mathbf{C}$ のオブジェクトを($c:\mathbf{A}$なら$f(c)$、$c:\mathbf{B}$なら$g(c)$として)構成できる

</div><div>

#### 論理体系

- 解釈 (選言)：命題 $\mathbf{A}$ または命題 $\mathbf{B}$ の少なくとも一方は成立（両方正しいも含む）
    - $+$ は $\vee$ とも書かれる
- 導入規則
    - 命題 $\mathbf{A}$ (あるいは$\mathbf{B}$) の証明 $a$ (あるいは$b$) を命題 $\mathbf{A} + \mathbf{B}$ の証明として埋め込める（解釈できる）
- 除去規則
    - 命題 $\mathbf{A} + \mathbf{B}$ の証明 $c$ 、命題 $\mathbf{A} \rightarrow \mathbf{C}$ の証明 $f$ および命題 $\mathbf{B} \rightarrow \mathbf{C}$ の証明 $g$ が与えられたら、場合分けによって命題 $\mathbf{C}$ の証明を($c:\mathbf{A}$なら$f(c)$、$c:\mathbf{B}$なら$g(c)$として)構成できる

</div></div>

---

#### 例

ここまで来ると、論理学における様々な論理式が計算体系として解釈できる。例えば、

$$
((\mathbf{A} \vee \mathbf{B}) \rightarrow \mathbf{C})
\rightarrow
(\mathbf{A} \rightarrow \mathbf{C})
\wedge
(\mathbf{B} \rightarrow \mathbf{C})
$$

という論理式を考える。これは

$$
\mathrm{input}\;
((\mathbf{A}+\mathbf{B}) \rightarrow \mathbf{C})
\;\;\Longrightarrow\;\;
\mathrm{output}\;
(\mathbf{A} \rightarrow \mathbf{C})
\times
(\mathbf{B} \rightarrow \mathbf{C})
\; .
$$

とも書き直せる事に注意する。

すると、これは $\mathbf{A} + \mathbf{B}$ から $\mathbf{C}$ への関数を与えると、
$\mathbf{A} \rightarrow \mathbf{C}$
および
$\mathbf{B} \rightarrow \mathbf{C}$
の型を持つ関数が両方手に入る事を示唆する。

実際、

$$
f: (\mathbf{A} + \mathbf{B}) \rightarrow \mathbf{C}
$$

が与えられた時に、
引数をそれぞれの型に制限した関数 $f_\mathbf{A}, f_\mathbf{B}$ は所望の関数となっている。

---

全称型 $\forall \mathbf{T}. \mathbf{F(T)}$
や
依存型 $\Pi_{a: \mathbf{A}} \mathbf{B}(a)$
も同様であるが、今回は省略する。
(特に全称型および型演算子は、次に紹介する論理体系側の理解に、発表者の自信がない、という理由もある)

---

# Curry-Howard対応

これまで見たような、計算体系と論理体系の間の対応は、 **Curry-Howard対応 (correspondence / isomorphism)** などと呼ばれる。
より詳細には(最小命題論理の場合)以下の対応が存在する：

<center>

<table class="table" style="width:60%; text-align: center;">
  <tr><th style="text-align: center;">

<font color="green">

型システム

</font>

  </th><th style="text-align: center; font-color: blue;">

<font color="blue">

論理体系

</font>

  </th></tr>


  <tr><td>

型 $\mathbf{T}$

  </td><td>

論理式 $\varphi$

  </td></tr>


  <tr><td>

基本型 $\mathbf{A}$

  </td><td>

原子命題 $\mathbf{P}$

  </td></tr>


  <tr><td>

関数 $\rightarrow$

  </td><td>

含意演算子 $\rightarrow$

  </td></tr>


  <tr><td>

型環境 $\Gamma$

  </td><td>

公理系・仮定 $\Gamma$

  </td></tr>


  <tr><td>

型付け可能 $\Gamma \vdash t: \mathbf{T}$<br>
型$\mathbf{T}$を持つ項$t$の存在

  </td><td>

(自然演繹における)証明可能 $\Gamma \vdash \varphi$

  </td></tr>


  <tr><td>

単純型付き$\lambda$計算 $\lambda_{\rightarrow}$

  </td><td>

含意$\rightarrow$のみを持つ命題論理

  </td></tr>


  <tr><td>

型導出図

  </td><td>

証明図

  </td></tr>


  <tr><td>

型付け規則

<div style="text-align: left; margin-left: 50px;">

- 変数導入 [T-VAR]
- $\lambda$-抽象 [T-ABS]
- 関数適用 [T-APP]

</div>

  </td><td>

推論規則

<div style="text-align: left; margin-left: 50px;">

- 仮定公理 [AA]
- 含意導入規則 [$\rightarrow$ I]
- 含意除去規則 [$\rightarrow$ E]

</div>

  </td></tr>

</table>

</center>

---

# 参考：$L$-cube

これまでは $\lambda_\to$ を念頭に置いた Curry-Howard対応を考えてきたが、
この対応を先の8つの計算体系 $\lambda$-cubeへ拡張し、論理体系側の対応物を同定する事もできる。
これは **$L$-cube** とも呼ばれ、やはり8つの関連する論理体系が存在する。

<div align="center">

<style>
table th {
    font-weight: bold;
    text-align: center;
    line-height: 1;
}
</style>

<table class="table" style="width: 90%; text-align: center;">
<tr>
  <th style="width: 20%;">

計算体系 ($\lambda$-cube)

  </th><th style="width: 35%;">

計算体系名称

  </th><th style="width: 35%;">

論理体系名称 ($L$-cube)

  </th>
</tr>



<tr><td>

$\lambda_{\rightarrow}$

</td><td>

単純型付き$\lambda$計算

</td><td>

(直観主義)命題論理

</td></tr>



<tr><td>

$\lambda_{2}$

</td><td>

System $F$

</td><td>

2階命題論理

</td></tr>



<tr><td>

$\lambda_{\underline{\omega}}$

</td><td>

</td><td>

弱高階命題論理

</td></tr>



<tr><td>

$\lambda_{\omega}$

</td><td>

system $F_{\omega}$

</td><td>

高階命題論理

</td></tr>



<tr><td>

$\lambda_{P}$

</td><td>

Logical Framework

</td><td>

(直観主義) 述語論理

</td></tr>



<tr><td>

$\lambda_{P2}$

</td><td>

</td><td>

2階述語論理

</td></tr>



<tr><td>

$\lambda_{P\underline{\omega}}$

</td><td>

</td><td>

弱高階述語論理

</td></tr>



<tr><td>

$\lambda C = \lambda_{P \omega}$

</td><td>

Calculus of Constructions (CoC)

</td><td>

高階述語論理

</td></tr>

</table>

</div>



---

# Further extensions: continuation v.s. classical logic

通常の論理学において、 **否定 (negation)** という論理演算子が存在するが、これの対応物は、実行すると値を返さない関数である。
これは $\neg \mathbf{A} = \mathbf{A} \rightarrow \bot$ という関係式と、矛盾記号 $\bot$ が値の存在しない $\mathrm{Bottom}$ 型 ($\mathrm{Never}$型という名称で導入される事が多い) である事実から理解できる。

ただし重要な点として、二重否定 $\neg \neg \mathbf{A} \rightarrow \mathbf{A}$ や排中律 $\mathbf{A} \wedge \neg \mathbf{A}$ といった論理式に対応する型に属するオブジェクトは、 $\lambda_\to$ の中で一般に構築できない。これはCurry-Howard対応の破綻を意味するわけではなく、これらの命題は、論理体系側でも実際に証明できない命題である。
この($\lambda_\to$に対応する)論理体系は **直観主義論理 (intuitionistic logic)** と呼ばれ、二重否定や排中律が証明可能な論理体系は **古典論理 (classical logic)** と呼ばれている。

そして面白い事に、古典論理にCurry-Howard対応する計算体系は存在する。
それを実現するのが、 **継続** や **call/cc** と呼ばれている、計算を**中断**する機能であるが、詳細は割愛する。

---

# Further extensions: region v.s. linear logic

また **線形論理 (linear logic)** と呼ばれる論理体系が存在するが、計算体系側の対応物は、 **線形型システム (linear type system)** と呼ばれる。
これはオブジェクトの呼び出し回数に制限、具体的にはぴったし1回だけ呼び出す、という制約条件を設けるものである。
特に、メモリの確保や開放、排他ロックといった操作は二重操作が基本許されないため、このような制約条件で実現する事ができる。
これを **リージョン (region)** と呼ばれる所有権およびスコープの概念に昇華し、機能として取り入れる事に成功した実践的な言語がRustである。
(なおaffine型システムでは、という主張もあるが、詳しくないので分からない...)

<br>

このようにCurry-Howard対応によって、計算体系と論理体系との間には密接な結びつきがあり、
一方の進展が、他方の発展に影響を与えてきた歴史がある。
ちなみに、論理体系に対応物が存在しないような計算体系は存在するので、本対応は万能ではない。(なお逆の主張はよく知らない)

いずれにせよ、プログラミングという行為は、数学の証明を考える事と実質同じであり、Coqにおいては、
定理を証明する事でプログラミングを行う事が可能なのである。

---

# Coqデモ？

- 証明を行うと、関数が得られることを簡単に見る：

---

# Summary

- オブジェクトの世界とは別に型の世界が拡がっており、関数 (型演算子)が住んでいる
    - 型の世界にも『型」が存在する (カインド)
- オブジェクトの世界と型の世界をつなぐ事もできる
    - ←：パラメトリック多相型
    - →：依存型
- これらを整理して立方体上に配置したものが $\lambda$-cube
    - その最も"高い"頂点に住んでいるのがCalculus of ConstructionsでCoqのベース(のベース)となる計算体系
- 計算・プログラミングという行為は、数学の証明を構築する行為と同じ (Curry-Howard対応)
    - 型が命題に対応する
- この対応は $\lambda$-cubeや、より広いクラスの計算体系・論理体系でも成立し、プログラミング言語開発に影響を与えている

---

# Future topics

今回詳しく話せなかったが、次回以降の関連テーマとして、もしも需要があれば...

- Curry-Howard対応の詳細（計算する事＝証明する事）
- 線形型とリージョン（勉強中。Rustの所有権に関する型理論）
- 対話型定理証明の詳細（Coqでどうやってプログラミングするの？）
- Pure Type System (今回の話の延長だが、マニアック過ぎて需要ないかも)
- 部分型（TypeScriptなどの背景）
- 直観型理論（最近勉強中。ほぼ数学...）
- $\lambda$計算の詳細（ほぼ数学ですが、手を動かせるので楽しい）

---

# Main references

上から順に役立たった、と思われる。

- Benjamin.C.Pierce, "Types and Programming Langauges", MIT Press 2002
  - メインはこれ(8,9,11,23,29,30章あたり)。勉強会開催中。(connpass参照)
- Henk. Barendregt, "An Introduction to Generalized Type Systems", Journal of Functional Programming, Volume 1, Issue 2, April 1991, pp.125-154.
  - TaPL呼んだら次のこれ読むべき。「型システムチョットデキル」になれる。
- 照井 一成『コンピュータは数学者になれるのか？』、青土社 2015
  - ローストビーフと見せかけて、中身はビフテキ。噛めば噛むほど味が出る。コンピュータ科学に興味ある人なら全員読んでほしいくらいの名著。<font size="2">(注：細部にこだわると読めないので、絶対に細部にこだわってはいけない。)</font>
- Nederpelt and Geuvers, "Type Theory and Formal Proof", Cambridge 2014
  - 前半1/3は$lambda$-cubeまでの導入が完結にまとまっている。
- 萩谷 昌巳・西崎 真也『論理と計算の仕組み』、岩波書店 2007
  - CH対応の理解に必要な、命題論理、述語論理の基本的な事項が一通りまとまっている。ただ肝心のCH対応はほとんど解説してくれていない。

---

- 萩原 学・Reynald Affeld, 『Coq/SSReflect/MathCompによる定理証明:フリーソフトではじめる数学の形式化』, 森北出版 2018
  - Coqに関するほぼ唯一(?)の和書。読めばCoqに関する基本的な(?)事が身につくと思われる。
- Jacques Garrigueさんの講義録, https://www.math.nagoya-u.ac.jp/~garrigue/lecture/
  - Coqについて有益な資料が幾つか。多分、ここに置かれていないものもある気がするので、ググる必要あり。



---

# Sub references

今回直接参照はしていない。

- MacCormick (長尾訳) 『計算できるもの、計算できないもの　実践的アプローチによる計算理論入門』、Oreilly社 2020
- 龍田 真『型理論』、近代科学社 1992
- 五十嵐 淳『プログラミング言語の基礎概念』、サイエンス社 2011
- 小林直樹・住井英二郎『プログラミング意味論の基礎』、サイエンス社 2022
- 大堀 淳『新装版 プログラミング言語の基礎理論』、共立出版 2019
- 小野 寛晰『情報科学における論理』、日本評論社 1994
- 高野 祐輝『ゼロから学ぶRust』、講談社 2022

今回は、計算論・意味論の話そのものはしなかったので、そちらがメインの本は紹介を割愛する。
