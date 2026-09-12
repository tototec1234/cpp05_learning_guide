# ex01 事後クイズ

> 対象: `CPP05_ex01_解説.md` のあと

先に自分の回答を書く。そのあと `模範回答（クリックで表示）` をクリックして開き確認すること。

---

### Q1. 官僚 grade 25、Form の署名要求 25、実行要求 5。`beSigned` は成功するか。成功したとして、この官僚は後で execute できるか（ex02 の先取り）。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

署名は成功する（25 <= 25）。  
実行要求は 5 なので、同じ官僚では実行できない（25 <= 5 は偽）。署名と実行は別の閾値。

</details>

---

### Q2. `signForm` の中で `if (this->getGrade() > form.getSignGrade()) print error; else form.beSigned(*this);` と書いた場合の問題は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

判定が Bureaucrat と Form に二重になる。Form 側の規則を変えたとき、signForm の判定が古いまま残る。  
`beSigned` に判定を任せ、例外を捕捉する方が、規則の置き場所が一つになる。

</details>

---

### Q3. `catch (Form::GradeTooLowException &)` だけを書いた `main` で、Form コンストラクタが TooHigh を投げたらどう見えるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

捕捉されず、関数の外へ伝播する。未捕捉なら `std::terminate`。  
コンストラクタの High/Low と、`beSigned` の Low を、テストでは別の try に分ける。

</details>

---

### Q4. Form の代入演算子で `_name = other._name` と書くと、何が起きるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

コンパイルエラー。`_name` は const。`_signGrade` と `_executeGrade` も同様。代入できるのは `_isSigned`。

</details>

---

### Q5. 次の出力コードの問題点は何か。

```
os << form.getName()
   << ", Grade required to sign " << form.getGradeToSign()
   << ", Grade required to execute " << form.getGradeToSign();
```

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

実行等級の getter が署名等級になっている。レビューで実際に指摘された取り違え。  
`getGradeToExecute()`（名前は自分の getter に合わせる）を呼ぶ。

</details>

---

### Q6. `Form.hpp` の先頭で `#include "Bureaucrat.hpp"` し、`Bureaucrat.hpp` の先頭で `#include "Form.hpp"` すると何が起きるか。避け方は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

循環インクルード。インクルードガードがあると、一方のクラス定義が完了する前に、他方がその型を使う。  
ヘッダでは前方宣言、メンバ関数の本体は `.cpp` で相手のヘッダをインクルードする。

</details>

---

実装する。次は ex02。Form を抽象化するので、ex01 の Form が「署名状態と等級を private で持っている」ことが前提になる。
