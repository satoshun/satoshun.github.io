+++
date = "Mon Jan 28 00:08:35 UTC 2019"
lastmod = "Mon Jan 28 22:58:03 UTC 2019"
title = "R8/Proguard: JarファイルからProGuard設定ファイルを読み込んでくれるようになりました"
tags = ["android", "r8", "proguard"]
blogimport = true
type = "post"
+++

~~AGP3.3.0~~ProGuardはAGP3.3.0、R8は導入されたAGP3.2.0から、JarファイルのProGuard設定ファイルを組み込めるようになりました。その機能紹介です。
今まで、aarでは`consumerProguardFiles`で、ライブラリのProGuard設定を指定できました。それのJar、Javaバージョンとなります。

## ライブラリ開発者側の設定

RetrofitなどのSquare社のライブラリでは、早くもこの機能に対応しているので、それを例にして説明します。

まず、`resources/META-INF/prougard`ディレクトリの中にProGuardの設定ファイルを置きます。

[square/retrofit](https://github.com/square/retrofit/tree/master/retrofit/src/main/resources/META-INF/proguard)

ライブラリ側の設定はこれで完了です。

## 使う側の設定

AGP3.3.0にアップデートするだけで使えます。META-INF/ProGuardはRetrofitの2.5.0から入っているので、まずはMETA-INFが入っていない、2.4.0でビルドをしてみます。

```
implementation "com.squareup.retrofit2:retrofit:2.4.0"

> ./gradlew installRelease
...
Warning: there were 267 unresolved references to classes or interfaces.
         You may need to add missing library jars or update their versions.
         If your code works fine without the missing classes, you can suppress
         the warnings with '-dontwarn' options.
         (http://proguard.sourceforge.net/manual/troubleshooting.html#unresolvedclass)
Warning: Exception while processing task java.io.IOException: Please correct the above warnings first.
Thread(Tasks limiter_2): destruction
```

失敗しました😂

次にMETA-INFが入った2.5.0でビルドをします。

```
implementation "com.squareup.retrofit2:retrofit:2.5.0"

> ./gradlew installRelease
...

BUILD SUCCESSFUL in 21s
38 actionable tasks: 18 executed, 20 from cache
```

成功しました😊

META-INFファイルをちゃんと読み込めているようです。ProGuardのconfigurationファイルを確認したところ、RetrofitのProGuard設定が入っていました。

```
// configuration.txt
...
# Retain service method parameters when optimizing.
-keepclassmembers,allowshrinking,allowobfuscation interface  * {
    @retrofit2.http.*
    <methods>;
}
...
```

## 補足

この機能はR8/ProGuard、両方とも対応しているようです。

## 追記

最初、冒頭でR8もAGP3.3.0からと書いたんですが、それは誤りで、AGP3.2.0のR8が導入されたタイミングでした。

[@kafumi__](https://twitter.com/kafumi__/status/1089816485386747905)さんにご指摘いただきました！ありがとうございます😊

## まとめ

- ライブラリがJarであったとしても、ライブラリ作者が対応してくれればProGuardの設定が楽になる!!!
    - 現状、有名なライブラリでは、OkHttp、Retrofit、Coroutineなどが対応しています😊
- この機能がProGuardでも使えるのを知らなかったので、公式ドキュメントなどのリンクを知っている方がいれば教えてほしいです🙏🙏🙏
