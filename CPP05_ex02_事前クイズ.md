# ex02 事前クイズ

> 対象: ex01 が動いたあと、`CPP05_ex02_解説.md` の前  
> CPP04 の抽象クラスを忘れているなら `CPP05_テーマと発展.md` の橋渡しを再読する

先に自分の回答を書く。そのあと `模範回答（クリックで表示）` をクリックして開き確認すること。

---

### Q1. 課題は Form を抽象クラスにして名前を `AForm` に変えろ、と書く。抽象にする手段は何か。なぜオブジェクト `AForm a;` を作れなくするのか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

純粋仮想関数を1つ以上置く。  
「署名と等級だけの書類」は存在させず、必ず具体的な手続き（植樹・ロボトミー・恩赦）としてだけ存在する、という設計。

</details>

---

### Q2. 属性は基底の private のままである。派生の `ShrubberyCreationForm` が署名済みかを知る方法は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

基底の getter を呼ぶ。または、チェック自体を基底の `execute` に寄せ、派生はチェック済み前提で仕事だけする。後者が課題書の「より elegant」な方。

</details>

---

### Q3. `AForm::execute` が確認すべき2条件は何か。足りないとき何を投げればよいか（課題は「適切な例外」としか書いていない）。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

署名済みであること。実行者の等級が実行に必要な等級以上（数値 <=）であること。  
未署名は独自例外（例: `FormNotSignedException`）が説明しやすい。等級不足は既存の `GradeTooLowException`。

</details>

---

### Q4. `AForm* p = new RobotomyRequestForm("x"); delete p;` で派生のデストラクタを呼ぶ条件は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`AForm` のデストラクタが `virtual`。CPP04 と同じ。

</details>

---

### Q5. 3つの具体 Form のコンストラクタ引数は何か。sign / exec の数値はコンストラクタ引数か、型に固定か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

引数は target だけ。  
sign/exec は型ごとに固定（145/137、72/45、25/5）。呼び出し側に等級を選ばせない。

</details>

---

次: `CPP05_ex02_解説.md`
