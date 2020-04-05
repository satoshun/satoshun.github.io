+++
date = "2017-05-04"
title = "RxJava: Connectable Observableについて"
tags = ["android", "rxjava"]
blogimport = true
type = "post"
draft = false
+++

## 結論

- 同じStreamを複数のObserverで扱いたいときはConnectable Observableを使う
    - 効率的!!
- Connectable Observableはsubscribeをコールしても, Streamが開始しない
    - `connect`メソッドをコールした時に, Streamが開始する


## 基本的な使い方

同じデータを必要としているObserverがあるとします.
これを単純に書くと以下のようになります.

```java
Obervable<User> getUsers() { return Observalbe.just(User1, User2, User3);}

Obervable<User> users = getUsers();
users.subscribe(o1);
users.subscribe(o2);
users.subscribe(o3);
```

これだと, subscribeされるたびに新しいStreamが作られてしまいます.
上の例だと3つの独立したObservableが作成されてしまい, 非常に効率が悪いです.

Connectable Observableを使うことで, 1つのStreamからsubscribeすることが出来ます.

```java
ConnectableObservable<User> users = getUsers().publish();
users.subscribe(o1);
users.subscribe(o2);
users.subscribe(o3);
users.connect(); // 重要!!
```

`publish`オペレータを適用することで, Connectable Observableに変換出来ます. そして, Connectable Observableに対してObserverをsubscribeし, 最後に`connect`をコールすることで,
共通のStreamからsubscribeすることが出来ます.

流れとしては以下のようになります.

1. `publish`をコールすると, Connectable Observableが作成出来る
1. Connectable ObservableをいくつかのObserverでsubscribeする
    - ここではStreamは開始されず, 登録のみされる
1. 最後に`connect`をコールする
    - Streamが開始される. 登録したObserverに対して`共通の値`がemitされていく

これが基本的なConnectable Observableの使い方になります.


## まとめ

`publish` + `connect`は, Connectable Observableの中でも一番ポピュラーなものです.
RxJavaには他にも, `replay`や`cache`といったConnectable ObservableのファミリーAPIがあります.
これらは, ユースケースによってはConnectable Observableをより便利に使うことが出来るものです.

これらについては, また他の記事で紹介いたします.

最後まで読んでいただきありがとうございました!


## 参照

- [Connectable Observable](https://github.com/ReactiveX/RxJava/wiki/Connectable-Observable-Operators)
- [Publish Operator](http://reactivex.io/documentation/operators/publish.html)
- [Connect Operator](http://reactivex.io/documentation/operators/connect.html)
