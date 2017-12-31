+++
date = "2015-05-22T00:00:00Z"
title = "ReactiveX: Hot, Coldの違い"
tags = ["reactive"]
blogimport = true
type = "post"
+++


[ReactiveX](http://reactivex.io/)における, `Cold/Hot` Observableの違いを説明します.

(本文中の「subscribeする」と, 「Observerを登録する」は同義です.)


## Cold Observable

Cold ObservableはSubscribeされると動作を開始します.

```js
var source = Rx.Observable.range(1, 10);

// 何か処理
// ...

source.subscribe(function(x) {
  console.log(x); // 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
});
```

例えば, 上のコードだと, sourceを定義した時点ではストリームが生成されておらず,
source.subscribeされた時に, 初めてストリームが生成されます.
遅延評価(lazy evaluation)のような振る舞いをします.

次に, 1つのCold Observableに対して複数subscribeしたとします.

```js
var source = Rx.Observable.range(1, 10);

// 何か処理
// ...

source.subscribe(function(x) {
  console.log(x); // 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
});
source.subscribe(function(x) {
  console.log(x); // 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
});
```

2つのsubscribeに対して, 別々のストリームが生成されます. 各subscribeは, `完全に独立して動作`します.

Cold Observableをまとめると,

- subscribeされるまでCold状態(ストリームが生成されない)
- ストリームはObserverごとに独立して生成される


## Hot Observable

次にHot Observableです. Hot Observableはsubscribeしないでもストリームが生成されます.

```js
var source = Rx.Observable.interval(1000);
var hot = source.publish();

// この時点ではストリームは生成されていない
hot.subscribe(function() {
  console.log('part1');
});

// ストリームが生成される
hot.connect();

hot.subscribe(function() {
  console.log('part2');
});
```

RxJSでは, publishメソッドをコールするとHotなObservableを作成することが出来ます. それに対して, connectするとストリームが流れ始めます.
subscribeとは無関係にストリームが生成されるのがポイントです. Observerが登録されていない時は, データは消滅します.
また, HotなObservableは, 1つのストリームに対して複数subscribeすることが出来ます. Coldの場合は, ストリームとsubscribeは, 1対1だったのが, Hotの場合は1対多になります.

Hot Observableをまとめると,

- subscribeとは無関係にストリームが生成される
- 複数Observerを1つのストリームの上に乗せることが出来る


## まとめ

Hot, ColdなObservableについて説明しました. 1つのストリーム(同じデータ)に対して複数のことをしたいときはHot, そうでない時はColdを使えば良いと思います. 基本的にはColdを扱うことが多いとは思います.
