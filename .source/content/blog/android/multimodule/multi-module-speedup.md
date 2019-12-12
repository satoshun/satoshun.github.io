+++
date = "Mon Feb 11 01:40:29 UTC 2019"
lastmod = "Mon Feb 11 02:09:16 UTC 2019"
title = "でかいappモジュールがあるときに、中間モジュールを入れることで差分ビルドを上手く効かせる"
tags = ["android", "multimodule", "gradle", "build"]
blogimport = true
type = "post"
thumbnail = "https://bit.ly/2DqyKVb"
+++

マルチモジュール構成のメリットに差分ビルドの効率化というものがあります。しかし、モノシリックなappモジュールから、マルチモジュール構成に変更していく過程ではappモジュールがでかいままなので、差分ビルドによる恩恵が受けにくいという問題があります。（最終段階まで進めばappモジュールは十分に小さくなるので、差分ビルドの恩恵を受けられます）

例えば、以下のモジュール構成を考えます。

{{< figure src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLusJ3boOwrBnlxdg_evk9ApiyjoCzBpIjHY7xSkEznum8OkVnnO_VZnfR4WeB7pOiUD-rutBpqSVEUnyshOnKIYnM0mYXQcxO-RfvcY4rbSMcI8QPI8nnAZRYuk81cA-Ycv9VdwTf1TAC90DKufEQb0Bq40000" width="400" >}}

頑張って2つのモジュールを切り出しました。ただし、これではどこのモジュールを変更してもかなりのビルド時間がかかります。なぜなら、Gradleでは依存関係にあるモジュールが変更されたときに、自分自身も（ある程度?）再ビルドされるためです。なので、上記のモジュール構成だと、どこのモジュールを修正しても、常に大きいappモジュールが再ビルドされてしまうため、ビルド時間がかかってしまいます。

そこで、間に中間モジュールを挟むテクニックを紹介します。このテクニックを使うと以下のようになります。

{{< figure src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLusJ3boOwrBnlxdg_evk9ApiyjoCzBpIjHY7xSkEznum8OkVnnO_VZnfR4WeB7pOiUD-rutBpqSVEUnyshOnKIYnM0miXQGGPEcunDOMPnQHAA9KrR7pTFCyIc5AZI45Ef4GwbHbnSN41NAEYcv9VdwTf1BE82aN0Xi87e8a1z3gbvAS000G00" width="400" >}}

途中に適当なモジュールを挟むことで、サブ1、サブ2が変更されたときにappモジュールの再ビルドを防ぐことができます。

ただし、いくつか条件があります。

### 1. 中間モジュールで公開可能なものに限る

例えばサブ1でSubActivityを公開していて、これを直接appから参照している場合は駄目です。
これをSubActivityとしてではなく、Activityとして参照できるなら大丈夫です。サブ1モジュールで定義されているクラスがappモジュールから見れないための制約です。

中間モジュールのコードイメージとしては以下のようになります。

```kotlin
fun createUserFragment(userName: String, age: Int): Fragment {
  return UserFragment.createFragment(userName, age)
}

fun createUserIntent(context: Context): Intent {
  return Intent(context, UserActivity::class.java)
}
```

UserActivity、UserFragmentが公開されていないことが分かります。Androidのいわゆるfeatureモジュールでは、Activity、Fragmentを公開する場合が多いと思うので、その場合には有効に使うことができます。

### 2. implementationで依存を定義する

apiを使うと、依存が推移するため再ビルドが行われてしまうためです。implementationで依存を記述する必要があります。

### 3. Dagger2使ってると多分無理

Dagger2では、解決する依存をAppComponentで知っている必要があります。上記の構成だと、appでAppComponentを持つことになるので、appからsub1、sub2が見えていないと最終的にDagger2で解決できません。なので、中間モジュールで、appからsubの依存が見えなくなるこのパターンは使えません。

詳しくは[Dagger/#970](https://github.com/google/dagger/issues/970)にあります。

## まとめ

- やりすぎ感はある
    - ただでさえ複雑な、モジュール構成がさらに煩雑になりそう。ただし、最終的には消えるので、差分ビルドの恩恵を受けるためのステップだとすれば許せるかも?
    - Dagger2を使っていると推移的依存が必要になり、使えない、もしくは工夫が必要になる
- サンプルは[satoshun/ApplicationModulesSpeedUpExample](https://github.com/satoshun-android-example/ApplicationModulesSpeedUpExample)にあります
    - サブモジュールを変更したときのビルドは爆速でした😊

Daggerの部分のいい解決方法を知っている人がいたら、教えて頂けると幸いです😊😊😊
