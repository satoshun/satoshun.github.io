+++
date = "Tue Jul  2 13:46:54 UTC 2019"
title = "Retrofit: Coroutineを使うときの、Response<T>と生のTの使い分け"
tags = ["android", "retrofit", "coroutine"]
blogimport = true
type = "post"
draft = false
+++

psideさんの、[Retrofit2でRxJavaを使う時の Result<T>, Response<T>, そのままT の使い分け所感](https://qiita.com/pside/items/e546f0f12989e8dcd729)のCoroutineバージョンの記事となります。

上記の記事に書いてある、Rxとは違い、Result型は用意されていないので、Response型で包むか、生で値ｗ受け取るかのどちらかが基本となります。

```kotlin
interface HogeService {
    suspend fun getHoge(): Hoge
    or
    suspend fun getHoge(): Response<Hoge>
}
```

Retrofitの2.6.0で、挙動の違いを確認しました。

|| 生 | Response<T>  |
|---|---|---|
| 200 | 成功 | 成功 |
| 404 | 例外 | 成功 |
| ネットワークに繋がっていない | 例外 | 例外 |
| シリアライズが出来ない（型がおかしい）| 例外 | 例外 |

- 生の場合、HTTPのstatus Code的に失敗とされるものは例外になる
- Response型で包めば、HTTPのstatus Code的に失敗だとしても例外が発生しない

っていう感じの挙動になります。

## どっちを使えばいいの?

サーバがエラーコードを返してきた時に、特別な振る舞いをしたいエンドポイントってあると思うので、そういうときはResponseで包んであげて、それ以外は生でいいんじゃない？って思ってます（小並感
