# ex01 事前クイズ

> 対象: ex00 を実装したあと、`CPP05_ex01_解説.md` を読む前  
> 前提: 等級 1 が最高であることは説明できる

先に自分の回答を書く。そのあと `<details>` を開く。

---

### Q1. Form が持つ4つの情報は何か。どれが const で、署名済みフラグの初期値は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

名前（const）、署名済みか（bool、初期値 false）、署名に必要な等級（const）、実行に必要な等級（const）。  
属性はすべて private。protected ではない。

</details>

---

### Q2. 「官僚の等級が、書類が要求する等級以上（高いまたは等しい）」を、整数比較で書け。官僚 50、署名要求 30 のとき、署名できるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`bureaucrat.getGrade() <= form.getSignGrade()` なら権限が足りる。  
50 は 30 より数値が大きいので権限は低い。署名できない。`GradeTooLowException`。

</details>

---

### Q3. `Form::beSigned` と `Bureaucrat::signForm` の役割の違いは何か。どちらが例外を投げ、どちらがメッセージを出すか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`beSigned`: 書類が自分の規則で署名状態を変える。足りなければ例外。  
`signForm`: 官僚が書類に依頼し、成功／失敗の文章を標準出力に出す。失敗理由は捕まえた例外から取る。

</details>

---

### Q4. Form の名前と必要等級が const のとき、代入演算子は何をコピーできるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

署名済みフラグだけ。名前と2つの等級は代入できない。ex00 の名前と同じ制約。

</details>

---

### Q5. なぜ Form の属性を protected にして派生から直接触らせないのか。この時点では派生クラスはまだない。課題が先に禁じる理由を推測せよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

ex02 で具体 Form が増えても、等級と署名状態の唯一の管理者を基底に残すため。派生が `_isSigned` を直接 true にすると、署名に必要な等級のチェックを迂回できる。

</details>

---

次: `CPP05_ex01_解説.md`
