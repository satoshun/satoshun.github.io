+++
date = "Mon Jun 24 13:52:14 UTC 2019"
title = "RxJava 3.xの開発が本格的に始まりました"
tags = ["rxjava", "rxjava3"]
blogimport = true
lastmod = "Tue Jun 25 12:38:16 UTC 2019"
type = "post"
draft = false
+++

現状での差異をまとめておきます。

## RxJava2との差異

### READMEから

[README.md](https://github.com/ReactiveX/RxJava/blob/3.x/README.md)

RxJava2 との差分は以下のようになっています。

- fixed API mistakes and many limits of RxJava 2
    - RxJava2のいくつかのAPIのミス、制限を直している
- intended to be a replacement for RxJava 2 with relatively few binary incompatible changes
    - APIに多少の変更があり、バイナリ互換がない
- test and diagnostic support via test schedulers, test consumers and plugin hooks
    - テストのサポートの充実

### 3.x different docsから

[3.x different docs](https://github.com/ReactiveX/RxJava/blob/3.x/docs/What's-different-in-3.0.md)

#### asメソッドとtoメソッド

toメソッドはFunction型を引数から取っていた。しかし、あらゆるReactive型でFunction型を受け取っていたので、共通のConverterを作ることが出来なかった。

```java
// Obsevable.java
public final <R> R to(Function<? super Observable<T>, R> converter)

// Single.java
public final <R> R to(Function<? super Single<T>, R> convert
```

同じFunction型を引数に取るので、共通のクラスを作ることが出来ない。

→ そこで、asメソッドが誕生

asメソッドでは、CompletableConverter、ObservableConverterなど、専用のインターフェース型が定義され、1つのクラスに実装できるようになりました。

```java
// Observable.java
public final <R> R to(@NonNull ObservableConverter<T, ? extends R> converter)

// Single.java
public final <R> R to(@NonNull SingleConverter<T, ? extends R> converter) {
```

→ autodisposeみたいな、ライブラリを作るときに便利

従来のtoメソッドは消えて、RxJava 3ではasに統合された。（メソッド名はtoです)

#### Functional typesがThrowableをthrowするようになった

今まではFunctional typesとして、Callableインターフェースを使っていました。

```java
@FunctionalInterface
public interface Callable<V> {
    /**
      * Computes a result, or throws an exception if unable to do so.
      *
      * @return computed result
      * @throws Exception if unable to compute a result
      */
    V call() throws Exception;
}
```
これからは、Supplierインターフェースを使うようになります。

```java
public interface Supplier<T> {

    /**
     * Produces a value or throws an exception.
     * @return the value produced
     * @throws Throwable if the implementation wishes to throw any type of exception
     */
    T get() throws Throwable;
}
```

throwする例外がException -> Throwableに広がりました。

ちなみに、lambda式を使っている場合は変更するコードを必要がないかもしれません。

```java
// before
source.to(flowable -> flowable.blockingFirst());

// after
source.to(flowable -> flowable.blockingFirst());
```

#### startWithメソッド

startWith(T)、startWith(Iterable)、 startWith(Publish)の同名で3つのメソッドがありましたが、
startWithItem、startWithIterableにそれぞれリネームされました。

## メモ・その他

- continued support for Java 6+ & Android 2.3+
    - RxJava 2.xとサポートバージョンは変わらないっぽい🎉
- RxJava 2からの、大きな変更はなさそう。APIの整理がメイン？
- 12月に正式版のリリースのようです [twitter/Rxjava](https://twitter.com/RxJava/status/1141324394595266562)

## 追記

- 3.0.0-RC0が出ました
  - https://search.maven.org/artifact/io.reactivex.rxjava3/rxjava/3.0.0-RC0/jar