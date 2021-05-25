+++
date = "Tue May 25 12:39:26 UTC 2021"
title = "Android: Dagger Hiltで推移的な依存を解決する"
tags = ["android", "dagger"]
blogimport = true
type = "post"
draft = false
+++

従来のDaggerだと、app(root)モジュールは、すべてのクラスパスが見えている必要がありました。

例えば、

`|appモジュール| -> |dataモジュール| -> <retrofitライブラリ>`

のような関係だと、appモジュールでもretrofitライブラリが見えている必要があります。
なので、appモジュールに `implementation retrofit` を記述するか、もしくは、dataモジュールに `api retrofit` を記述する必要があります。

Dagger Hiltの2.31以降に、この問題を解決する Classpath Aggregation が追加されたので、それを紹介します。

## Classpath Aggregation

Classpath Aggregationを有効にするには、次の設定をGradleに記述します。

```
hilt {
  enableExperimentalClasspathAggregation = true
}
```

これを追加することで、`implementation` から `api` にしなくても、コンパイルを成功させることが出来ます。

便利なフラグですが、最新のv2.35だと、パフォーマンスに影響が出るので、規模が大きいプロジェクトの場合には注意して導入する必要があります。
今後、改善予定だそうです。

## 参照

- https://dagger.dev/hilt/gradle-setup#classpath-aggregation
