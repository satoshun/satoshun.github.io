+++
date = "2017-12-23"
title = "RxJava: as, toの違いについて"
tags = ["android", "rxjava"]
blogimport = true
type = "post"
draft = false
+++

RxJavaでobservable型(Flowable、Observableなど)を他の型に変換したいときには、toまたはasを使います。

この記事ではそれぞれの違いを説明します。

## toについて

toのシグニチャは以下のようになります。

```java
<R> R to(Function<? super Flowable<T>, R> converter)
<R> R to(Function<? super Observable<T>, R> converter)
<R> R to(Function<? super Maybe<T>, R> convert)
<R> R to(Function<? super Single<T>, R> convert)
<U> U to(Function<? super Completable, U> converter)

public interface Function<T, R> {
    R apply(@NonNull T t) throws Exception;
}
```

それぞれconverterをFunctionインターフェースで指定でき、upstream(オリジナルのObservable)を任意の型に変換することが出来ます。


## asについて

asのシグニチャは以下のようになります。

```java
<R> R as(@NonNull FlowableConverter<T, ? extends R> converter)
<R> R as(@NonNull ObservableConverter<T, ? extends R> converter)
<R> R as(@NonNull MaybeConverter<T, ? extends R> converter)
<R> R as(@NonNull SingleConverter<T, ? extends R> converter)
<R> R as(@NonNull CompletableConverter<? extends R> converter)

public interface FlowableConverter<T, R> {
    R apply(@NonNull Flowable<T> upstream);
}
public interface ObservableConverter<T, R> {
    R apply(@NonNull Observable<T> upstream);
}
// MaybeConverter, SingleConverter, CompletableConverterも同じ内容
```

それぞれのobsevable型に対して、converterを専用のインターフェースを与えられることが分かります。

しかし中身はほとんどFunctionインターフェースと変わりません。converterを専用のインターフェースに分けると何が嬉しいんでしょうか?


## 複合インターフェースを作るときに便利

https://github.com/ReactiveX/RxJava/issues/5654 ここにasオペレータが誕生した理由が書いてあります。そこには

```
For the compose() operators, ObservableTransformer was introduced to allow for classes to implement multiple transformer interfaces for composite implementations`
```

と書いてあります。つまり、Javaの場合はインターフェースが同じ場合、型パラメータが異なっていたとしても複合インターフェースにすることは出来ません。asのようにインターフェースを分けることで複合インターフェースを作ることが可能になります。具体的なコードでは、AutoDisposeは下のような使い方をしています

```java
public interface AutoDisposeConverter<T> extends FlowableConverter<T, FlowableSubscribeProxy<T>>,
    ObservableConverter<T, ObservableSubscribeProxy<T>>,
    MaybeConverter<T, MaybeSubscribeProxy<T>>,
    SingleConverter<T, SingleSubscribeProxy<T>>,
    CompletableConverter<CompletableSubscribeProxy> {
}
```

https://github.com/uber/AutoDispose/blob/deab8fb26f2ebeed906cb766e6ad253e772ec84c/autodispose/src/main/java/com/uber/autodispose/AutoDisposeConverter.java

複合インターフェースを構成していることが分かります。

## まとめ

- asはconverterのインターフェースが異なるので複合インターフェースを構成でき、汎用的な実装クラスを作成することが可能
