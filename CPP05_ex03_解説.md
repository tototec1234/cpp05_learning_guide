# ex03 解説 — Intern と生成の分離

提出物: ex02 まで + `Intern.hpp` / `Intern.cpp`  
官僚は書類の種類を `new` しなくてよくなる。文字列を Intern に渡す。

---

## 1. なぜ Intern が分かれているか

```
ex02: main が ShrubberyCreationForm f("home"); と型を直接書く
ex03: Intern が "shrubbery creation" から AForm* を返す
```

呼び出し側は具体型のヘッダを知らなくてよい、わけではない（リンクは必要）。  
意図は、**生成規則を一箇所に閉じる**こと。種類を足すとき、直す場所が Intern になる。

IRC の B 層で、コマンド名からハンドラを選ぶdispatcher と同じ問題である。  
if/else の森は、種類が増えるたびに分岐が伸び、読み手が条件を追い切れなくなる。

---

## 2. 課題が拒否する形

```
if (name == "shrubbery creation")
    return new ShrubberyCreationForm(target);
else if (name == "robotomy request")
    return new RobotomyRequestForm(target);
else if (name == "presidential pardon")
    return new PresidentialPardonForm(target);
else
    エラー
```

3個なら動く。課題は「Piscine ではない」と書いて拒否する。  
評価者は構造を見る。動くことと通ることは別。

---

## 3. 許可される骨格（疑似コード）

関数ポインタの表。コンテナではない。C の配列。

```
型 FormCreator = AForm* (*)(string const & target)

createShrubbery(target):  return new ShrubberyCreationForm(target)
createRobotomy(target):   return new RobotomyRequestForm(target)
createPardon(target):     return new PresidentialPardonForm(target)

表:
  { "shrubbery creation",   createShrubbery }
  { "robotomy request",     createRobotomy }
  { "presidential pardon",  createPardon }

makeForm(name, target):
    i を 0 から表の長さ未満:
        if name == 表[i].文字列:
            出力 Intern creates <form>
            return 表[i].関数(target)
    明確なエラーメッセージ
    return NULL   # または throw
```

`switch` を使うなら、名前照合で得た index に対して1回だけ。  
名前ごとに `case` を並べて `new` するだけなら、if/else と同じ読みにくさ、と評価者が判断することがある。  
表に名前と生成関数を並べる方が、「種類を足す = 行を1行足す」と説明できる。

過去のレビューコメント（提出者不明）: 配列でも足りるが、構造体で名前と処理を組にした方が分かりやすい。

---

## 4. キー文字列

課題書の例:

```
rrf = someRandomIntern.makeForm("robotomy request", "Bender");
```

キーは次で揃えるのが安全である。

- `"shrubbery creation"`
- `"robotomy request"`
- `"presidential pardon"`

大文字小文字を吸収するかは本文に無い。例はすべて小文字。例どおりでよい。

---

## 5. 所有権

```
AForm *f = intern.makeForm("robotomy request", "Bender");
if (f != NULL) {
    // 署名・実行
    delete f;
}
```

`AForm` のデストラクタが virtual であること（ex02）が、ここでも必要。  
`makeForm` が失敗で `NULL` を返すなら、呼び出し側の null チェックを忘れるとクラッシュする。  
例外にするなら、成功時だけポインタが返り、失敗時は `catch` する。所有権の説明は例外の方が短い。課題はメッセージ必須で、例外必須ではない。

Intern は状態を持たない。コピーも代入も「何もしない」で OCF を満たす。`(void)other;` で足りる。

---

## 6. テスト

- 例と同じ `"robotomy request", "Bender"`
- 残る2種のキー
- 未知の名前でメッセージが出る
- 返したポインタを署名・実行できる
- `delete` する（リークチェック）
- Intern をコピーしても `makeForm` が同じように動く

---

## 7. モジュール全体の振り返り

```
Bureaucrat : 等級という不変条件。例外で「存在しない」
Form       : 2者間の権限比較。例外は書類、ログは官僚
AForm      : 共通チェック + 派生の副作用。抽象と virtual
Intern     : 文字列から具体へ。分岐表。ヒープの所有は呼び出し側
```

ex00 の High/Low の向きが、最後まで署名と実行の判定に残る。  
最初の誤解を残したまま Intern まで進まない。

---

次: `CPP05_ex03_事後クイズ.md`
