+++
date = "2018-04-12T07:10:00Z"
title = "Android: Data Bindingを使っていると本当のエラーログが出ない話 + 対処法"
tags = ["android", "databinding"]
blogimport = true
type = "post"
+++

## 問題

Android開発でData Bindingを使っていて、さらにDaggerなどのkaptを必要とするライブラリを使っていると、エラーログが非常に見にくい or エラーログに本当の問題が出てこないことがあります。

理由としては、Data Bindingの生成が失敗すると、`MainActivityBinding`みたいなクラスが生成されないので、いたるところでBinding クラスの参照エラーが出ます。
デフォルトではエラーを100行?しか出さないようになっているため、参照エラーだけでデフォルトのエラー行数を超えてしまい、本当のエラーが出力されないケースがあります。(大規模なプロジェクトだと起こりがちだと思います)

## 解決法

全てのエラーログが欲しい時は、build.gradleに以下の記述をすれば良いです

```groovy
kapt {
    javacOptions {
        option("-Xmaxerrs", 5000)
    }
}
```

これは、エラーの行数を増やすための設定です。5000はとりあえずでかい値を入れておけば大丈夫だろうという考えです。

これを入れたことで、弊プロダクトではSupport libraryを27.1.1に上げることに苦労していたのですが、解決することが出来ました。

before(一部ログ修正しています)

```
...
// 長いエラーログ
...
...
  symbol:   class DataBindingComponent
  location: class ActivityMainBinding
e: ActivityMainBinding.java:91: error: cannot find symbol
      @Nullable ViewGroup root, boolean attachToRoot, @Nullable DataBindingComponent component) {
                                                                ^
  symbol:   class DataBindingComponent
  location: class ActivityMainBinding
e: ActivityMainBinding.java:102: error: cannot find symbol
      @Nullable DataBindingComponent component) {
                ^
  symbol:   class DataBindingComponent
  location: class ActivityMainBinding
:app:kaptProductDebugKotlin FAILED
```

after

```
...
// 長いエラーログ
...
...
e: :96: error: incompatible types: possible lossy conversion from long to int
    @android.support.annotation.IntDef(value = {-1L, 500L, 20002L, 20004L})
                                                ^
e::96: error: incompatible types: possible lossy conversion from long to int
    @android.support.annotation.IntDef(value = {-1L, 500L, 20002L, 20004L})
                                                     ^
e: :96: error: incompatible types: possible lossy conversion from long to int
    @android.support.annotation.IntDef(value = {-1L, 500L, 20002L, 20004L})
                                                           ^
e: .java:96: error: incompatible types: possible lossy conversion from long to int
    @android.support.annotation.IntDef(value = {-1L, 500L, 20002L, 20004L})
                                                                   ^
:app:kaptProductDebugKotlin FAILED
```

beforeではDataBindingの参照エラーしか出ていないんですが、afterではIntDefでのエラーログが出ていることが分かります。
これが本当に欲しかったエラーログで、これを修正することで無事解決することが出来ました。

## まとめ

- Data Binding便利だけどやっぱつれぇわ
