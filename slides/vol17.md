## &lt;meta name="description"
## "content"="落ち穂拾い"/&gt;

いなむのみたま

---

## std::is_aggregate&lt;T&gt;

型`T`がaggregateかどうかを判定するメタ関数、以上！

**Preconditions**

`std::remove_all_extents_t<T>`が完全型であるか、
（cv修飾された）void型であること。

---

## ところで……
### aggregateって知ってる？


---

## aggregateの定義

#### 型Tが集成体であるための条件は以下である：

1. ユーザー定義されたコンストラクタ、explicitなコンストラクタ、継承コンストラクタを持たない

2. private／protectedな非静的メンバ変数を持たない

3. 仮想関数を持たない

4. 仮想基本クラス、private／protected基本クラスを持たない

---

## aggregateだと
### 集成体初期化できる

```cpp
struct Point{
  int x,y;
};

Point a = {1, 2}; // aggregate-initialization
```

---

## 集成体初期化の罠

**ユーザー定義**されたコンストラクタ、explicitなコンストラクタ、継承コンストラクタを持たない

...定義？

```cpp
struct WTF{
  WTF() = delete; // 定義ではない
};
WTF a = {}; // well-formed in C++11/14/17
```

---

## 集成体初期化の罠

#### C++20
~~ユーザー定義されたコンストラクタ、explicitなコンストラクタ、継承コンストラクタを持たない~~

**ユーザ宣言**されたコンストラクタまたは継承されたコンストラクタを持たない。

```cpp
struct Point{
  WTF() = delete; // わたくし、宣言です
};
WTF a = {}; // ill-formed in C++20
```

---

## designated initialization

#### C++20

```cpp
struct Point{
  int x,y,z;
};
// C++20 feature: designated initialization
Point a = {.x=1, .y=2, .z=0};
```

GNU拡張ではしれっと使えていた。

---

## std::bool_constant

```cpp
namespace std {
template < bool B >
using bool_constant = integral_constant<bool, B>;
}
```

---

## std::bool_constant
### example

いい例が思いつかないが・・・
クラスを場合分けで2つ書いて、std::ture_typeとstd::falseを別々に継承するの面倒なので、直接条件を羅列したりするのに使えそう。

（ほんまに使うか？）


```cpp
template < class... Pred >
struct is_hoge
    : std::bool_constant<(Pred::value && ...)> {};
```

---

## Logical operation metafunctions

```cpp
std::cunjunction
std::disjunction
std::negation

std::cunjunction_v
std::disjunction_v
std::negation_v
```

---

## わりと専門用語だよね

#### conjunction=連言（れんげん）
#### つまり∧（論理積）
#### disjunction=選言（せんげん）
#### つまり∨（論理積）
#### negation=否定（ひてい）
#### つまり¬（論理否定）

---

## std::conjuncion
`B...`の論理積を行うメタ関数。

一般にはメタ述語`A`と`B`（あるいはもっとたくさん）のすべてを満たすか判定したい場合などに使うはず。
```cpp
namespace std{
template <class... B>
struct conjunction;

template <class... B>
inline constexpr bool
    conjunction_v = conjunction<B...>::value;
}
```

---

## std::conjuncion
### example

いい例が思いつかないが・・・
パラメータパックの型がすべて同じことをSFINAEしたいときとかに使えそう。

```cpp
template<typename T, typename... P>
std::enable_if_t<std::conjunction_v<std::is_same<T, P>...>>
func(T head, P... tail) {
    // ...
}
```

---

## std::conjuncion
### in detail

- テンプレート引数が空のとき、`std::true_type`を基底クラスに持つ
- B_1, ..., B_nというテンプレート引数を持つとき、bool(B_i::value)がはじめて`false`になる型を基底クラスに持つ
- B_{n-1}::valueが`true`のとき、B_nを基底クラスに持つ

---

## std::conjuncion
### in detail

```cpp
static_assert(
    std::disjunction<std::integral_constant<int, 3>,    
                    std::integral_constant<int, 4>
            >::value == 4); // OK
```

---

## std::conjuncion
### Tips

- B_i::valueがはじめて`false`となるとき
- j>iであるようなB_jはそもそも`value`を持たなくて良い

```cpp
static_assert(
    std::disjunction<std::false_type,    
                     nullptr
            >::value == false); // OK
```

---

## std::disjuncion
`B...`の論理和を行うメタ関数。

一般にはメタ述語`A`と`B`（あるいはもっとたくさん）のどれかを満たすか判定したい場合などに使うはず。

```cpp
namespace std{
template <class... B>
struct disjuncion;

template <class... B>
inline constexpr bool
    disjuncion_v = disjuncion<B...>::value;
}
```

---

## std::disjunction
### example

いい例が思いつかない、つらい・・・
型`T`のデフォルト値をなんやかんやする例。

```cpp
template < class T >
inline std::enable_if_t<std::disjunction_v<
    std::is_default_constructible<T>,
    std::is_aggregate<T>>, T>
default_v = []{
    if constexpr (std::is_aggregate_v<T>) { return T{}; }
    else { return T(); }
}();
```

---

## std::disjunction
### in detail

- テンプレート引数が空のとき、`std::false_type`を基底クラスに持つ
- B_1, ..., B_nというテンプレート引数を持つとき、bool(B_i::value)がはじめて`true`になる型を基底クラスに持つ
- B_{n-1}::valueが`false`のとき、B_nを基底クラスに持つ

---

## std::disjunction
### in detail

```cpp
static_assert(
    std::disjunction<std::integral_constant<int, 4>,    
                    std::integral_constant<int, 3>
            >::value == 4); // OK
```

---


## std::disjunction
### Tips

- B_i::valueがはじめて`true`となるとき
- j>iであるようなB_jはそもそも`value`を持たなくて良い

```cpp
static_assert(
    std::disjunction<std::integral_constant<int, 4>,    
                    nullptr
            >::value == 4); // OK
```

---

## std::negation

あるメタ述語`B`の論理否定を行うメタ関数。
`std::bool_constant<!bool(B::value)>`を基底クラスに持つ。
メタ述語の否定のメタ述語を作りたいときに。

```cpp
namespace std{
template <class B>
struct negation: bool_constant<!bool(B::value)>{};

template <class B>
inline constexpr bool
    negation_v = negation<B...>::value;
}
```

---

## std::negation
### example

`std::conjunction`, `std::disjunction`のパラメータパック内でメタ述語の否定をしたいときに使うのは想定されるよね。

そもそも`std::is_same_v<T,U>`とか書いてるときには、
普通に`!std::is_same_v<T,U>`と書けばいい。

```cpp
template <typename... Ts>
std::enable_if_t<std::conjunction_v<
        std::negation<std::is_integral<Ts>>...>>
func(Ts... xs) { // Ts... はすべて整数ではない型
  // ...
}
```

---

## is_swappableの一族

```cpp
template< class T, class U >
struct is_swappable_with;

template< class T >
struct is_swappable;

template< class T, class U >
struct is_nothrow_swappable_with;

template< class T >
struct is_nothrow_swappable;
```

~~規格書のTable 42 — Type property predicatesでひときわ長い説明が書いてあるので印象深い。~~

---

## is_swappable_with&lt;T, U&gt;

#### `using std::swap`されたという条件下かつ、
#### 未評価な文脈で
#### `swap(declval<T>(),declval<U>())`と
#### `swap(declval<U>(),declval<T>())`が
#### ともにwell-formedな式となるかを判定する。


**Preconditions**
型TとUが、完全型であること。
もしくは（cv修飾された）voidか、要素数不明の配列型であること。

---

## § 23.15.4.3 Type properties/C++17
### in Table 42 — Type property predicates

Access checking is performed as if in a context unrelated to T and U.
Only the validity of the immediate context of the swap expressions is considered.
[ Note: The compilation of the expressions can result in side effects such as the instantiation of class template specializations and function template specializations, the generation of implicitly-defined functions, and so on.

Such side effects are not in the “immediate context” and
can result in the program being ill-formed. — end note ]


---

## cpprefjpより引用

このメタ関数はTとUについてのswap関数の直接のコンテキストの妥当性（そのシグネチャで有効なswapがあるかどうか）のみをチェックする。

そのため、結果がtrueとなったとしてもswap関数の呼び出しができることは保証されない（その他の要因によりコンパイルエラーが発生する可能性がある）。

---

## 別の言い方でもう一度

シグネチャだけ見ると呼び出し可能だけども、（実際には）`T`や`U`の使用によって、なんらかのテンプレートの実体化等やら暗黙に定義されたの特殊メンバの生成なんやらをもろもろ起こした結果、それがなんらかのエラーとなるかもしれない。

そのような場合でも`is_swappable_with`はしっかりと実体化し、`value`メンバ変数が`true`となるかもしれないよ。

---

## is_swappable&lt;T&gt;

`T`が参照可能な型であれば、`value`メンバ変数は
`std::is_swappable_with<T&, T&>::value`と同じ。
それ以外であれば`false`。

---

## 最後
### is_nothrow_swappable&lt;T&gt;
### is_nothrow_swappable&lt;T&gt;

### 無例外保証検査がついてお得！
### オペレーターを増やしてお待ちしております

---

# C++20はよ
