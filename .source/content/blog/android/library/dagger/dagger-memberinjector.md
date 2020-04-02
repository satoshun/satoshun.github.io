+++
date = "2017-12-19"
title = "Dagger: MembersInjectorを使い依存を注入する
tags = ["android", "dagger"]
blogimport = true
type = "post"
+++

## 悩んでいたこと

Dagger Androidを使いカスタムViewクラスに対して依存を注入する方法


## 背景/前提知識

Dagger AndroidはActivity, Fragment, Serviceなどのコンポーネントを起点にして依存関係を解決してくれます。しかし、カスタムViewのコンスタラクタはAndroid Frameworkから呼び出されるため、 コンストラクタインジェクションを使うことは出来ません。ということはセッター/フィールドインジェクションを使いカスタムViewに対して、依存を注入する必要があります。

上のことをDagger Androidで実現するにはどうすれば良いのかを考えてみました。


## 解決策

`MembersInjector<T>`を使い依存を注入する方法が良いのではと思っています。`MembersInjector<T>` はとあるクラスに依存を注入するためのインターフェースになります。Component, Module側に特別な記述すること無く使えるDaggerサイドから提供されているインターフェースです。

```java
public interface MembersInjector<T> {
  void injectMembers(T instance);
}
```

具体的な疑似コードは下のようになります。

```java
// カスタムView
class MainView extends View {
  @Inject UserRepository repository;  // 適当なRepositoryクラスを注入したい
}

// エントリポイントActivity
class MainActivity extends Activity {
  @Inject MembersInjector<MainView> injector;

  @Override
  protected void onCreate(@Nullable Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
    MainView view = findViewById(R.id.main_view);s
    injector.injectMembers(view);
  }
}
```

Daggerで正しく`UserRepository`を注入するための依存関係を定義できていれば、特殊な記述をすることなく、`MainActivity`に`MembersInjector<MainView>`を注入することが出来ます。注入された`MembersInjector<MainView> `を 使うことで、MainViewに依存を注入することが出来ます。


## まとめ

- MembersInjectorは神
