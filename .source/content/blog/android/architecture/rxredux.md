+++
date = "Sun Dec 16 23:39:41 UTC 2018"
title = "RxReduxの簡単な紹介"
tags = ["android", "architecture"]
blogimport = true
type = "post"
draft = true
+++

https://github.com/freeletics/RxRedux

## 概要

RxReduxはReduxを実現するためのライブラリです。

{{< figure "https://raw.githubusercontent.com/freeletics/RxRedux/master/docs/rxredux.png" >}}

次の拡張関数を提供しています。

```kotlin
fun <S : Any, A : Any> Observable<A>.reduxStore(
    initialState: S,
    sideEffects: Iterable<SideEffect<S, A>>,
    reducer: Reducer<S, A>
): Observable<S>
```

各変数、型の意味は次のとおりです。

- ジェネリック型S: Stateを表す
- ジェネリック型A: Actionを表す
- initialState: 初期値
- SideEffect: 副作用がある振る舞いをここで定義する
- Reducer<S, A>: Actionを受け取り、新しいStateを生成する

この拡張関数を使うことで、1本のRxストリームを作り「Action -> (SideEffect) -> State」の単一方向データフローを達成することが出来ます。

## 解決したい問題

状態管理を
