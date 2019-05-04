+++
date = "Sat May  4 12:52:28 UTC 2019"
title = "（仮）MvRx"
tags = ["android", "architecture", "mvrx"]
blogimport = true
type = "post"
draft = true
+++

雑多なことを書いただけのブログになります。内容はあまりないよう〜。

---

[MvRx](https://github.com/airbnb/MvRx)はAirbnbが開発をしているOSSフレームワークです。

特徴としては

- Kotlinファースト
- AAC（Android Architecture Components）をベースにしている
- [Epoxy](https://github.com/airbnb/epoxy)と相性が良い
  - 一緒に使うことを推奨している
- RxJavaを使っている
- 多くの部分でReactのAPIを参考にしてる
  - State、render、Epoxyなど

## 個人的に気になった部分、好きなとこ

### StateでView状態を管理するところ

Stateを定義することのメリットは以下かなと思ってます。

- Stateを見れば、Viewの構成要素が分かる
    - MVPアーキテクチャのViewインターフェースのような役割を果たす
- 状態の管理が楽
    - Androidでは、configuration changes時の状態の保持が難しいが、Stateだけをケアすれば良い
        - （MvRxの流儀に習って、正しく実装すれば）
- Viewからロジックを取ることが出来る
    - もちろん実装次第なのですが、StateでViewの状態を表現するようにすれば、Viewはマッピングするだけで良くなる

### Asyncがすごい良い

MvRxではAsyncというsealed classが定義されていて、

- Uninitialized
- Loading
- Success
- Fail

の4状態を表現することができます。画面の初期値は上記4状態で、大体のケースはケア出来ると思います。

Asyncだけ取り入れるのもアリだと思います。

##　個人的にどうするんだろうと思ったところ

### Single Eventの処理をどうするのか

State内で保持すると、毎回発火してしまうので、Stateではない、他のstreamで表現することになると思う
- invalidateメソッドではなく、直接observeすることになるので、他のページと比較したときに違和感があるかも
    - やっぱりSingle Eventの取り扱いって大変なんやなって
