+++
date = "2018-12-04T00:00:00Z"
title = "（仮）Architectureまとめ"
tags = ["android", "architecture"]
blogimport = true
type = "post"
draft = true
+++

- RxRedux

## RxRedux

[RxRedux](https://github.com/freeletics/RxRedux)

{{< figure "https://raw.githubusercontent.com/freeletics/RxRedux/master/docs/rxredux.png" width=360 >}}

- Activity、Fragmentの継承: なし
- Kotlin: Kotlinファースト
- ライブラリ依存: RxJava
- 似ている他ライブラリ: mobius

名前の通り、Reduxパターン補佐するライブラリです。Action -> (side effects) -> Reducer -> Stateとイベントが流れます。
View層からActionを投げて、Redux層でActionをReducerでStateに変換します。
投げられたActionに副作用がある場合はside effectとして登録が可能です。

簡単なサンプルコードは次になります。

```kotlin
val actions : Observable<Action> = ...
val sideEffects : List<SideEffect<State, Action> = listOf(::loadNextPageSideEffect, ... )

actions
  .reduxStore( initialState, sideEffects, ::reducer )
  .subscribe( state -> view.render(state) )
```

## workflow

[workflow](https://github.com/square/workflow)

- ライブラリ依存: Coordinator

## mobius

- プロダクションで使われている
