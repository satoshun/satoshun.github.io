+++
date = "2017-05-04"
title = "Android: RxJava subscribeOnとobserveOnの挙動について"
tags = ["android", "rxjava"]
blogimport = true
type = "post"
draft = true
+++

## subscribeOnについて

subscribeOnを複数回登録しても1つしか採用されません.

例えば,

```java
Observable.just(1)
    .subscribeOn(Schedulers.io())
    .map((i) -> String.valueOf(i))
    .subscribeOn(Schedulers.computation())
    .subscribe();
```

この場合, `Schedulers.io()`が実行されます. `Schedulers.computation()`は無効です(厳密にいうと実行自体はされるのですが, Streamには影響を与えません)

これはreactivexの仕様で, UpStreamのsubscribeOnが採用されるようになっているため, この挙動になります.

(RxJavaには, unsubscribeOnがありますが今回はそこには触れません)


## observeOnについて

observeOnはDownStreamに影響を与えます.

例えば,

```java
Observable.just(1)
    .observeOn(Schedulers.io())
    .map((i) -> String.valueOf(i))
    .observeOn(Schedulers.computation())
    .subscribe();
```

この場合, mapが`Schedulers.io()`, subscribeが`Schedulers.computation()`で実行されます.

observeOnは常にDownStreamに影響を及ぼします.


## まとめ

subscribeOnはUpStreamで定義されているものが採用されるので, UpStreamで定義するより, 出来る限りDownStream(subscribeする直前)で定義したほうが柔軟性が高くなります.
逆に言うと, UpStreamでsubscribeOnを定義しておけば, Schedulerを固定することができるので,
ワーカースレッドで必ず実行しなければ行けないObservableの場合はUpStreamで定義すると, 間違いが起きないため良いかもしれないです.
ただ, 基本的には, subscribeする直前で定義したほうが可読性が高いので良いと思います.


## refs

- [document](http://reactivex.io/documentation/scheduler.html)
