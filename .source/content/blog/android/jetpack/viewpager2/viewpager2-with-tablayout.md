+++
date = "Fri May 17 04:08:42 UTC 2019"
title = "ViewPager2 with TabLayout"
tags = ["android", "viewpager2", "jetpack"]
blogimport = true
type = "post"
draft = false
+++

Material ComponentsでViewPager2 + TabLayoutのコードが入ったのでそれの紹介。まだalphaへのリリースもされていないので、APIは大きく変わるかもしれません。おそらく`1.1.0-alpha07`に入ってくると思います。

TabLayoutはViewPagerでサポートされていましたが、それがViewPager2にも来たって感じです。

## 使い方

新しく追加された[TabLayoutMediator](https://github.com/material-components/material-components-android/blob/67e4489293290853de83ef1b00205058ae25fa8e/lib/java/com/google/android/material/tabs/TabLayoutMediator.java)を使います。

まず、`TabLayoutMediator`インスタンスを生成します。

```kotlin
val viewPager: ViewPager2 = findViewById(R.id.viewpager)
val tabLayout: TabLayout = findViewById(R.id.tab)
val mediator = TabLayoutMediator(tabLayout, viewPager) { tab: TabLayout.Tab, position: Int ->
  tab.text = "test $position" // タブにタイトルをセット
}
```

コンストラクタには、TabLayout、ViewPager2、OnConfigureTabCallbackを渡します。
`OnConfigureTabCallback`は、tabとpositionを受け取り、tabに対して、タイトルをセットします。
ViewPagerのPageAdapterとは違い、RecyclerViewのAdapterからはタイトルを取得できないので、このような変更になったと思われます。

最後に`attach`関数を呼び出します。

```kotlin
mediator.attach()
```

これで、ViewPager2 + TabLayoutを実現できます。とても簡単！！

## まとめ

- ViewPager2もエコシステムが整いつつある😊
- 今回試したサンプルコードは [satoshun-android-example/ViewPager2](https://github.com/satoshun-android-example/ViewPager2)にあります😃
