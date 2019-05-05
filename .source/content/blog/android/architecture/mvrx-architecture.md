+++
date = "Sun May  5 05:05:09 UTC 2019"
title = "MvRxの雑な感想"
tags = ["android", "architecture", "mvrx", "android-architecture"]
blogimport = true
type = "post"
+++

雑多なことを書いただけのブログになります。内容はあまりないよう〜。

---

[MvRx](https://github.com/airbnb/MvRx)はAirbnbが開発をしているOSSフレームワークです。

特徴としては

- Kotlinファースト
- RxJavaを使っている
- AAC（Android Architecture Components）をベースにしている
    - AACをRxJavaであったり、便利関数、クラス群で補佐している感じ
- 多くの部分でReactのAPIを参考にしてる
    - State、renderなど
- [Epoxy](https://github.com/airbnb/epoxy)と相性が良い
    - 一緒に使うことを推奨している
    - ReactのComponentのように振る舞わうことが出来る
- ViewModelが保持しているState（状態）に対して、Viewがpure functionのように振る舞う
    - 副作用がない（減らしたい）

ボイラープレートなコードを減らすことが期待できます😃

## 個人的に気になった部分、好きなとこ

### StateでView状態を管理するところ

Stateを定義することのメリットは以下かなと思ってます。

- Stateを見れば、Viewの構成要素が分かる
    - MVPアーキテクチャのViewインターフェースのような役割を果たす
- 状態の管理が楽
    - Androidでは、configuration changes時の状態の保持が難しいが、Stateだけをケアすれば良い
        - MvRxの流儀に習って、正しく実装すればよしなに状態の管理をしてくれる
- Viewからロジックを取ることが出来る
    - もちろん実装次第なのですが、StateでViewの状態を表現するようにすれば、Viewはマッピングするだけで良くなる

### Asyncがすごい良い

MvRxではAsyncというsealed classが定義されていて、

- Uninitialized
- Loading
- Success
- Fail

の4状態を表現することができます。画面の初期値は上記4状態で、大体のケースはケア出来ると思います。

プロジェクトに、Asyncだけ取り入れるのもアリだと思います。

### Single Eventの処理をどうするのか

State内で保持すると、毎回発火してしまうので、Stateとは違う、他のstreamで表現することになると思う

- invalidateメソッドではなく、直接ViewModelに定義したフィールドを、observeすることになるので、他のページと比較したときに違和感があるかも
    - やっぱりSingle Eventの取り扱いって大変なんやなって
    - とはいえ、sealed classで定義すれば、同じように見えることが出来ると思うので、大きな話ではない

## 参考

- [Tivi](https://github.com/chrisbanes/tivi)
    - プロジェクトも大きく、Coroutineなども使っている
    - これ見れば、なんとなく肌感は分かると思います
