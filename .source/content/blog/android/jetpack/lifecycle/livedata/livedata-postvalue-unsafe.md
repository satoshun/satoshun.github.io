+++
date = "2017-12-06"
title = "LiveDataのpostValueは全て流れてくるとは限らない"
tags = ["android", "jetpack", "livedata"]
blogimport = true
type = "post"
draft = false
+++

## 結論

- LiveData.postValueを短い間に複数回コールすると最後の値しかObseverに流れてこないことがある


## 背景

RxJavaのSubjectの代替としてMutableLiveData(postValueとsetValueがpublicになっている)を使っている部分があり、短い間に2回コールされたとき、全ての値が流れて欲しかったが何故か1回しか叩かれていなかった。
どこに原因があるか調査したところ、どうもpostValueメソッドの仕様(LiveDataクラス全体の仕様?)的に短い間に複数回コールされた場合は、最後にpostValueした値のみをpostするようになっていた。


## 実装詳細

LiveDataのpostValueのコードは以下のようになっています(v1.0.0)

```java
protected void postValue(T value) {
    boolean postTask;
    synchronized (mDataLock) {
        postTask = mPendingData == NOT_SET;
        mPendingData = value;
    }
    if (!postTask) {
        return;
    }
    ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
}
```

postValueメソッドでは

2. 1. `mPendingDataフ`ィールドにObserverに渡すデータをセットする
2. メインスレッドで`mPostValueRunnable`を実行する
    (ただし`mPendingData`にまだ以前のデータが残っていたら、以前のデータを更新する)

といったことをやっています。

このコードを見て分かる通り、前にセットした`mPendingData`がObserverに渡される前に、新しい値で上書きされる可能性があることが分かります。
よって、短い間に複数回のpostValueをコールすると最後の値のみしかObserverに流れてこない可能性があります。

## まとめ

- RxJavaのSubjectのように、全ての値を流す動作を意図しているとハマる
- そもそもLiveDataは最新のViewの状態を保持する用途だと思うので、短い間に複数回コールされたら、最後の値のみを流すのは正しい。LiveDataは悪くない。悪いのは俺。
