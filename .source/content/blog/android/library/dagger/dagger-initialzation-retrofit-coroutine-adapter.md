+++
date = "Fri Jan 10 13:15:11 UTC 2020"
title = "Dagger2でRetrofit用のOkHttpClientをバックグラウンドで作ったときにうまくいかない場合がある"
tags = ["android", "dagger", "startuptime", "retrofit", "di"]
blogimport = true
type = "post"
draft = true
+++

[Dagger Party Tricks: Deferred OkHttp Initialization](https://www.zacsweers.dev/dagger-party-tricks-deferred-okhttp-init/)を参考にして、バックグラウンドでOkHttpClientインスタンスを生成しようとしたけど、メインスレッドで生成されてしまった話をします。

結論から言うと、`RxJava2CallAdapterFactory`を使うと、OkHttpClientを生成する部分が、IOスレッドで呼び出されるので、バックグラウンドで生成されます。この場合は問題ありません。

しかし、Coroutineを使っている場合は、OkHttpClientを生成する部分が、呼び出したスレッドに依存します。メインスレッドから呼び出したら、メインスレッドでOkHttpClientが生成されてしまいます。

今回は、Coroutineを使っていたため、メインスレッドで生成されてしまいました。

## なんでこんな動作になるのかコードを追ってみる。

まず最初にサンプルコードになります。

```kotlin
// MainActivity
@Inject lateinit var gitHubApi: GitHubApi

lifecycleScope.launch { // ktxのlifecycleScope関数
  gitHubApi.test()
}

// GitHubApi
interface GitHubApi {
  @GET("hoge")
  suspend fun test()
}
```

`gitHubApi.test()`からコードを追っていきます。

まず、`gitHubApi.test()`はメインスレッド上で実行されます。これは`lifecycleScope`が`Dispatchers.Main.immediate`に紐付けられているためです。この
