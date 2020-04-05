+++
date = "2017-05-04"
title = "RxJava: SubjectでonErrorを取り扱う時"
tags = ["android", "rxjava"]
blogimport = true
type = "post"
draft = false
+++

## 結論

- materialize, dematerializeを使う.


## 背景

onErrorをSubjectに流すと, Subjectはdispose状態になるため, これ以降のEventを流すことが出来なくなります.

```java
sub.subscribe(
    v -> Log.d("onNext", String.valueOf(v)),
    e -> Log.d("onError", e.getMessage()));
sub.onNext(v1);
sub.onNext(v2);
sub.onError(e1);
sub.onNext(v3); // ignore
sub.onNext(v4); // ignore
```

これだと, Subjectがこれ以降に値を受け付けないため不便になることがあります.(ユースケース次第)

`materialize` + `dematerialize`オペレータを使うと, この問題を解決出来ます.

```java
Observable.create((ObservableOnSubscribe<String>) e -> {
    e.onNext("1");
    e.onNext("2");
    e.onError(new RuntimeException("e1"));
}).materialize().subscribe(sub::onNext);
Observable.create((ObservableOnSubscribe<String>) e -> {
    e.onNext("3");
    e.onNext("4");
    e.onError(new RuntimeException("e2"));
}).materialize().subscribe(sub::onNext);
```

`materialize` を使い, ErrorイベントをNextイベントに変換します

そして, Subscribeするタイミングで, `dematerialize`でNextイベントをErrorイベントに戻します.

```java
sub.dematerialize().subscribe(
    v -> Log.d("onNext", String.valueOf(v)),
    e -> Log.d("onError", e.getMessage()));
```

こうすることで, Subjectをdispose状態にせずに, ObserverでonErrorイベントを処理することが出来ます.


## 参照

- [ReactiveX - Materialize & Dematerialize operators](http://reactivex.io/documentation/operators/materialize-dematerialize.html)
