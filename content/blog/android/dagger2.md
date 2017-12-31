+++
date = "2015-05-08T00:00:00Z"
title = "Android: Dagger2でDIをする. 基本編 Part1"
tags = ["android", "dagger"]
blogimport = true
type = "post"
+++

## 概要

この記事では, 最初にDIとは何かについて説明します. DIを理解した後にDagger2の基本的な使い方を紹介します.
Dagger2はDI(Dependency Injection)をするライブラリです.


## DIとは?

DIとはDependency Injectionの略で, 訳すと「依存性の注入」です.
ここでいう依存とは `クラス同士の依存関係`のことを表します.
クラス同士の依存関係は, 委譲パターンの時に現れます.

例えば, 以下のコードがあったとします.

```java
class Blogger {
    public Blogger() {
    }

    public void post(String title) {
        // 何かメインの処理
        // ...

        // fileにlogを取る
        FileLogger logger = new FileLogger(title);
        logger.logging();
    }
}

class FileLogger implements Logger {
    @Override
    public void logging(String... messages) {
        // fileにmessagesを書き出す
        File file = new File("hoge.txt");
        ...
    }
}

interface Logger {
    void logging(String messgae);
}
```

`クラスBloggerはクラスFileLoggerに依存している`. ことが分かります. なぜなら, 関数post内でFileLoggerクラスを利用しているためです.
この書き方だと, BloggerクラスはFileLoggerに依存しているため, Loggerインターフェースを実装した旨味がありません.

次に, 少しコードを変更して, Loggerインターフェースを実装したインスタンスを引数で渡せるように変更します.

```java
class Blogger {
    Logger logger;

    public Blogger(Logger logger) {
        this.logger = logger;
    }

    public void post(String title) {
        // 何かメインの処理
        // ...

        // logを取る
        logger.logging();
    }
}
```

こうすることで外部から引数で, Loggerインターフェースを実装した, FileLoggerクラス等を注入できるようになります. これがDI(依存性の注入)パターンです.

これの何が嬉しいんでしょうか? それは, Bloggerクラスを変更せずに, logの取り方を変えることが出来る点です!　委譲する先のクラス(Logger)を簡単に変更することが出来ます.

余談ですが, AngularJSを使ったことのある方なら`function($scope, $resources) {}`のようなコードを見たことがあると思います. これも典型的なDIパターンになります.

DIが便利なのは分かっていただけたと思います. しかし, いちいち引数にインスタンスを指定するのはちと面倒です.
なので, DIを補佐してくれるライブラリを使うのが一般的です. その中でもAndroidで若手有望株Dagger2について説明します.


## Dagger2概念

Dagger2は大きく分けて, `Component`, `Module` `Provide`, `Inject` の4つの要素があります.
上記を定義しクラス間の依存関係を解決します.

- @Inject: 依存を注入します.
- @Module: インスタンスをProvideするメソッド郡を定義します.
- @Provide; 依存を解決するためのインスタンスを提供します.
- @Component: @Injectと@Module間の関係を定義します.

これだけでは分からないと思うので, 具体的な例をあげて説明します.


## 依存関係の定義

最初に依存関係を定義します. `javax.inject.Inject`アノテーションを使います.

上記のBloggerクラスに適用してみます. Field Injection, Constructor Injectionの2種類の定義方法があります.

```java
// inject field directly
class Blogger {
    @Inject Logger logger;

    public Blogger() {
    }
}

or

// Constructor Injection
class Blogger {
    Logger logger;

    @Inject
    public Blogger(Logger logger) {
        this.logger = logger;
    }
}
```

これで, 依存関係の定義は完了です. Field, Constructor Injectionの両方の書き方を覚えて下さい.


## Moduleの作成

Moduleには, 上記の`@Inject`で定義したクラスのインスタンスの生成方法を記述します.
`@Module`と`@Provides`アノテーションを使い, 定義します.

Loggerを提供するModuleを作成します.

```
@Module
class DebugModule {
    @Provides
    Logger provideLogger() {
        return new FileLogger();
    }
}
```

これで, Loggerを提供するModuleの定義が出来ました.

最後にComponentを作成します.


## Componentの作成

Componentは複数Moduleをまとめたものです.

```java
@Component(modules = DebugModule.class)
interface ApplicationComponent {
    Blogger make();
}
```

これで, 依存関係を解決してBloggerを生成することが出来ます.

アプリ側では以下のように記述します.

```java
ApplicationComponent component = DaggerApplicationComponent.create();
Blogger blogger = component.make();
blogger.post();
```

これが基本的なDagger2の使い方です. まとめると

- @Injectで, 依存関係を定義する
- @Module, @Providesで依存関係を解決するようにインスタンスを提供する
- @Componentで複数のModuleを用いて, 依存解決Graphオブジェクトを生成する
- アプリで依存解決Graphを使い, DIを行う


## 感想

DIのとはなんぞやということと, Dagger2の基本概念を説明しました.
最初は手順が多くて大変だと思いますが, 覚えてしまうと案外簡単に使えます.
Androidの場合, テスト, Debug, Production環境の大きく3つの環境がありますが, それぞれの環境を切り替えることも容易になります(詳しくはPart2で話します).

Part2では, より実践的なDagger2の使い方について説明します.


## 参考

-	[Dependency Injection with Dagger 2 (Devoxx 2014)](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014)
-	[Dependency Injection With Dagger 2 on Android](http://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345)
-	[DAGGER 2 - A New Type of dependency injection](https://www.youtube.com/watch?v=oK_XtfXPkqw)
- [Inversion of Control コンテナと Dependency Injection パターン](http://kakutani.com/trans/fowler/injection.html)
