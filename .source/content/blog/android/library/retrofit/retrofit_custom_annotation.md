+++
date = "2018-11-23T00:00:00Z"
title = "Retrofitでカスタムアノテーションを使う"
tags = ["android", "retrofit"]
blogimport = true
type = "post"
draft = false
+++

[Retrofit 2.5.0](https://github.com/square/retrofit/blob/master/CHANGELOG.md#version-250-2018-11-18)からカスタムアノテーションが使えるようになったので、それの紹介です。

例をあげて説明します。特定のリクエストのヘッダーに認証情報を付与したいとします。

まず最初にアノテーションを定義します。

```kotlin
annotation class RequireAuth
```

次に、上記で定義したアノテーションを使い、Apiを定義します。

```kotlin
interface ApiService {
  @RequireAuth
  @GET("login")
  fun login(: retrofit2.Call<Unit>
}
```

次に、`RequireAuth`を処理するための`okhttp3.Interceptor`を定義します。

```kotlin
class AuthInterceptor : Interceptor {
  override fun intercept(chain: Interceptor.Chain): Response {
    var request = chain.request()

    val invocation = request.tag(Invocation::class.java)
    val authAnnotation = invocation?.method()?.getAnnotation(RequireAuth::class.java)
    if (authAnnotation != null) {
      request = request
        .newBuilder()
        .addHeader("Authorization", "Basic AAAAA").build()
    }
    return chain.proceed(request)
  }
}
```

ここでのポイントは、`val invocation = request.tag(Invocation::class.java)`です。
Retrofit 2.5.0から`Invocation`が追加され、`Request`から`Invocation`が取得できるようになりました。
`Invocation`には、処理している`Request`の`java.lang.reflect.Method`が格納されており、
そこからアノテーションの情報を取得することができます。

`val authAnnotation = invocation?.method()?.getAnnotation(RequireAuth::class.java)`で、
メソッドに`RequireAuth`アノテーションが付与されているかどうかを知ることが出来ます。
`RequireAuth`アノテーションがついていれば、`Request`のヘッダーに認証情報を追加します。

最後に、上記の`Interceptor`をOkHttpクライアントに付与します。

```kotlin
val client = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor())
    .build()
```

これでRetrofitでカスタムアノテーションを使うことが出来ます!!

## Invocation以前の場合

Invocation以前は、`Headers`を使い認証情報を付与するテクニックがありました。
詳しくは[Making Retrofit Work For You](https://speakerdeck.com/jakewharton/making-retrofit-work-for-you-ohio-devfest-2016?slide=39)にあります。

```kotlin
@Headers("Auth: true")
@GET("useheaderlogin")
fun login(): retrofit2.Call<Unit>
```

```kotlin
class Auth2Interceptor : Interceptor {
  override fun intercept(chain: Interceptor.Chain): Response {
    var request = chain.request()

    if (request.header("Auth") != null) {
      request = request
        .newBuilder()
        .addHeader("Authorization", "Basic BBBBB").build()
    }
    return chain.proceed(request)
  }
}
```

このコードでも動くのですが、カスタムアノテーションを定義するやり方のほうが意味が伝わりやすいと思うので、よりよいと思います。

## まとめ

- Retrofit 2.5.0でInvocationが追加されてカスタムアノテーションが使えるようになりました
  - さらにInvocationはメソッドの引数リストを持っており、ログやアナリティクスなどに有効に使うことができます

今回の検証に用いたサンプルコードは[ここに](https://github.com/satoshun-android-example/RetrofitCustomAnnotationExample)あります😃
