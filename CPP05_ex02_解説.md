# ex02 解説 — AForm と実行

提出物: `Bureaucrat`、`AForm`、3つの具体 Form、`Makefile`、`main.cpp`  
Form は「紙」から「手続き」になる。チェックは共通、副作用は個別。

---

## 1. 発展の意味

```
ex01 Form   : 署名できるかどうか
ex02 AForm  : 署名したうえで、実行できるかどうか
```

3つの具体クラス:

| クラス | sign | exec | 副作用 |
|--------|------|------|--------|
| ShrubberyCreationForm | 145 | 137 | `<target>_shrubbery` に ASCII ツリーを書く |
| RobotomyRequestForm | 72 | 45 | ドリル音のあと、50% 成功 / 50% 失敗と出力 |
| PresidentialPardonForm | 25 | 5 | `<target> has been pardoned by Zaphod Beeblebrox.` |

コンストラクタは target のみ。等級を引数にしない。

---

## 2. チェックをどこに置くか

課題書: 各具体クラスでチェックしてもよいし、基底でチェックして別関数を呼んでもよい。後者の方がきれい、と明記している。

テンプレートメソッド:

```
AForm::execute(executor) const:
    if 未署名: throw 未署名例外
    if executor.grade数値 > _executeGrade: throw GradeTooLowException
    executeEachForm(executor)   # 純粋仮想。派生が実装

ShrubberyCreationForm::executeEachForm:
    ファイルを開いてツリーを書く

RobotomyRequestForm::executeEachForm:
    音を出して 50% 判定

PresidentialPardonForm::executeEachForm:
    恩赦の一文を出す
```

派生が `execute` を全部自分で書くと、未署名チェックを1クラス忘れうる。  
基底に寄せると、忘れられない。

スケルトン:

```cpp
class AForm {
public:
    virtual ~AForm();
    void execute(Bureaucrat const &executor) const;
    // beSigned, getters, OCF, 例外クラス
protected:
    virtual void executeEachForm(Bureaucrat const &executor) const = 0;
private:
    const std::string _name;
    bool _isSigned;
    const int _signGrade;
    const int _executeGrade;
};
```

関数名 `executeEachForm` は課題指定ではない。仕事用の private/protected 純粋仮想であればよい。

`execute` を純粋仮想にして、各派生の先頭で同じ if を3回書く、という形は動く。課題書が言うきれいな方ではない。

---

## 3. Bureaucrat::executeForm

課題:

```
成功: <bureaucrat> executed <form>
失敗: 明確なエラーメッセージ
```

疑似コードは `signForm` と同じ構造。

```
executeForm(form) const:
    try:
        form.execute(*this)
        成功メッセージ
    catch (std::exception &e):
        失敗メッセージ（e.what() を理由にしてよい）
```

過去のレビューコメント（提出者不明）:

> `Bureaucrat::executeForm(AForm const & form)` これだけ足りていなかった

公開の typo 指摘では、課題書本文が `Form`、評価シートが `AForm` と揺れる、とある。  
ex02 以降の引数型は `AForm const &` にする。関数自体を忘れない。

`signForm` は非 const、`executeForm` は課題が `const` を付けている。官僚の状態は実行では変わらない。

---

## 4. 具体クラスでつまずく点

### Shrubbery

- ファイル名は `<target>_shrubbery`
- `<fstream>` はコンテナではない。使ってよい
- 開けないときは例外を投げてよい（課題はファイル失敗を詳しく書いていない。評価で聞かれたら説明できるようにする）
- ASCII ツリーの見た目は指定されていない

### Robotomy

- 50%: `rand()` / `std::rand()` と `time` で種をまく実装が多い
- 種を `execute` のたびに `srand(time(NULL))` すると、1秒以内の連続実行が同じ結果になる
- 種は `main` で一度、または静的フラグで一度

### Presidential

- 出力文面が課題に固定されている

3クラスとも OCF を書く。target をメンバに持つ。target は課題が const とは書いていないが、実行中に変える理由はない。const にしてよい。

具体クラスのコンストラクタから `AForm(name, sign, exec)` を初期化リストで呼ぶ。  
name は `"ShrubberyCreationForm"` のように型名でも、課題の短い名前でも、評価者が `<<` とメッセージで識別できればよい。Intern（ex03）は文字列 `"shrubbery creation"` で生成するので、Form の `getName()` と Intern のキーは別物、と割り切ってよい。

---

## 5. 例外の種類

| 状況 | 投げうるもの |
|------|----------------|
| AForm の等級が 1..150 外 | `AForm::GradeTooHigh/LowException`（ex01 と同じ） |
| 署名時に官僚が足りない | `GradeTooLowException` |
| 未署名で execute | 独自例外が分かりやすい |
| 実行時に官僚が足りない | `GradeTooLowException` |

`GradeTooLowException` が「書類の等級が 150 超」と「官僚が足りない」の両方に使われる。  
`what()` の文言で区別するか、未署名だけ別クラスにする。  
課題はクラスを増やせとは書いていない。説明できることが評価の本体である。

---

## 6. テスト

- 3種それぞれ、署名できる官僚とできない官僚
- 署名せずに execute
- 署名したが exec 等級が足りない
- `executeForm` の成功・失敗メッセージ
- shrubbery ファイルができたか
- robotomy を複数回（片方に偏りすぎないこと。厳密 50.000% は要求されない）
- `AForm*` で保持して `delete`（virtual デストラクタの確認）

---

## 7. STL

`<fstream>`、`<ctime>`、`<cstdlib>` はコンテナではない。  
`std::vector` に Form を並べるのはまだ禁止。

---

次: `CPP05_ex02_事後クイズ.md`
