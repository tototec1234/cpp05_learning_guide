# ex00 事前クイズ

> 対象: `CPP05_テーマと発展.md` を読んだあと、`CPP05_ex00_解説.md` を読む前  
> 目的: いまの理解を書き出す。分からなければ `?`

先に自分の回答を書く。そのあと `模範回答（クリックで表示）` をクリックして開き確認すること。

---

### Q1. C の関数は失敗をどう返すことが多いか。C++ のコンストラクタが同じ方法を使えない理由は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

C では戻り値（`NULL`、`-1`）と `errno` が多い。  
コンストラクタに戻り値はない。失敗を呼び出し側へ伝える手段として、このモジュールでは例外を使う。

</details>

---

### Q2. 等級 1 と 150 のどちらが「高い」か。`incrementGrade()` を等級 3 に対して呼ぶと数値はどうなるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

1 が最高、150 が最低。  
`incrementGrade()` は権限が上がるので、数値は減る。3 → 2。

</details>

---

### Q3. 等級 0 で `Bureaucrat` を作ろうとしたとき、投げるべき例外の名前は何か。等級 151 では何か。なぜその対応になるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

0 → `GradeTooHighException`（数値が 1 より小さい = 権限が高すぎる）。  
151 → `GradeTooLowException`（数値が 150 より大きい = 権限が低すぎる）。  
例外の High/Low は権限の高低であり、数値が大きい／小さいことではない。

</details>

---

### Q4. 課題書の `catch (std::exception & e)` が自作例外を捕まえられるために、自作例外クラスは何を満たす必要があるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`std::exception` を直接または間接に継承していること。  
`std::logic_error` 経由でも技術的には捕捉できる。課題書の例は `catch (std::exception & e)` である。この教材では `std::exception` を直接継承する実装を推奨する。ただし、直接継承は課題の必須条件ではない。

</details>

---

### Q5. クラスのメンバが `const std::string _name` のとき、コピーコンストラクタと代入演算子で `_name` にできることの違いは何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

コピーコンストラクタは初期化リストで `_name` を初期化できる。  
代入演算子は既に初期化済みの const メンバを代入できない。`_grade` だけコピーする。

</details>

---

### Q6. 例外クラスは Orthodox Canonical Form が免除される。Bureaucrat 本体は免除されるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

Bureaucrat は OCF 必須。デフォルトコンストラクタ、コピーコンストラクタ、代入演算子、デストラクタを書く。  
例外クラスは OCF が免除される。`std::exception` を直接継承する場合は `what()` を定義する。標準例外クラスを基底にする場合は、基底クラスの `what()` を利用できる。

</details>

---

### Q7. `what()` のシグネチャが `const char *what() const throw();` である理由を、`const` と `throw()` に分けて述べよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

- 関数名の後ろの `const`: 例外オブジェクトの状態を変更せずにメッセージを返す。`catch (std::exception const &)` からも呼べる
- `const char *`: 戻り値はポインタである。指し先の文字は、呼び出し側から書き換えられない
- `throw()`: C++98 の動的例外仕様で、「この関数は例外を投げない」（この関数の外へ例外を送出しない）。C++11 の `noexcept` に近い役割を持つ古い書き方

</details>

---

### Q8. 次の疑似コードのどちらが「強い例外保証」か。理由を書け。

```
A:
  _grade を 1 減らす
  if 範囲外: throw

B:
  if 減らすと範囲外: throw
  _grade を 1 減らす
```

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

B。失敗時に `_grade` が元のまま残る。A は `throw` した時点で既に書き換えている。

</details>

---

次: `CPP05_ex00_解説.md`
