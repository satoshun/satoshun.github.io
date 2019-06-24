+++
date = "Mon Jun 24 13:52:14 UTC 2019"
title = "RxJava 3.xの開発が本格的に始まりました"
tags = ["rxjava", "rxjava3"]
blogimport = true
type = "post"
draft = false
+++

現状での差異をまとめておきます。

## RxJava2との差異

### READMEから

[README.md](https://github.com/ReactiveX/RxJava/blob/3.x/README.md)

- fixed API mistakes and many limits of RxJava 2
    - RxJava2のいくつかのAPIのミス、制限を直している
- intended to be a replacement for RxJava 2 with relatively few binary incompatible changes
    - RxJava2から、多少の変更がある
- test and diagnostic support via test schedulers, test consumers and plugin hooks
    - テストのサポートの充実

### 3.x different docsから

[3.x different docs](https://github.com/ReactiveX/RxJava/blob/3.x/docs/What's-different-in-3.0.md)

#### asメソッドとtoメソッド

toメソッドはFunction型を引数から取っていた。しかし、あらゆるReactive型でFunction型を受け取っていたので、共通のConverterを作ることが出来なかった。

→ そこで、asメソッドが誕生

asメソッドでは、CompletableConverter、ObservableConverterなど、専用のインターフェース型になったので、1つのクラスに実装できるようになった

→ autodisposeみたいな、ライブラリを作るときに便利

従来のtoメソッドは消えて、RxJava 3ではasに統合された。

#### Functional typesがThrowableをthrowするようになった

今まではCallableを使っていた。

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
これからはSupplierを使う。

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

throwする例外がException -> Throwableに広がった。

#### startWithメソッド

startWith(T)、startWith(Iterable)、 startWith(Publish)の同名で3つのメソッドがあったが、startWithItem、startWithIterableにそれぞれリネームされた。

## メモ

- continued support for Java 6+ & Android 2.3+
    - RxJava 2.xとサポートの範囲は変わらないっぽい🎉
- 12月に正式版のリリースっぽい
