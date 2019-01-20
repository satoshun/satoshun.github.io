+++
date = "Sun Jan 20 06:56:16 UTC 2019"
title = "R8/Proguard: Daggerの生成コードがR8でどのように変わるかを見る"
tags = ["android", "r8", "proguard", "dagger"]
blogimport = true
type = "post"
draft = true
+++

コードの最適化の話です。この記事では実践に寄せて、Daggerの生成コードがR8によってどのように変化するかを見ます。

まずはサンプルコードです。

```kotlin
@Component(
  modules = [
    AppModule1::class,
    AppModule2::class
  ]
)
interface AppComponent {
  @Component.Builder
  interface Builder {
    fun build(): AppComponent
  }

  fun inject(activity: MainActivity)
}

@Module
class AppModule1 {
  @Provides
  fun provideService(): AppService {
    val retrofit = Retrofit.Builder()
      ...
    return retrofit.create()
  }
}

@Module
object AppModule2 {
  @JvmStatic
  @Provides
  fun provideService2(): AppService2 {
    val retrofit = Retrofit.Builder()
      ...
    return retrofit.create()
  }
}

---

class MainActivity : AppCompatActivity() {
  @Inject lateinit var appService: AppService
  @Inject lateinit var appService2: AppService2

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val appComponent = DaggerAppComponent.builder().build()
    appComponent.inject(this)

    ...
  }
}
```

シンプルなAppComponentを定義して、そこにシンプルなAppModule1とAppModule2を紐づけています。それを、MainActivityで使うコードになっています。

これを最適化なしでdex変換して、デコンパイルしてみます。

```java
public final class MainActivity extends AppCompatActivity {
    @Inject
    @NotNull
    public AppService appService;
    @Inject
    @NotNull
    public AppService2 appService2;

    ...

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        DaggerAppComponent.builder().build().inject(this);
        ...
    }
}
```

普通のコードです。KotlinをJava変換しただけなので、特におかしな部分もありません。

次にR8による最適化を実行します。

```java
public final class MainActivity extends m {
    public AppService o;
    public AppService2 p;
    ...

    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView((int) R.layout.activity_main);
        Object a = new AppModule1().a();
        a.a(a, "Cannot return null from a non-@Nullable @Provides method");
        this.o = a;
        this.p = AppModule2_ProvideService2Factory.a();
        ...
    }
}
```

何ということでしょう。`DaggerAppComponent`が消えてしまいました！

DaggerAppComponentの各メソッドがMainActivity側にインライン展開されることで、完全にDaggerAppComponentを消すことが出来ます。
実際にapkの中身を見て、DaggerAppComponentが存在しないことを確認しました。R8すごい😃

## 補足

Proguardだと、デフォルトの使い方だと上記のサンプルから、`DaggerAppComponent`を消すことが出来ませんでした。

## まとめ

- R8すごい😃😃😃
