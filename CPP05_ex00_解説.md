# ex00 解説 — Bureaucrat と例外

提出物: `Makefile`, `main.cpp`, `Bureaucrat.hpp`（または `.h`）, `Bureaucrat.cpp`  
課題の核: 等級の向き、コンストラクタでの例外、ネストした例外クラス、const 名前と OCF。

完成した `.cpp` は載せない。疑似コードとスケルトンに止める。

## 全体資料の関連節

先に次の節を読むと、この解説で扱う概念の位置付けを確認できる。

- [4.1 Cのエラー処理との違い](./CPP05_テーマと発展.md#exceptions-c): 戻り値による失敗通知と例外による失敗通知の違い
- [4.2 コンストラクタから送出された例外](./CPP05_テーマと発展.md#exceptions-constructor): 構築が完了しなかったオブジェクトと、その基底クラス・データメンバの扱い
- [4.3 C++98のthrow()例外仕様](./CPP05_テーマと発展.md#exceptions-specification): `what()`の末尾にある`throw()`の約束
- [4.4 発展: 例外安全保証](./CPP05_テーマと発展.md#exceptions-safety): 失敗しても`_grade`を変更前の状態に保つstrong保証
- [5.2 constデータメンバとOrthodox Canonical Form](./CPP05_テーマと発展.md#design-const): constな`_name`とコピー代入演算子の関係

---

## 1. 等級は数直線ではない

数値の大小と権限の高低が逆である。

```
権限が高い ←  1  2  3  ...  149  150  → 権限が低い
              最高                    最低
```

| 操作 | 権限 | 数値 |
|------|------|------|
| increment | 上がる | 減る（3 → 2） |
| decrement | 下がる | 増える（3 → 4） |
| 等級 0 で生成 | 高すぎ | `GradeTooHighException` |
| 等級 151 で生成 | 低すぎ | `GradeTooLowException` |

`if (grade < 1)` と `GradeTooHighException` が対になる。  
「小さい数字なのに High」はバグではない。

### いまの誤解の材料

出典: https://github.com/tomtomvx/cpp05-09 （2026-09-08 時点の `05/ex00/Bureaucat.cpp`）

```cpp
void Bureaucrat::checkGrade(int const grade) const
{
	if (grade < 1)
		throw Bureaucrat::GradeTooHighException();
	else if (grade > 150)
		throw Bureaucrat::GradeTooHighException();  // ここが Low ではない
}
```

151 も TooHigh になる。`GradeTooLowException` クラスを宣言しておきながら使っていない。  
自分のテストでは0と151の両方を試す。同じ例外しか出ない実装では、二つの例外クラスを要求する課題の仕様を満たさない。

---

## 2. コンストラクタの失敗を例外で伝える

```cpp
try {
    Bureaucrat bureaucrat("Alice", 0);
} catch (std::exception & e) {
    std::cout << e.what() << std::endl;
}
```

課題書の General rules は、特に指定がない限り出力メッセージを標準出力へ出すと定めている。`subject_cpp05.pdf` 全体を確認しても、標準エラー（`std::cerr`）への出力指定はない。例外やエラーメッセージも、課題書が別ストリームを指定していない限り `std::cout` に出す。

ただし、実務上の落とし穴としては指摘は妥当だ。`catch` で `std::cerr` に出すのは慣習としてよくある。

流れ:

1. `throw 例外オブジェクト;`で例外を送出し、現在の処理を中断する
2. スタックを巻き戻す（スタックアンワインド）
3. 途中で構築済みの自動オブジェクトを破棄する
4. 型が一致する`catch`で例外を捕捉する
5. `catch`で処理できれば、その後の処理を継続できる

コンストラクタが例外を送出すると、そのコンストラクタは完了せず、対象のオブジェクトは作られない。オブジェクト全体のデストラクタは呼ばれないが、構築済みの基底クラスとデータメンバは破棄される。呼び出し側は`Bureaucrat b("x", 0);`の`b`を使えない。

一致する`catch`が呼び出し履歴上に存在しなければ、`std::terminate()`が呼ばれ、プログラムは異常終了する。詳細は[4.2 コンストラクタから送出された例外](./CPP05_テーマと発展.md#exceptions-constructor)を参照する。

IRC のエラー処理と比較するなら、B 層がエラーリプライを返すのは「オブジェクトは既にある」前提である。  
CPP05 のコンストラクタ例外は「オブジェクトを作ること自体を拒否する」。

<details>
<summary>ここでの「スタック」とは（クリックで表示）</summary>

[Chromeの検索AIとの対話](https://share.google/aimode/WlZIr9FP3zcslrn4S)の結論のみ抜粋。

## ここでの「スタック」とは、プログラムが今どの関数やブロックを実行しているかという「実行階層（コールスタック）」と、そこで使うローカル変数を管理するメモリ領域のことです。
ご指定いただいた「階層のイメージ」を中心に、提示されたコードの「流れ」で何が起きているのかを最も分かりやすく書き直しました。
------------------------------
## コールスタック（実行階層）で見る例外処理の流れ
try と catch は完全に独立した別々のスコープ（ブロック）です。これらがコールスタックという「実行階層」のなかでどう動くのか、順を追って見ていきましょう。
## 階層 1：親の階層（main 関数など）

* ここに try-catch というセット（例外ハンドラ）が配置されます。
* プログラムはまず、try ブロックという「一歩中に入った階層」へ進みます。

## 階層 2：try ブロックの階層（一歩中に入る）

* スタック領域に bureaucrat のためのメモリが積まれ、コンストラクタが動き出します。
* 引数の 0 が不正なため、コンストラクタの内部で throw（例外送出） が発生し、現在の処理が中断します。

## 階層 3：スタックの巻き戻し（スタックアンワインド）

* 例外が発生したため、プログラムは今いる try ブロックの階層を終了（崩壊）させ、一歩外の「親の階層」へと強制的に戻っていきます。
* 【途中で構築済みの自動オブジェクトを破棄】
このとき、bureaucrat 自体はコンストラクタが完了していない（オブジェクトが未完成）ため全体のデストラクタは呼ばれません。しかし、コンストラクタの途中ですでに初期化が終わっていた一部のデータメンバ（名前の文字列など）があれば、この階層が崩壊する際に自動で安全に破棄されます。

## 階層 4：catch ブロックの階層（隣の部屋に入るイメージ）

* 親の階層に戻ると、「この try に対応する catch はこれだ」と案内されます。
* 型（std::exception）が一致するため、プログラムはそのまま catch ブロックという「隣の部屋」に入り、例外を捕捉してエラーメッセージを出力します。
* catch 内でエラーを適切に処理できれば、プログラムはクラッシュすることなく、その後の処理を継続できます。
* なお、呼び出し側は try スコープの崩壊とともに消えた（そもそも完成しなかった）bureaucrat を使うことはできません。

------------------------------
## まとめ
ここでのスタックとは、「実行中のコードの階層（コールスタック）」であり、スタックを巻き戻すとは「例外が起きた部屋（try）から親の階層へ戻りつつ、その部屋に置いてあったメモリ（構築途中のメンバなど）を安全に片付けて、次の部屋（catch）へ移動するプロセス」を指します。
この「階層と部屋のイメージ」で、提示された仕様（データメンバの破棄やオブジェクトが使えない理由）のつながりはスッキリ整理できましたでしょうか？
もしさらに深掘りしたい部分があれば、以下からお選びください：

* catch ブロックの部屋に渡される 例外オブジェクト（e）はどの階層にいるのか
* もし try-catch を書かなかったら、スタックはどこまで巻き戻ってクラッシュするのか
* 関数を何個もまたいで例外が投げられた場合の 階層の遡り方
</details>



---

## 3. ネストした例外クラス

課題は `Bureaucrat::GradeTooHighException` という名前を要求する。  
これは内側クラス（ネストクラス）にする。

```cpp
class Bureaucrat {
public:
    class GradeTooHighException : public std::exception {
    public:
        virtual const char *what() const throw();
    };
    class GradeTooLowException : public std::exception {
    public:
        virtual const char *what() const throw();
    };
    // ...
};
```

### なぜネストするか

- 名前が `Bureaucrat::` で修飾され、Form 側で同名例外を別に持てる
- 課題書の`catch`例は`std::exception &`。継承していれば捕捉できる
- 例外クラスは OCF 不要

### `what()` の実装場所

ヘッダに宣言だけを書き、`.cpp`に定義を書かなければ、`what()`や例外クラスの仮想関数表を必要とするコードをリンクするときに未定義シンボルとなる。これは実行時に例外を送出した瞬間のエラーではなく、ビルドのリンク段階のエラーである。

```cpp
const char *Bureaucrat::GradeTooHighException::what() const throw()
{
    return "Grade is too high";
}
```

`what() const throw()`は関数外へ例外を送出しないと宣言している。関数内で`std::string`や`std::ostringstream`へ動的に書き込むと、メモリ確保の失敗によって例外が発生する可能性がある。固定メッセージなら、文字列リテラルを返す構成が単純であり、`throw()`の約束も守りやすい。

### `std::exception` と `std::logic_error`

| 基底 | 課題の例 `catch (std::exception &)` | 意味の正確さ |
|------|-------------------------------------|--------------|
| `std::exception` | 捕捉できる | 汎用 |
| `std::logic_error` | 間接継承なので捕捉できる | 「プログラマが渡した値が論理的におかしい」に近い |

課題が要求しているのは、`catch (std::exception &e)`で自作例外を捕捉できることである。`std::exception`の直接継承でも、`std::logic_error`を介した間接継承でも、この技術要件を満たす。

ex00では`std::exception`を直接継承し、`what()`を定義する構成が最小である。ただし、直接継承だけが課題に適合するという意味ではない。

---

## 4. const 名前と代入演算子

スケルトン:

```cpp
class Bureaucrat {
public:
    Bureaucrat();
    Bureaucrat(const std::string &name, int grade);
    Bureaucrat(const Bureaucrat &other);
    Bureaucrat &operator=(const Bureaucrat &other);
    ~Bureaucrat();

    const std::string &getName() const;
    int getGrade() const;
    void incrementGrade();
    void decrementGrade();

    class GradeTooHighException : public std::exception { /* what() */ };
    class GradeTooLowException  : public std::exception { /* what() */ };

private:
    const std::string _name;
    int _grade;
};

std::ostream &operator<<(std::ostream &os, const Bureaucrat &b);
```

処理の骨格。境界判定そのものは、自分で埋める。

```cpp
Bureaucrat::Bureaucrat(const std::string &name, int grade)
    : _name(name), _grade(grade)
{
    // 最高等級の境界を超えていれば、対応する例外を送出する
    // 最低等級の境界を超えていれば、対応する例外を送出する
}

Bureaucrat::Bureaucrat(const Bureaucrat &other)
    : _name(other._name), _grade(other._grade)
{
}

Bureaucrat &Bureaucrat::operator=(const Bureaucrat &other)
{
    // 自己代入を確認し、変更可能なデータメンバだけをコピーする
    return *this;
}

void Bureaucrat::incrementGrade()
{
    // 変更可能か先に検査し、成功するときだけ等級を更新する
}

void Bureaucrat::decrementGrade()
{
    // incrementGrade()と反対向きの境界を検査する
}
```

メンバ初期化子リストはコンストラクタ本体より先に実行される。したがって、一般的な実装では`_name`と`_grade`を初期化してから、本体で等級を検査する。不正な等級ならコンストラクタが完了しないため、不正な状態の完成済みオブジェクトは呼び出し側へ渡らない。

コピーコンストラクタで`*this = other`を呼ぶ書き方は、constメンバがあると「名前は初期化子リスト、等級は代入演算子」にコピー処理が分かれる。初期化子リストで両方をコピーする方が、コピーコンストラクタの処理を一か所で確認できる。

`#define HIGHEST_GRADE 1` より、クラス内の `static const int HighestGrade = 1;` の方が型とスコープがある。  
`#define` はプリプロセスで置換されるだけで、名前空間に入らない。

### 発展: `incrementGrade()`とstrong保証

`incrementGrade()`と`decrementGrade()`は、変更後の値を先に検査し、範囲内の場合だけ`_grade`を変更する。例外を送出した場合に`_grade`が変更前のまま残るため、strong保証を満たす。

詳細は[4.4 発展: 例外安全保証](./CPP05_テーマと発展.md#exceptions-safety)を参照する。

---

## 5. 出力演算子

課題の形式（角括弧は出さない）:

```
<name>, bureaucrat grade <grade>.
```

最後のピリオドと改行の扱いに注意。`operator<<` は改行を入れず、呼び出し側が `std::endl` することが多い。課題は「この形式」とだけ書いてある。ピリオドは形式に含まれている。

---

## 6. いまの誤解の材料（ファイル名・ビルド・テスト）

同じリポジトリ `05/ex00`（2026-09-08）。

### ファイル名とクラス名が違う

| 実ファイル | クラス名 | 課題の提出名 |
|------------|----------|--------------|
| `Bureaucat.hpp` / `Bureaucat.cpp` | `Bureaucrat` | `Bureaucrat.{h,hpp}` / `Bureaucrat.cpp` |

課題はファイル名をクラス名に合わせる。`Bureaucat` のまま提出すると、必須ファイル欠落とみなされる。

### `what()` が未定義

ヘッダでは`what()`を宣言しているが、`Bureaucat.cpp`に定義がない。例外型の仮想関数表を必要とするコードがリンク対象に含まれると、`what()`を実行時に呼ぶ前でもリンクエラーになる。

### `main` が空

```cpp
int main()
{
	return (0);
}
```

課題はテスト提出を要求する。少なくとも次を試す。

- 正常生成（例: 42）
- 等級 1 と 150
- 等級 0 と 151（それぞれ別例外）
- increment / decrement の境界（1 で increment、150 で decrement）
- `operator<<`
- コピーと代入（代入後も名前が変わらないこと）

### Makefile のパターン規則

```
%.o: %.cpp %.hpp
```

`main.cpp`に対応する`main.hpp`がないため、この自作パターン規則は`main.o`には適用できない。GNU Makeの組み込み規則へフォールバックしてコンパイルできる場合はあるが、それに依存するとMakefileの動作が環境に左右される。

自作規則を`%.o: %.cpp`とし、ヘッダ依存関係は`-MMD -MP`で生成される依存ファイルに任せる構成が明確である。

---

## 7. テストの書き方（骨格）

```cpp
int main() {
    try {
        // 正常系
    } catch (std::exception &e) {
        // ここに来たら失敗
    }

    try {
        Bureaucrat bad("x", 0);
        // ここに来たら失敗
    } catch (std::exception &e) {
        // e.what() を出す
    }
    return 0;
}
```

`try`を細かく分ける。一つの`try`に正常系と異常系を混ぜると、どこで例外を送出したか分からない。

---

## 8. 比較: よくある実装の分岐

| 項目 | 提出で使う | 理解しておくこと |
|------|------------|------------------|
| 例外の基底 | `std::exception` | 課題書の例に合わせて `std::exception` を直接継承する実装が一般的である。ただし、`std::logic_error`も`std::exception`から派生しているため、課題の捕捉条件は満たす。`std::logic_error`でも捕捉できる |
| 等級定数 | `static const int` | `#define` はコンパイル前の単純なトークン置換であり、型やスコープを持たない |
| デフォルトコンストラクタ | OCF 用に必要。デフォルトの等級を 150 にする実装をよく見る | 課題本文は名前付きコンストラクタしか書いていない |
| コピー代入 | `_grade` のみ | 名前を無理にコピーしない |
| `unsigned int` 等級 | 使わなくてよい | 負数を渡すと大きな正数へ変換され、TooHighではなくTooLowと判定する可能性がある |


<details>
<summary>`logic_error`とは（クリックで表示）</summary>

std::logic_errorもstd::exceptionから派生しているため、課題の捕捉条件は満たす。
主に「プログラムの論理上、不正な値が渡された」という種類のエラーを表します。

継承関係は概ね次の通りです。
```
std::exception
    └── std::logic_error
            ├── std::invalid_argument
            ├── std::domain_error
            └── std::out_of_range
```

std::logic_errorを使う学生が少なそうなのも確かです。理由は主に次の通りです。
- 課題書の例が直接 std::exception を示している
- 自作例外クラスでは直接継承の方が構造を説明しやすい
- std::logic_errorにはメッセージを渡すコンストラクタが必要になる
- CPP05では標準例外クラスの分類より、例外クラスの自作と継承を学ぶ意図が強い

```
実務で慣れている学生がいるとしたらlogic_errorを使うこともありそうですか？またその理由は？
```
はい、実務経験のある学生なら std::logic_error を選ぶ可能性は十分あります。

理由は、等級 `0` や `151` は「実行中に偶然起きた障害」ではなく、呼び出し側がクラスの前提条件に反する値を渡したエラーだからです。
これは標準例外の分類では論理エラーに近く、std::logic_errorの意味に合います。
```cpp
class GradeTooHighException : public std::logic_error
{
public:
    GradeTooHighException()
        : std::logic_error("Grade is too high")
    {
    }
};
```
この構成には次の実務的な利点があります。

- std::logic_errorはstd::exceptionを継承している
- what()を自分で実装しなくても、基底クラスのメッセージを使える
- 例外の種類から「プログラムの使い方・前提条件の問題」と分かる
- catch (std::logic_error&)で、論理エラーだけを分けて処理できる

ただし、この課題では直接 `std::exception`を継承する実装も完全に正当です。
課題書が要求しているのは、例外名と、`catch (std::exception &e)`で捕捉できることです。`logic_error`を使う場合は、課題の最小要件を超えて、標準ライブラリの例外分類を活用していると考えるとよいです。

さらに詳しくは[標準例外型と独自例外型の選び方](./CPP05_ex00_例外型の選び方.md)を参照する。

</details>

---

次: `CPP05_ex00_事後クイズ.md`。解けたら実装に入る。ex01 には進まない。
