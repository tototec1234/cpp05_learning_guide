# ex03 事後クイズ

> 対象: `CPP05_ex03_解説.md` のあと。CPP05 の締め

先に自分の回答を書く。そのあと `模範回答（クリックで表示）` をクリックして開き確認すること。

---

### Q1. 次は課題の禁止に当たるか。理由を書け。

```
AForm *Intern::makeForm(...) {
    if (name == "shrubbery creation") return new ShrubberyCreationForm(target);
    if (name == "robotomy request")   return new RobotomyRequestForm(target);
    if (name == "presidential pardon") return new PresidentialPardonForm(target);
    エラー; return NULL;
}
```

`else` が無い。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

当たる。else の有無ではなく、種類ごとに分岐が並ぶ森である。  
表＋走査に置き換える。

</details>

---

### Q2. `std::map<std::string, FormCreator>` で名前と生成関数を結ぶ。C++ としてはきれいである。提出してよいか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

いけない。STL コンテナは Module 08/09 まで禁止。評価は -42 と課題書にある。  
C 配列と関数ポインタで同じことをする。

</details>

---

### Q3. `makeForm` が未知名で `NULL` を返し、`main` がすぐ `b.signForm(*form)` する。何が起きるか。メッセージは出ていたとする。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

null 参照。メッセージのあとで落ちる。  
`NULL` を選ぶなら、使う前に必ず検査する。例外にすればこの経路は無い。

</details>

---

### Q4. Intern のデストラクタで「今まで作った Form を全部 delete する」設計は、なぜこの課題に合わないか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

Intern は作ったポインタを保持していない。戻り値として所有権を渡している。  
保持してデストラクタで消すと、呼び出し側が使っているオブジェクトを破棄する。二重 delete にもなる。

</details>

---

### Q5. 評価者が「種類を4つ目に足すならどこを直すか」と聞いた。表方式と if 方式で、答えはどう違うか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

表: 生成関数を1つ足し、表に1行足す。`makeForm` の制御流れは変えない。  
if: `makeForm` に分岐を足す。関数本体が伸び続ける。  
課題が表を求める理由を、この質問で確認している。

</details>

---

### Q6. CPP05 全体。次の文は正しいか。誤りなら直す。

「`GradeTooHighException` は、等級の数値が大きすぎるときに投げる。」

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

誤り。数値が小さすぎる（1 未満）= 権限が高すぎる、ときに投げる。  
数値が 150 超は TooLow。ex00 から ex02 の実行判定まで、この向きが共通。

</details>

---

### Q7. 自分の提出前チェックリストを、ファイル名 / 例外 / メモリ / Intern の4項目で書け。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

例:  
ファイル名: `Bureaucrat` / `AForm` / 3 Form / `Intern`。`Bureaucat` ではない。  
例外: `what()` 定義あり。High/Low が逆でない。`catch (std::exception &)` で捕まる。  
メモリ: `makeForm` の戻り値を `delete`。`AForm` デストラクタが virtual。  
Intern: 課題のキー文字列。if の森ではない。未知名でメッセージ。

</details>

---

CPP05 の教材はここまで。実装は exercise 順。ex00 の等級の向きをテストで固定してから先へ進む。
