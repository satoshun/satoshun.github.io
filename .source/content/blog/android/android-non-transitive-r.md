+++
date = "Sun Nov 14 01:58:16 UTC 2021"
title = "Android: non-transitive Rの導入と、ビルド時間の変化について"
tags = ["android", "build", "gradle"]
blogimport = true
type = "post"
draft = false
+++

non-transitive Rというオプションがあり、これを有効にすると、マルチモジュールなプロジェクトでのビルド時間の改善が期待できます。

この記事では、non-transitive R対応の仕方と、どれくらいビルド時間が向上したかについて紹介したいと思います。

## non transitive R対応の仕方

比較的新しいAndroid Studio（4.2以降?）を使っているなら、Refactor機能から変換することが出来ます。

**`Refactor > Migrate to Non-transitive R Classes.`**

この機能を使うと、各モジュールのRファイル参照が、フルパッケージ名参照に置換されます。

擬似コードだとこんな感じです。

```kotlin
// before
import jp.hoge.sample.R

class HogeFragment(R.layout.hoge_layout) {
}

// after
class HogeFragment(jp.hoge.sample.R.layout.hoge_layout) {
}
```

non-transitve Rを有効にすると、各モジュールで、Rファイルのマージ?のようなタスクが行われなくなり、Rの参照先が変わるためです。

動作自体は上記のコードでも問題なく出来ます。ただ、フルパッケージ指定が気になるなら、import aliasまたは、typealiasを使うことで解決できます。

擬似コードだとこんな感じです。

```kotlin
// import alias
import jp.hoge.sample.R as SampleR

class HogeFragment(SampleR.layout.hoge_layout) {
}

// typealias
typealias SampleRLayout = jp.hoge.sample.R.layout

class HogeFragment(SampleRLayout.hoge_layout) {
}
```

non-transitive R対応はこれで完了です。

## どれくらい速度向上があったか?

前提として、CI上で計測したので、ローカル環境とは異なります。

計測には、Gradle-Profilerを使い、applicationモジュールのファイルが変更された時の、インクリメンタルビルド時間の変化について、計10回反復し測定しました。

結果、おおよそ20%程度の速度向上が見られました。R参照がフルパッケージになるなど、多少不便な点はありますが、十分に導入する価値があると感じました。

## まとめ

マルチモジュールな環境で、大きなプロジェクトなら導入するのを検討するのが良いと感じました。
また、Refactor機能が賢いので、導入にもそう手間は掛からないと思います。
