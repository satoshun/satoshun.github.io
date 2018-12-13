+++
date = "Thu Dec 13 11:16:06 UTC 2018"
title = "GradleのMatching repositories to dependenciesを使ってライブラリのダウンロード先を指定する"
tags = ["gradle", "android", "dependency"]
blogimport = true
type = "post"
draft = false
+++

`JitPack`からライブラリをインストールしたかったところ、jcenterからライブラリをインストールしてしまう事件がありました。
詳しくは次のリンクを参照してください。[A Confusing Dependency](https://blog.autsoft.hu/a-confusing-dependency/)

従来のGradle4系ではおそらく、上記の問題を解決することは出来ない、もしくは非常に困難でした。しかし新しくGradle5.1に`Matching repositories to dependencies`が導入され、上記の問題を解決できます。（Gradle5.1はまだrcです）

今回の検証にはGradle 5.1-rc-1を使いました。[サンプルコードはここにあります](https://github.com/satoshun-android-example/GradleDependencyMatchingExample)

まず最初に従来の書き方を説明して、次に新機能を使った書き方を紹介します。今回は例として、`cloudflare`のSDKを依存関係に入れることを目指します。

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

これだと全てのライブラリに対して `https://storage.googleapis.com/cloudflare-maven/public/`のチェックをしにいきます。このUrlはcloudflareのライブラリにしか使われないことが想定されるので、他のライブラリに対しては制限をかけたいところです。

次に新機能を使った書き方です。

```groovy
// topのbuild.gradle
allprojects {
    repositories {
        ...

        maven {
            url "https://storage.googleapis.com/cloudflare-maven/public/"
            content {
                // group idがcom.cloudflareのライブラリだけこのURLが有効になる
                includeGroup "com.cloudflare"
            }
        }
    }
}

---

// projectのbuild.gradleは一緒
dependencies {
    ...

    implementation "com.cloudflare:cloudflare-mobile-sdk:2.1.0@aar"
}
```

新しくcontentブロックが追加されました。ここでgroup idを指定することが出来ます。今回のURLは`com.cloudflare:cloudflare-mobile-sdk`でのみ有効になって欲しいので、`includeGroup "com.cloudflare"`と指定することで達成できます。これでcloudflareのライブラリに対して制限をかけられます!!

また今回の事件の場合、jcenterに悪意のあるライブラリがアップロードされたのが問題なので、jcenterからダウンロードしたいライブラリのgroup idを`includeGroup`に追記してあげれば良いです。

```groovy
repositories {
    ...
    jcenter().mavenContent {
        includeGroup "org.jetbrains.kotlin"
        includeGroup "org.jetbrains.kotlinx"
        includeGroup "com.google.dagger"
        includeGroup "org.jetbrains"
        includeGroup "javax.inject"
        ...
    }
}
```

これで、jcenterからダウンロードするライブラリに制限をかけられます😃

## 補足

AndroidだとGradle5系を使えるのがAndroid Gradle Plugin3.4からになると思うので、まだ先は長い😂
