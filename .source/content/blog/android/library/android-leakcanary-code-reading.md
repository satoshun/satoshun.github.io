+++
date = "2016-10-16"
title = "LeakCanary: ソースコードリーディング"
tags = ["android", "leakcanary"]
blogimport = true
type = "post"
draft = false
+++

LeakCanaryはメモリリークを発見してくれるライブラリです.

どうやってメモリリークを検知しているのかが気になったのでまとめ.


## おおまかな流れ

1. Application#registerActivityLifecycleCallbacksを使い, ActivityのonDestroyされたときにコールされる`onActivityDestroyed`をリッスンする
2. `onActivityDestroyed`がコールされたら, onDestroyされたActivityへの参照がなくなっているかをチェックする
3. refrenceが残っていたら(メモリリーク!), どこから参照されているかを探す
4. 参照先が探せたら, notificationを作成し, メモリリークしていることを通達する
5. notificationがクリックされたら, どこが原因でメモリリークが発生しているかを表示する

という流れになっています. 肝は2, 3だと思うのでそこがどのように実装されているかを説明します.


## onDestroyされたActivityへの参照がなくなっているかをチェックする

これには`ReferenceQueue`を利用します.
`ReferenceQueue`は登録したオブジェクトがGarbage Collectorにより解放されたかどうかを調べるクラスです.
これにonDestroyがコールされたActivityを登録し, GCにより解放されたかどうかを検知することでメモリリークを調べることを出来ます.

ここで, `ReferenceQueue`がGCを検知しなかったら, どこからの参照が原因でGCが呼ばれないのかを探します.


## どこから参照されているかを探す

どこからActivityが参照されているかを探すために, まず現在のheapのdumpを取ります. heapのdumpは `Debug.dumpHprofData(/your/file/path);`で取れます.

それを `https://github.com/square/haha`を使ってどこからActivityが参照されているかを探索します.
HAHAは, hprofファイルから木構造を作成し, 各オブジェクトがどのような参照関係になっているかを表現します.
HAHAを使い, Activityを起点とし, それの親の参照, 親の参照, ...と探索することでrootの参照を探します.

あとは, 4, 5でrootの参照を通知します.


## まとめ

ざっくりとLeakCanaryの説明をしました. HAHAはhprofの読み込みをいい感じにしてくれるだけなので, 肝は`ReferenceQueue`だと思います.

`ReferenceQueue`はなかなか使う機会は無いと思いますが, 覚えておくといつか得するかもしれません(適当)
