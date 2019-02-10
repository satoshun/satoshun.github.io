+++
date = "Sun Feb 10 10:19:29 UTC 2019"
title = "でかいappモジュールがあるときに、中間モジュールを入れることで差分ビルドを上手く効かせる"
tags = ["android", "multimodule", "gradle", "build"]
blogimport = true
type = "post"
thumbnail = "https://bit.ly/2MWwKIu"
+++

マルチモジュール構成のメリットに差分ビルドの効率化というものがあります。しかし、モノシリックなappモジュールから、マルチモジュール構成に変更していく過程ではappモジュールがでかいままなので、差分ビルドによる恩恵が受けにくいという問題があります。（最終段階まで進めばappモジュールは十分に小さくなるので、差分ビルドの恩恵を受けられます）

例えば、以下のモジュール構成を考えます。

<img src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLusJ3boOwrBnlxdg_evk9ApiyjoCzBpIjHY7xSkEznum8OkVnnO_VZnfR4WeB7pOiUD-rutBpqSVEUnyshOnKIYnM0mYXQcxO-RfvcY4rbSMcI8QPI8nnAZRYuk81cA-Ycv9VdwTf1TAC90DKufEQb0Bq40000" width=400>

頑張って2モジュールを切り出しました。ただし、これではどこのモジュールを変更してもかなりのビルド時間がかかります。なぜなら、Gradleでは依存関係にあるモジュールが変更されたときに、自分自身も（ある程度?）再ビルドされるためです。なので、上記のモジュール構成だと常に大きいappモジュールが再ビルドされてしまうので、ビルド時間がかかってしまいます。

そこで、その間に中間モジュールを挟むテクニックを紹介します。このテクニックを使うと以下のようになります。

<img src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLusJ3boOwrBnlxdg_evk9ApiyjoCzBpIjHY7xSkEznum8OkVnnO_VZnfR4WeB7pOiUD-rutBpqSVEUnyshOnKIYnM0miXQGGPEcunDOMPnQHAA9KrR7pTFCyIc5AZI45Ef4GwbHbnSN41NAEYcv9VdwTf1BE82aN0Xi87e8a1z3gbvAS000G00" width=400>

途中に適当なモジュールを挟むことで、サブ1、サブ2が変更されたときにappモジュールの再ビルドを防ぐことができます。

ただし、いくつか条件があります。

1. 中間モジュールで公開可能なものに限る

例えばサブ1でSubActivityを公開していて、これを直接appから参照している場合は駄目です。
これをSubActivityとしてではなく、Activityとして参照できるなら大丈夫です。サブ1モジュールで定義されているクラスをappモジュールから見れないための制約です。

2. implementationで依存を定義する

apiを使うと、依存が推移するため再ビルドが行われてしまうためです。implementationで記述する必要があります。

3. Dagger2使ってると多分無理

Dagger2では、解決する依存の関係を知っていないといけないためです。この構成だと、appからsub1、sub2が見えてないと最終的にDagger2で解決できないです。
詳しくは[Dagger/#970](https://github.com/google/dagger/issues/970)にあります。

## まとめ

- 使い所あるかもしれない?
    - ただDagger2を使っていると推移的依存が必要になり、使えない、もしくは工夫が必要になる
- やりすぎ感はある
    - モジュール構成が煩雑になりそう。ただ最終的には消えるので、差分ビルドの恩恵を受けるための途中段階だとすれば許せる?

Daggerの部分のいい解決方法を知っている人がいたら、教えて頂けると幸いです😊😊😊
