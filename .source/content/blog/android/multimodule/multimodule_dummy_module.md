+++
date = "Sat Dec 12 04:18:00 UTC 2020"
title = "マルチモジュール環境での、dummyモジュール導入によるビルドの高速化"
tags = ["android", "multimodule", "build"]
blogimport = true
type = "post"
draft = false
+++

## 最初にまとめ

各モジュール間の結合度(publicなクラス、インターフェース)を低く保ち、settings.gradleを工夫することで、最小限のコードでdummyモジュールを導入できる。dummyモジュールのコード量は限りなく少ないので、ビルド時間の短縮につながる。

サンプルは [https://github.com/satoshun-android-example/GradleDummyProject](https://github.com/satoshun-android-example/GradleDummyProject) にあります。少し本記事とモジュール名などが異なります。

## 本題

プロジェクトの規模が大きくなると、ビルド時間は増加する傾向にあります。マルチモジュールによるアプローチは、差分ビルドを効率良く動作させることが出来ます。本記事では、マルチモジュールプロジェクトに、dummyモジュールを差し込むことでビルド時間の改善をする方法を紹介します。

また、この記事では触れませんが、機能（画面）ごとにappモジュールを作るアプローチも良いと思います。

## dummyモジュールとは

例えば、注文を行うorderモジュールがあるとします。今回の開発では、その注文画面は使う必要がないとします。使う必要がないので、orderモジュールは一時的に削除しても良いことになります。しかし、単純にモジュールを削除してしまうと、何かしらの参照エラーになってしまうので、order-dummyモジュールを作り、参照エラーになるクラスを実装します。具体的には、orderモジュールで定義されているpublicなクラス、インターフェースです。

今回、orderモジュールには唯一Orderクラスのみがpublicとして定義されているとすると、次のように書きます。

```kotlin
class Order {
  fun canOrder(): Boolean { ... }
  fun order() { ... }
}
```

```kotlin
class Order {
  fun canOrder(): Boolean { TODO("dummy") }
  fun order() { TODO("dummy") }
}
```

必要に応じて、orderモジュール、order-dummyモジュールを切り替えることでビルド時間を短縮することが出来ます。

## どのようにモジュールを切り替えるか?

settings.gradleとGradleプロパティを用います。

まず、settings.gradleは次のようになります。

```groovy
if (properties["dummyorder"]) {
  include "order"
  project(':order').projectDir = file('dummyorder')
} else {
  include ':order'
}
```

dummyorderプロパティに、何かしらの値がセットされていたら、dummyorderモジュールを使う設定になります。

dummyorderプロパティの設定はコマンドラインからは `-P`、Android StudioからはCommand-Line Optionsから指定します。

{{< figure src="/blog/android/multimodule/multimodule_dummy_module_as.png" width="500" >}}

これで完了です。普段使わないモジュールをdummyに差し替えるテクニックでした。各モジュール間の結合度が低いなら、コード量少なくお手軽に導入することが出来ます。

## 雑談

まだ、この機能をプロダクションに導入していないので、もし導入したら、どれくらいの効果があったかを追記します。効果がない可能性も微レ存です。
