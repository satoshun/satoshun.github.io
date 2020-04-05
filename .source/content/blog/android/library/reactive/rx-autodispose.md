+++
date = "2017-05-04"
title = "RxJava: AutoDisposeで自動的にdisposeする"
tags = ["android", "rxjava", "autodispose"]
blogimport = true
type = "post"
draft = false
+++

非同期処理を開始した後に, Activityのライフサイクルなどに連動して非同期処理をキャンセルする必要があります.

RxJavaでは, Disposableからキャンセルの処理を行います.

```java
Disposable d;

public void onCreate() {
    d = myObservable
        .map(a -> b)
        .subscribe();
}

public void onPause() {
    d.dispose();
}
```

このコードだと, ストリームの定義と, キャンセルのコードが離れ過ぎているため, 可読性が低いです.

[AutoDispose](https://github.com/uber/AutoDispose)を使うことで, コード離れすぎ問題を解決することが出来ます.

```java
public void onCreate() {
    myObservable
        .to(new ObservableScoper<>(this))
        .subscribe();
}
```

このような感じで, ストリームの定義の中にAutoDisposeが提供している`Scoper`を埋め込むことで自動的に`dispose`してくれます.

また, `Scoper`は, カスタムで柔軟に定義することが出来ます. 例えば,

- Activityのライフサイクル
- Fragmentのライフサイクル
- RecyclerViewのライフサイクル

など, こちら側で実装することが出来ます(実装の手間は少し掛かります)

これで, Disposableをマニュアルで呼び出すことから解放されます!


## Error Proneと組み合わせると最強

とはいえ, そもそも`Scope`をストリームに埋め込むのを忘れる問題があります(だって人間だもの)

そこで, Googleが出した[Error Prone](http://errorprone.info/)を組み合わせることで埋め込むのを忘れる問題を解決出来ます.

Error ProneはJavaの間違った使い方を教えてくれる `Linter`のようなものです. これを使うと,

```java
public void onCreate() {
    myObservable
        .map(a -> b)
        .subscribe();
}
```

と, `Scope`の指定を忘れた時に, エラーを出力してくれます.
なぜかというと, RxJava2は `@CheckReturnValue`が返り値についているため, それをError Proneが検知してくれるためです.

```java
public void onCreate() {
    myObservable
        .to(new ObservableScoper<>(this))
        .subscribe();
}
```

と書くと, `@CheckReturnValue`が返り値についていない `Disposable`が返されるようになるので, 返り値をチェックしなくてもError Proneに引っかかることはなくなります.

RxJavaの`dispose`忘れはついやってしまいがちですが, AutoDispose + Error Proneを組み合わせることで機械的に防ぐことが出来ます!


## まとめ

- AutoDisposeは超便利
- AutoDispose + Error Proneを組み合わせると`dispose`忘れを防げる
