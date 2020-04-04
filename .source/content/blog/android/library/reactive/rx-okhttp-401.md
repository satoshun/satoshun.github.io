+++
date = "2017-08-10"
title = "Android: RxJava + OkHttpを使って401の処理をいい感じにする"
tags = ["android", "okhttp", "rxjava"]
blogimport = true
type = "post"
draft = false
+++

(この記事はRxJavaとOkHttpを知っている前提で進めていきます。またこの記事はRxJavaすごーいしたいだけの記事です。401のハンドリングのところは、RxJavaを使いたいだけのユースケースというか1例になります。)

この記事のサンプルコードは [ここ(Github)](https://github.com/satoshun-example/authenticateSample)にあります。

---

API周りの開発をしていて、401のときの処理をどうするかという永遠の課題があると思います。
クライアント側で頑張るとしたら以下の感じになるかなと思います。

1. OkHttpのAuthenticatorを使い、そこで401のときのハンドリングをする。[公式ドキュメントのリンク](https://github.com/square/okhttp/wiki/Recipes#handling-authentication)
2. OkHttpのInterceptorを使い、そこで401のときのハンドリングをする。[公式ドキュメントのリンク](https://github.com/square/okhttp/wiki/Interceptors)
3. 各処理(例えばRxJavaのTransformerとか)に401用の処理を埋め込んで、retryを掛ける。

などがあるかなと(OkHttp、RxJavaを使うことを前提とする)

このブログではOkHttpのAuthenticatorを使う方法で401の処理をいい感じにしたいと思います。

またrefresh tokenは非同期、APIリクエストが必要であるとします。擬似コードはこんな感じです。

```
fun refreshToken(): Observable<String> {
  return Observable.just(Credentials.basic("sato", "passwordhoge"))
      .doOnNext { println("GET credential: $it") } // Log出力
      .delay(2000, TimeUnit.MILLISECONDS)
}
```

時間が掛かるんだなあくらいに思って下さい。


## 401の処理を書いていく

OkHttpのAuthenticatorは401のときにフックされます。なのでこの中で401ならrefresh tokenをし、結果をauthorization headerにセットして再リクエストをすれば良さそうです。

ただここで1つ問題があります。それは、APIリクエストは並列に行われるということです。ということは、何も考えないとrefresh tokenが複数回叩かれてしまいます。なのでrefresh tokenをしているときは、後続の401はそれを待つようにしないといけません。

愚直に表現するなら、refresh tokenをしていますよフラグをどこかに立てておいて、それがtrueなら待つ。falseならrefersh tokenをして、結果を待つといった処理になります。
並列にリクエストが行われるとしたら、マルチスレッドからコールされるのでsynchorizeなどの制御をする必要があり、めっちゃムズい。


## RxJavaを使ってみる

上記の縛りをRxJavaのpublish + refCountを使うとめっちゃいい感じに表現できます。

```kotlin
private val authenticator: Authenticator = object : Authenticator {

  // ここがポイント。publishとrefCountを使う。
  private val tokenStream = refreshToken().publish().refCount()

  // これだと駄目
  // private val tokenStream = refreshToken()

  // ここは並列に呼ばれる
  override fun authenticate(route: Route, response: Response): Request? {
    val credential = tokenStream.blockingFirst()
    return response.request().newBuilder()
        .header("Authorization", credential)
        .build()
  }
}
```

これだけです。synchronizeもフラグも出てこないコードになります。それでいて、並列に401リクエスト(authenticateメソッド)が呼ばれてもrefresh tokenリクエストは1つしか呼ばれません。


## なんでか?

`publish` はConnectableObservableを作り出すオペレータです。Connectableは複数のObserverで同じデータを受け取るための機能です。`publish`を使うことで共通のデータが使え、今回の場合でいうと最初の1回だけ refresh tokenリクエストを送り、後続はそのリクエストを投げているObservableを見ることになります。これだけでは不十分で `refCount`などのConnectableObservable専用のオペレータを組み合わせる必要があります。

`refCount`は全てのObserverがdisposeされたら初期化され、少なくとも1つのObserverがいるときは後続は再利用するオペレータです。なので今回の場合には、少なくとも1つの401があるときはrefresh tokenを投げてくれるので上手く動作します。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/7700/3d4082df-9da0-e344-9853-7d9a222bc89a.png)

出典: http://reactivex.io/documentation/operators/refcount.html

1つのObservableストリームを共通で使っているのがわかります。


## まとめ

- RxJava + publish + refCountすごーい
- [サンプルコード](https://github.com/satoshun-example/authenticateSample)を読むと理解が深まると思います(ΦωΦ)
- もっと良い書き方があったら教えてちょ
