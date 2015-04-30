+++
date = "2015-04-27T00:00:00Z"
title = "Android: Dagger2でDI"
tags = ["android"]
blogimport = true
type = "post"
draft = true
+++

Dagger2はDI(Dependency Injection)をするライブラリになります.

## DIとは?

最初にDIについて説明します. DIとは依存を注入することです. ここでいう依存とは「クラスオブジェクト同士の依存関係」のことを表しています.

コードで説明します. 以下のコードがあったとします.

```java
class Blogger {
    private String username;

    public Blogger(String username) {
        this.username = username;
    }

    public void post(String title) {
      PostBlog postBlog = new PostBlog();
      postBlog.post(username, title);
    }
}

class PostBlog {
    public PostBlog() {
    }

    public void post(String username, String title) {
        OkHttpClient client = new OkHttpClient();
        Request request = ...;
        client.newCall(request).execute();
    }
}
```

`BloggerはPostBlogに依存している`. ことが分かります. 上記コードでは, メソッド内でインスタンスを生成してしますが, 引数で渡せるように変更してみます.

```java
class Blogger {
    private String username;
    private PostBlog postBlog;

    public Blogger(String username, PostBlog postBlog) {
        this.username = username;
        this.postBlog = postBlog;
    }

    public void post(String title) {
      postBlog.post(username, title);
    }
}

class PostBlog {
    private OkHttpClient client;

    public PostBlog(OkHttpClient client) {
        this.client = client;
    }

    public void post(String username, String title) {
        Request request = ...;
        client.newCall(requests).execute();
    }
}
```

こうすることで外部から引数で, クラス同士の依存関係を指定できるようになります. これがDI(依存性の注入)パターンです.




## 参考

- [Dependency Injection with Dagger 2 (Devoxx 2014)](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014)
- [Dependency Injection With Dagger 2 on Android](http://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345)
- [DAGGER 2 - A New Type of dependency injection](https://www.youtube.com/watch?v=oK_XtfXPkqw)
