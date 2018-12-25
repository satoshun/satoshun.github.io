+++
date = "2018-08-21T00:00:00Z"
title = "RxAndroidのasyncパラメータを試してみた"
tags = ["android", "rxandroid", "rxjava"]
blogimport = true
type = "post"
+++

RxAndroid 2.1.0で新しくasyncパラメータが追加されました。これは`Message#setAsynchronous`を使うことで、UIのパフォーマンス向上を狙った機能です。
下のリンクに詳細な内容が書かれています。

[RxAndroid’s New Async API](https://medium.com/@sweers/rxandroids-new-async-api-4ab5b3ad3e93)

この記事では、asyncがfalseの場合とtrueの場合でどれくらいの差が出るかを検証してみました。

検証に使用したサンプルプロジェクトは以下になります。
https://github.com/satoshun-android-example/RxAndroidExample

サンプルプロジェクトをかいつまんで説明します。

まず2つのスケジューラを作成し、

```kotlin
private val mainScheduler = AndroidSchedulers.from(Looper.getMainLooper(), false)
private val asyncMainScheduler = AndroidSchedulers.from(Looper.getMainLooper(), true)
```

作ったスケジューラを使ったストリームで実行完了時間に差が出るかを試してみました。

```kotlin
// asyncがfalseの場合
Observable
    .fromCallable { System.currentTimeMillis() }
    .delay(index, TimeUnit.MILLISECONDS)
    .observeOn(mainScheduler)
    .subscribe(...)

// asyncがtrueの場合
Observable
    .fromCallable { System.currentTimeMillis() }
    .delay(index, TimeUnit.MILLISECONDS)
    .observeOn(asyncMainScheduler)
    .subscribe(...)
```

フルコードは以下になります。
https://github.com/satoshun-android-example/RxAndroidExample/blob/master/app/src/main/java/com/github/satoshun/example/rxandroidexample/MainActivity.kt

結果は、以下のようになりました。

```
// API27 エミュレータ
main=130988ms, async=126713ms　// forで500回ループさせた実行時間の総和
main=130857ms, async=126582ms
main=131401ms, async=126909ms
main=130763ms, async=126504ms
main=132758ms, async=127972ms

// API21 エミュレータ
main=129869ms, async=125795ms
main=130050ms, async=125888ms
main=129935ms, async=125853ms
main=129908ms, async=125838ms
main=129927ms, async=125824ms
```

asyncがtrueの場合、明らかに実行完了時間が短くなりました。導入するメリットがありそうです。

ただ注意点として、この機能は副作用がある可能性があります(なのでデフォルトではasyncはfasle)。ただ、Uberで1年間、プロダクションで運用したところ、特に大きな問題は起きなかったらしいので  https://github.com/ReactiveX/RxAndroid/pull/416 、ほぼ安全と考えて良さそうです。

## まとめ

- サンプルコードで検証した結果、async=trueでパフォーマンスの向上が得られそう
- 導入は以下のコードを追加するだけなので、とても簡単

```kotlin
RxAndroidPlugins.setInitMainThreadSchedulerHandler {
    AndroidSchedulers.from(Looper.getMainLooper(), true)
}
```
