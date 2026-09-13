# ex02 事後クイズ

> 対象: `CPP05_ex02_解説.md` のあと

先に自分の回答を書く。そのあと `模範回答（クリックで表示）` をクリックして開き確認すること。

---

### Q1. 派生の `execute` 先頭で「未署名なら throw」を3クラスに書いた。あとから「既に実行済みなら throw」を足したくなった。何がつらいか。基底に寄せた場合はどうか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

3箇所を同じように直す。1つ忘れる。  
基底の `execute` に条件を足せば、派生の仕事関数は触らない。

</details>

---

### Q2. `AForm::execute` を非 const にし、`executeForm` を非 const にする、のどちらが課題文とずれるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

両方ずれる。課題は `execute(Bureaucrat const & executor) const` と `executeForm(AForm const & form) const`。  
書類の実行は書類の署名状態を変えない、官僚の等級も変えない、という印が const。

</details>

---

### Q3. Robotomy を連続で10回呼んだら全部成功した。ありうる原因を2つ述べよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

1. `srand(time(NULL))` を毎回呼び、同じ秒内で種が同じ。  
2. 乱数を使わず、常に成功する実装。  
評価者は失敗側の出力も見る。

</details>

---

### Q4. レビューで `executeForm` が無かった提出が再提出になっている。`execute` だけ実装済みだと、何が足りないのか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

官僚側の公開 API と、成功／失敗の指定メッセージ。  
`form.execute(b)` を `main` から直接呼んでも動作確認はできるが、課題の提出物としては `Bureaucrat::executeForm` が必須。

</details>

---

### Q5. `ShrubberyCreationForm s("home"); AForm a = s;` はコンパイルできるか。スライシングとは何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`AForm` が抽象なら、`AForm a = s` はオブジェクトを作れずコンパイルエラー。それが目的。  
もし純粋仮想を忘れて抽象になっていないと、基底部分だけコピーされ、派生の target や関数が消える（スライシング）。抽象にしておくと、この事故を型が止める。

</details>

---

### Q6. 未署名例外を `std::exception` から継承し忘れた。`executeForm` の `catch (std::exception &)` はどうなるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

捕まらない。`executeForm` の外へ出る。メッセージが出ず、プログラムが落ちることがある。  
自作例外はすべて `std::exception` 系にする、という ex00 の約束がここでも効く。

</details>

---

実装する。次は Intern。`AForm*` で具体を扱う準備ができていることが前提。
