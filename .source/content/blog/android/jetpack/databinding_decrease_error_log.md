+++
date = "2018-12-05T00:00:00Z"
title = "DataBindingのエラーログが3.4.0-alpha07から見やすくなる"
tags = ["android", "jetpack", "databinding"]
blogimport = true
type = "post"
draft = true
+++

Data Binding + (Dagger2)を使うとエラーログが膨大になる問題があります。
それが`3.4.0-alpha07`で改善される見込みです🎉

https://issuetracker.google.com/issues/116541301

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

使う側の作業としては、上記のdeprecatedになったAPIを置き換える必要があります。ただ、互換性はあるのですぐに置き換える必要はありません。

## 補足

Data Bindingのバージョンだけを強制的にアップデートしようと思ったんですが、エラーが出てしまい出来ませんでした。

```groovy
kapt "androidx.databinding:databinding-compiler:3.4.0-alpha07"
```

```txt
ERROR: Data Binding annotation processor version needs to match the Android Gradle Plugin version. You can remove the kapt dependency androidx.databinding:databinding-compiler:3.4.0-alpha07 and Android Gradle Plugin will inject the right version.
```

AGPとDataBindingのバージョンは紐付いているため、片方だけをアップデートしようとしても無理なようです。
