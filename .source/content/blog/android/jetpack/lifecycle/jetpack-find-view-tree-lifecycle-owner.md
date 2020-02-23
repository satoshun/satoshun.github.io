+++
date = "Sun Feb 23 01:50:36 UTC 2020"
title = "Android: findViewTreeLifecycleOwnerでViewからLifecycleOwnerを取得する"
tags = ["android", "jetpack", "lifecycle"]
blogimport = true
type = "post"
draft = false
+++

lifecycle 2.3.0に、ViewからLifecycleOwnerを取得するメソッドが追加されそうなので、それのメモ。

まだ2.3.0-alphaはリリースされていないので、androidxのSNAPSHOTで試します。
SNAPSHOTの使い方は、ここが参考になります。

[Using AndroidX Snapshot Builds](http://rahulrav.com/blog/using_snapshot_builds.html)

## 使い方

関数の定義と使い方になります。

```kotlin
// 関数の定義
fun View.findViewTreeLifecycleOwner(): LifecycleOwner?

// 使い方
val owner = view.findViewTreeLifecycleOwner()

// 使い方 + Coroutine
view.findViewTreeLifecycleOwner()?.lifecycleScope?.launch {
  ...
}
```

Viewに拡張関数が定義されており、そこから使うことが出来ます。

## 内部実装

シンプルな作りになっており、ViewのタグにActivityやFragmentのLifecycleOwnerを登録しています。

```java
public class ViewTreeLifecycleOwner {
  public static void set(@NonNull View view, @Nullable LifecycleOwner lifecycleOwner) {
    view.setTag(R.id.view_tree_lifecycle_owner, lifecycleOwner);
  }

  @Nullable
  public static LifecycleOwner get(@NonNull View view) {
    LifecycleOwner found = (LifecycleOwner) view.getTag(R.id.view_tree_lifecycle_owner);
    ...
  }
}
```

じゃあsetはどこで行われるかというと、ライブラリ側で自動的に行われます。なので、明示的にsetを呼び出す必要は特にありません。
具体的には、FragmentとComponentActivityを継承していればViewを生成したタイミングで、勝手にセットしてくれます。

Fragmentで生成したViewから`findViewTreeLifecycleOwner`をした場合には、
Fragmentのライフサイクルが取得出来るので、適切なライフサイクルを取得することが可能です。

ただし、試したSNAPSHOTだとAppCompatActivityの継承の場合、自動でLifecycleOwnerが登録されていませんでした。
なので、現状だと、AppCompatActivityの継承の場合は手動でLifecycleOwnerを登録する必要があります。

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  ...
  ViewTreeLifecycleOwner.set(window.decorView, this)
  ...
}
```
