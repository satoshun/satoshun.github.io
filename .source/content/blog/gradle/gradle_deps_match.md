+++
date = "2018-12-17"
title = "GradleのMatching repositories to dependenciesを使って、リポジトリのmavenにfilterをかける"
tags = ["gradle", "android", "dependency"]
blogimport = true
type = "post"
draft = true
+++

本当は`JitPack`からライブラリをインストールしたかったところ、jcenterからライブラリをインストールしてしまう問題がありました。
詳しくは次のリンクを参照してください。[A Confusing Dependency](https://blog.autsoft.hu/a-confusing-dependency/)

従来のGradle4系ではおそらく、上記の問題を解決することは出来ない、もしくは非常に困難でした。しかし新しくGradle5.1に`Matching repositories to dependencies`が導入され、上記の問題を解決できます。

使い方は簡単です。従来の書き方をまず紹介して、次に新機能を使った書き方を紹介します。今回は例として、`cloudflare`のSDKを依存関係に入れることを目指します。

まずは従来の書き方です。

```groovy
// topのbuild.gradle
allprojects {
    repositories {
        ...

        maven {
            url "https://storage.googleapis.com/cloudflare-maven/public/"
        }
    }
}

---

// projectのbuild.gradle
dependencies {
    ...

    implementation "com.cloudflare:cloudflare-mobile-sdk:2.1.0@aar"
}
```

これだと全てのライブラリに対して `https://storage.googleapis.com/cloudflare-maven/public/`のチェックをしにいきます。このUrlはcloudflareのライブラリにしか使われないことが想定されるので、
