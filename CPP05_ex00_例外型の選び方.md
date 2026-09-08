# ex00補足 — 標準例外型と独自例外型

## 1. 課題で必要な型

ex00では、課題書が次の二つの例外型を指定している。

- `Bureaucrat::GradeTooHighException`
- `Bureaucrat::GradeTooLowException`

そのため、`std::invalid_argument`や`std::out_of_range`をそのまま送出するだけでは、課題の要求を満たさない。必要なのは、課題指定の名前を持つ独自例外クラスを作ることである。

独自例外クラスの基底クラスには、課題書の例に合わせて`std::exception`を使える。

```cpp
class GradeTooHighException : public std::exception
{
public:
    virtual const char *what() const throw();
};
```

## 2. 標準例外型を基底にする選択

実務経験がある場合、エラーの意味に合わせて標準例外型を基底に選ぶこともある。

```text
std::exception
    └── std::logic_error
            ├── std::invalid_argument
            └── std::out_of_range
```

### `std::invalid_argument`

呼び出し側が、関数やコンストラクタの引数として不正な内容を渡した場合に向いている。

例:

- 数値であるべき場所に不正な形式の文字列を渡した
- 許可されていない種類の値を渡した
- 値の内容が引数の前提条件に合わない

### `std::out_of_range`

値の意味や型は妥当だが、許容された範囲の外にある場合に向いている。

例:

- 等級に`0`や`151`を渡した
- 配列や文字列の範囲外を指定した

Bureaucratの等級は`1`から`150`までなので、値の範囲外という側面を強調するなら`std::out_of_range`が自然である。一方、等級を外部入力の不正な引数として扱う設計では、`std::invalid_argument`も考えられる。

## 3. 基底クラスを差し替えるときの注意

`std::exception`を`std::out_of_range`へ名前だけ差し替えれば終わり、とは限らない。`std::out_of_range`や`std::invalid_argument`は、メッセージを受け取る基底クラスのコンストラクタを初期化する必要がある。

```cpp
#include <stdexcept>

class GradeTooHighException : public std::out_of_range
{
public:
    GradeTooHighException()
        : std::out_of_range("Grade is too high")
    {
    }
};
```

この場合は標準例外クラスの`what()`を継承できるため、自分で`what()`を定義しなくてもよい。

```text
GradeTooHighException
    -> std::out_of_range
        -> std::logic_error
            -> std::exception
```

したがって、課題書の次の捕捉条件にも適合する。

```cpp
catch (std::exception &e)
```

必要なら、より具体的な基底型でも捕捉できる。

```cpp
catch (std::out_of_range &e)
```

## 4. `<stdexcept>`は制限に反しない

`<stdexcept>`はC++標準ライブラリの標準ヘッダであり、外部ライブラリではない。ex00の`Forbidden: None`にも反しない。また、C++98にも`std::logic_error`、`std::invalid_argument`、`std::out_of_range`は存在する。


## 5. ex00での現実的な選択

提出実装としては、次の二つが成立する。

| 方式 | 特徴 |
| --- | --- |
| `std::exception`を直接継承 | 課題書の例に近く、自作`what()`の練習になる |
| `std::logic_error`などを継承 | 標準例外の分類とメッセージを利用できる |

課題が要求しているのは、独自の例外名を持ち、`std::exception &`で捕捉できることである。したがって、`std::exception`の直接継承が必須という意味ではない。ただし、学習内容を明確にしやすく、課題書の例にも近いので、直接継承が最小で分かりやすい実装である。

標準例外型を基底にする場合でも、次の点は維持する。

- 例外クラスを`Bureaucrat`の中に置く
- クラス名を`GradeTooHighException`と`GradeTooLowException`にする
- 等級の上下に対応する例外を正しく投げ分ける
- `catch (std::exception &e)`で捕捉できる継承関係にする
