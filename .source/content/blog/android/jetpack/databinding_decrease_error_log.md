+++
date = "2018-12-05T00:00:00Z"
title = "DataBindingのエラーログが3.4.0-alpha07から見やすくなる"
tags = ["android", "jetpack", "databinding"]
blogimport = true
type = "post"
draft = true
+++

Data Binding + Dagger2などのアノテーション系のライブラリを使うとエラーログが膨大になる問題があります。
それが`3.4.0-alpha07`以降で改善される見込みです🎉

詳細はここにあります。https://issuetracker.google.com/issues/116541301

## エラーログの変化

どのようにエラーログが変更されるか、適当にDagger周りのコードでエラーを出して確認してみます。

まずは3.2.1から。

```
> Task :app:kaptGenerateStubsDebugKotlin
e: /Users/stsn/git/github.com/satoshun-android-example/DataBindingApiDeprecateExample/app/build/generated/data_binding_base_class_source_out/debug/dataBindingGenBaseClassesDebug/out/com/github/satoshun/example/sample/databinding/MainAct79Binding.java:17: error: cannot find symbol
  protected MainAct79Binding(DataBindingComponent _bindingComponent, View _root,
                             ^
  symbol:   class DataBindingComponent
  location: class MainAct79Binding
e: /Users/stsn/git/github.com/satoshun-android-example/DataBindingApiDeprecateExample/app/build/generated/data_binding_base_class_source_out/debug/dataBindingGenBaseClassesDebug/out/com/github/satoshun/example/sample/databinding/MainAct79Binding.java:31: error: cannot find symbol
      boolean attachToRoot, @Nullable DataBindingComponent component) {
                                      ^
...
```

DataBinding周りのエラーログが無限に出ます。かなしい😂

次に3.4.0-alpha07です。

```
> Task :app:kaptGenerateStubsDebugKotlin
e: /Users/stsn/git/github.com/satoshun-android-example/DataBindingApiDeprecateExample/app/build/tmp/kapt3/stubs/debug/com/github/satoshun/example/sample/MainActivityBuilder.java:6: error: incompatible types: NonExistentClass cannot be converted to Annotation
@error.NonExistentClass()
```

ちゃんと問題があるコード箇所でエラーログが出ました!!DataBinding周りのエラーは出ていません!!嬉しい😃

## クライアント側の対応

これに伴い、一部APIがdeprecatedになります。例えば、`main_act.xml`は次のようにAPIが変更されます。

```java
public abstract class MainActBinding extends ViewDataBinding {
  @NonNull
  public static MainActBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable ViewGroup root,
      boolean attachToRoot
  )

  @NonNull
  @Deprecated
  public static MainActBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable ViewGroup root, boolean attachToRoot, @Nullable Object component
  )

  @NonNull
  public static MainActBinding inflate(@NonNull LayoutInflater inflater)

  @NonNull
  @Deprecated
  public static MainActBinding inflate(@NonNull LayoutInflater inflater, @Nullable Object component)

  public static MainActBinding bind(@NonNull View view)

  @Deprecated
  public static MainActBinding bind(@NonNull View view, @Nullable Object component)
}
```

使う側の作業としては、上記のdeprecatedになったAPIを置き換える必要があります。ただ、消えたわけではないのですぐに置き換える必要はありません。

## 補足

AGPのアップデートは気軽に出来ないので、
Data Bindingのバージョンだけをアップデートしようと思ったんですが、エラーが出てしまい出来ませんでした。

```groovy
kapt "androidx.databinding:databinding-compiler:3.4.0-alpha07"
```

```txt
ERROR: Data Binding annotation processor version needs to match the Android Gradle Plugin version. You can remove the kapt dependency androidx.databinding:databinding-compiler:3.4.0-alpha07 and Android Gradle Plugin will inject the right version.
```

AGPとDataBindingのバージョンは紐付いているため、片方だけをアップデートしようとしても無理なようです。

## まとめ

- AGPのアップデートをすればData Bindingのツラミの1つであったエラーログから解放される（かも）

Happy Data Binding Life🎉🎉🎉
