# ex01 解説 — Form と署名

提出物: ex00 のファイル + `Form.hpp` / `Form.cpp`  
増える関係: 官僚が書類に署名する。例外は書類が出す。文章は官僚が出す。

---

## 1. ex00 からの変化
<details>
<summary>Design by Contract（契約による設計）の観点から言うと:</summary>

C++での「契約による設計（Design by Contract: DbC）」は、関数やクラスの事前条件（Preconditions）、事後条件（Postconditions）、および不変条件（Invariants）を明確にし、コンポーネント間の義務と権利を定義する設計手法です。

- 不変条件（Invariants）: オブジェクトが常に守る性質（等級は 1..150）
- 事前条件（Preconditions）: 操作を始めてよい条件（この官僚がこの書類に署名できるか）
- 事後条件（Postconditions）: 操作のあとで成り立つ性質（署名成功なら `_isSigned` が true。失敗なら状態は変わらない）

参考: [Design by Contract Programming in C++（EventHelix）](https://www.eventhelix.com/object-oriented/design-by-contract/)

</details>

```
ex00: クラス不変条件（1インスタンスの等級が常に 1..150）
ex01: 操作の事前条件（この官僚が、この書類に署名する権限があるか）
```
「クラス不変条件 / 操作の事前条件」は Design by Contract の用語そのものです。


平たく言うと:

```
ex00: オブジェクト単体の妥当性（等級が範囲内か）
ex01: オブジェクト間の権限判定（この官僚がこの書類に署名できるか）
```



比較対象が「絶対値の 『1 と 150』」から「相対値である『相手が要求する等級』」に変わる。  
向きを間違えると、高い官僚が署名できず、低い官僚が署名できる、という逆転バグになる。

```
権限が足りる  ⇔  官僚の等級数値 <= 書類が要求する等級数値
```

例:

| 官僚 | 署名に必要 | 数値比較 | 結果 |
|------|------------|----------|------|
| 1 | 150 | 1 <= 150 | 誰でもほぼ署名できる書類 |
| 150 | 1 | 150 <= 1 は偽 | ほぼ誰も署名できない |
| 50 | 30 | 50 <= 30 は偽 | 足りない |
| 30 | 30 | 30 <= 30 | 等しいので足りる（課題書: higher or equal） |

---

## 2. 責任の分割

```c++
Bureaucrat::signForm(form):
    try:
        form.beSigned(*this)
        出力: "<bureaucrat> signed <form>"
    catch (std::exception &e):
        出力: "<bureaucrat> couldn't sign <form> because <reason>."
```

```c++
Form::beSigned(bureaucrat):
    if 官僚の等級数値 > 署名に必要な等級数値:
        throw Form::GradeTooLowException
    署名済みにする
```

`signForm` の中で等級を自分で判定し、`beSigned` を呼ばない実装は、規則が二箇所に分かれる。  
判定は Form が担い、Bureaucrat は依頼とログ出力だけを担うのがきれい。

既に署名済みのとき、課題は何も書いていない。何もしない／メッセージだけ／例外、いずれも本文の必須ではない。評価で聞かれたら「課題に無いので、再署名は無視した」と説明できればよい。

---

## 3. Form のスケルトン

```cpp
class Form {
public:
    Form();
    Form(const std::string &name, int signGrade, int executeGrade);
    Form(const Form &other);
    Form &operator=(const Form &other);
    ~Form();

    // getters: name, isSigned, signGrade, executeGrade
    void beSigned(const Bureaucrat &bureaucrat);

    class GradeTooHighException : public std::exception { /* what() */ };
    class GradeTooLowException  : public std::exception { /* what() */ };

private:
    const std::string _name;
    bool _isSigned;
    const int _signGrade;
    const int _executeGrade;
};
```

コンストラクタの疑似コード:

```cpp
Form(name, signGrade, executeGrade):
    _isSigned = false
    各等級が < 1 なら Form::GradeTooHighException
    各等級が > 150 なら Form::GradeTooLowException
```

Bureaucrat と Form で例外クラスが別になる。  
```cpp
catch (std::exception &)
```
 ならどちらも捕まる。  
```cpp
catch (Bureaucrat::GradeTooLowException &)
```
では Form の例外は捕まらない。

- 前方参照: `Form.hpp` が `Bureaucrat` をポインタ／参照で使うなら、クラスの前方宣言で足りる。
- `getGrade()` を呼ぶ実装は `.cpp` で `Bureaucrat.hpp` をインクルードする。
- 循環インクルードをヘッダ同士で作らない。

---

## 4. Bureaucrat 側に足すもの
- `signForm(Form &form)` をメンバに追加。  
- ex00 のファイルをコピーしたあと、`Bureaucrat.hpp` に `signForm` の宣言を足す。ex00 のヘッダをそのまま使わない。
- `Bureaucrat.hpp` では `Form` を前方宣言する。
- `signForm` の実装は `.cpp` で `Form.hpp` をインクルードする。

---

## 5. テストに入れること

- 作成時の等級 0 / 151 で Form 自身の例外
- 権限が足りる署名、足りない署名
- 等号（等級ちょうど）
- `signForm` の成功メッセージと失敗メッセージ
- `<<` に name / signed / sign grade / exec grade がすべて出るか
- コンストラクタで throw した Form は存在しないこと

ex01 のレビューコメント（提出者不明）: 「例外に当てはまらない場合がどうなるのか」。  
正常系を1本も見せないと、評価者は失敗パスしか見られない。

---

## 6. 例外保証

`beSigned` は、throw するなら `_isSigned` を変えない。  
先に `_isSigned = true` してから等級を見ると、失敗しても署名済みになる。

---

次: `CPP05_ex01_事後クイズ.md`
