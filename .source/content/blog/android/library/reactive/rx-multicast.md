+++
date = "2017-05-04"
title = "RxJava multicastについて"
tags = ["android", "rxjava"]
blogimport = true
type = "post"
draft = false
+++

RxJavaには, stream上のアイテムを複数のSubscriberに渡せるmulticast(再利用のような感じ)の機能があります.
multicastは主に「キャッシュとして使う」, イベントをが発火したときに複数で処理をおこなう」 として使います.

RxJavaでは2つの方法でmulticastを実現します.

- `publish`を使い, `ConnectableObservable`を作る
- `Subject`を使う

また, RxJava2だとFlowableがありますが, Flowableの場合は `Processor`を使います.

説明だけだと分かりにくいので, サンプルコードも合わせてどうぞ.

- [ConnectableObservable](https://github.com/satoshun-example/RxJava-Samples/blob/master/app/src/main/java/com/github/satoshun/example/rxjava/sample/CachedNetworkWithConnectableActivity.java)
- [Subject](https://github.com/satoshun-example/RxJava-Samples/blob/master/app/src/main/java/com/github/satoshun/example/rxjava/sample/CachedNetworkWithSubjectActivity.java)


## ConnectableObservableについて

ConnectableObservableは`publish`がコールされたタイミングで生成されます.
ここがStreamの開始地点, 再利用する地点になります. なので, `map`や`filter`など, 共通の処理があるのなら,
`publish`のUpstreamに書いて共通化したほうが効率が良いです.

ConnectableObservableは`connect`メソッドがコールされたタイミングで, Streamの開始をします.
従来はSubscriberを登録した(subscribeメソッドをコールした)タイミングなので, 注意が必要です.

そして, `publish` + `connect`を組みわせることで1つのStreamに対して複数のSubscriberを登録することが出来ます.

![Publish Connect](publishConnect.png)

(source: https://github.com/ReactiveX/RxJava/wiki/Connectable-Observable-Operators)

他にも, `share`や`refcount`がありますが, 基本的な考え方は変わりません.


## Subjectを使う

Subjectは, ObservableとObserverの機能を同時に使うことができる実装です.
1例としては, Observerの機能を使い, 複数のObservableからデータを取得し, Observableの機能を使いそのデータをObserverに渡す, いわゆるBridgeのような事ができます.

`BehaviorSubject`, `ReplaySubject`などのバラエティがあるので, 必要に応じて使い分ける必要があります. (https://github.com/ReactiveX/RxJava/wiki/subject)


## まとめ

multicastを上手に使いこなすことで, データの取得回数を減らし, アプリがパフォーマンスに優れた構造になることが期待できます.
そのためには, どこにmulticastを入れるか, どこからConnectableObservableにするか(publishをコールするか)を決定する必要があります.
アプリごとに適切なポイント, 書き方は異なると思うので, サンプルを参考に頂けたら幸いです.

Happy RxJava2 life!!

## 参考

- [Multicasting in RxJava](http://blog.danlew.net/2016/06/13/multicasting-in-rxjava/)
- [RxJava](https://github.com/ReactiveX/RxJava)
