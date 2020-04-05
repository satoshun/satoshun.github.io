+++
date = "2017-05-04"
title = "RxJava 並行でリクエストをして, リクエストした順番で値を受け取る"
tags = ["android", "rxjava"]
blogimport = true
type = "post"
draft = false
+++

## 結論

- concatEagerを使う


## concat, concatEager

RxJavaにはconcatEagerと, concatオペレータがあります. 似た名前ですが, 振る舞いは異なります.
concatEagerはconcatと違い, 並列にObservableは処理するけど, 返してくる順番は保証されています

- concat: 直列にObservableを処理し, 返してくる順番は保証されている
- concatEager: 並列にObservableを処理し, 返してくる順番は保証されている

コードで説明すると,

```java
// o1 ----> |
// o2       |------>|
//          |       |
//  s ------o1------o2
Observable.concat(o1, o2)
 .subscribe();
```


```java
// o1 ----------->|
// o2 -->---------|-|
//                | |
//  s ------------o1-o2
Observable.concatEager(o1, o2)
 .subscribe();
```

みたいな感じになります.
繰り返しになりますが ,concatEagerは同時に与えたObservableを解釈し, 順番を守ってくれます.
