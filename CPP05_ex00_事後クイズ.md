# ex00 事後クイズ

> 対象: `CPP05_ex00_解説.md` を読んだあと、実装する前または実装の途中  
> 目的: 評価で口頭説明できるか、いまのコードの誤解を自分で直せるか

先に自分の回答を書く。そのあと `<details>` を開く。

---

### Q1. 等級 1 の官僚に `incrementGrade()` を呼ぶと何が起きるか。内部状態（`_grade`）はどうなっているべきか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`GradeTooHighException`を送出（`throw`）する。`_grade`は1のまま。先に減らしてから例外を送出すると`_grade`が0のまま残り、strong保証を満たさない。

詳細: [4.4 発展: 例外安全保証](./CPP05_テーマと発展.md#exceptions-safety)

</details>

---

### Q2. いまの誤解の材料。次の関数の欠陥を2つ述べよ。出典は GitHub `tomtomvx/cpp05-09` の `05/ex00`（2026-09-08）。ヘッダには`HIGHEST_GRADE`と`LOWEST_GRADE`が定義されているものとする。

```cpp
void Bureaucrat::checkGrade(int const grade) const
{
	if (grade < 1)
		throw Bureaucrat::GradeTooHighException();
	else if (grade > 150)
		throw Bureaucrat::GradeTooHighException();
}
```

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

1. `grade > 150`でも`GradeTooHighException`を送出している。ここでは`GradeTooLowException`を送出する。  
2. 定義済みの`HIGHEST_GRADE`と`LOWEST_GRADE`を使わず、マジックナンバーの1と150を直接記述している。

この関数はコンストラクタ、`incrementGrade()`、`decrementGrade()`から呼ばれるため、一つ目の誤りがすべての経路へ影響する。

</details>

---

### Q3. 同じリポジトリでクラス名は `Bureaucrat`、ファイル名は `Bureaucat.hpp` である。評価上の問題は何か。インクルード上の問題は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

評価: 必須ファイル `Bureaucrat.hpp` / `Bureaucrat.cpp` がない。  
コンパイル: `main.cpp` が `"Bureaucat.hpp"` を読めばビルドは通る。ファイル名の一致はコンパイラではなく課題規則の問題。

</details>

---

### Q4. ヘッダに`what()`の宣言があり、`.cpp`に定義がない。どの段階で、どのようなエラーとして表面化するか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

ビルドのリンク段階で、`what()`または例外クラスの仮想関数表に対する未定義シンボルとして表面化する。

これは、実行時に`what()`を呼んだ瞬間のエラーではない。例外型を構築するコードを含むオブジェクトファイルがリンクされれば、実行時にその経路を通る前でもリンクエラーになり得る。

</details>

---

### Q5. コピー代入のあと、左辺の名前と等級はどうなるべきか。次は正しいか。

```
Bureaucrat a("Alice", 50);
Bureaucrat b("Bob", 100);
a = b;
```

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

名前は Alice のまま。等級は 100。  
`_name` は const なので代入できない。これが仕様どおり。

</details>

---

### Q6. 次のうち、コンストラクタで throw したあとに正しいものはどれか。

A. 不完全な `Bureaucrat` が残り、デストラクタが後で呼ばれる  
B. そのオブジェクトは存在せず、既に構築済みのメンバだけ破棄される  
C. `new` していなければリークする

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

B。構築が完了した基底クラスとデータメンバは、構築と逆の順序で破棄される。オブジェクト全体は完成していないため、`Bureaucrat`自身のデストラクタは呼ばれない。

`new Bureaucrat(...)`のコンストラクタが失敗した場合、`new`式が確保したオブジェクト用の領域には、対応する`operator delete`が呼ばれる。ただし、コンストラクタ本体で別途確保し、RAIIで管理していないリソースはリークする可能性がある。

</details>

---

### Q7. `catch (Bureaucrat::GradeTooHighException &)` と `catch (std::exception &)` をこの順で並べる理由は何か。逆順にするとどうなるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

派生を先に書く。基底を先に書くと、TooHigh も全部 `std::exception` に吸われ、個別の catch に届かない。  
課題は基底だけで捕まえてよい。個別に分けるなら順序が必要。

</details>

---

### Q8. `#define LOWEST_GRADE 150` と `static const int LowestGrade = 150;` を、型・スコープ・デバッグの3点で比較せよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

#define: 型なし。クラスの外にも漏れる。デバッガに名前が残らないことがある。  
static const int: int。クラススコープ。メンバとして見える。C++98 では整数型ならヘッダ内初期化が可能。

</details>

---

### Q9. 自分の`main`に入れるテストを、正常系2種類以上・異常系3種類以上、具体的な等級と期待結果を添えて列挙せよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

例:

- 正常系: 等級42で生成できる。等級1と150の境界値で生成できる
- 異常系: 等級0で`GradeTooHighException`。等級151で`GradeTooLowException`。等級1で`incrementGrade()`すると`GradeTooHighException`
- 追加候補: 等級150で`decrementGrade()`すると`GradeTooLowException`。コピー代入後も左辺の名前が変わらない

</details>

---

### Q10. `operator<<`が出力すべき形式を、課題書どおりに正確に書け。`Alice`、等級42の場合の出力例も書け。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

指定形式:

```text
<name>, bureaucrat grade <grade>.
```

出力例:

```text
Alice, bureaucrat grade 42.
```

`operator<<`内で改行するか、呼び出し側で`std::endl`を付けるかは設計上の選択になる。課題書が指定する文字列本体には末尾のピリオドが含まれる。

</details>

---

### Q11. `Bureaucrat`がOrthodox Canonical Formを満たすために必要な4要素を挙げよ。例外クラスにも同じ4要素が必要か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

1. デフォルトコンストラクタ
2. コピーコンストラクタ
3. コピー代入演算子
4. デストラクタ

課題書は、例外クラスについてはOrthodox Canonical Formに従わなくてよいと明記している。

</details>

---

### Q12. `const char *what() const throw()`に現れる二つの`const`相当の指定と、`throw()`の意味をそれぞれ説明せよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

- `const char *`: 戻り値が、関数の呼び出し側から文字を書き換えられない文字列を指すポインタである
- 関数名の後ろの`const`: `what()`が例外オブジェクトの状態を変更しない
- `throw()`: C++98の動的例外仕様で、この関数の外へ例外を送出しないことを表す

`throw()`の約束に反して例外が関数外へ出ると、C++98では`std::unexpected()`を経由し、標準の設定では`std::terminate()`に至る。

</details>

---

実装に入る。ファイル名は `Bureaucrat`。`what()` を定義する。`main` を空にしない。  
次の教材は ex01。ex00 が動いてから読む。
