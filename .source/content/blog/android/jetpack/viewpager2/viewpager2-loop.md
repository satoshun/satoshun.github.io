+++
date = "Sun Oct 13 04:25:38 UTC 2019"
title = "ViewPager2で要素をループさせる"
tags = ["android", "viewpager2", "jetpack"]
blogimport = true
type = "post"
draft = true
+++

ViewPager2で要素をループさせる方法になります。いわゆる循環リスト的なのです。
ViewPager2ではRecyclerViewを使うことが出来るため、RecyclerViewとほぼ同じ実装で実現することが出来ます。

最終的にこんなのが作れます。

<img src="/blog/android/jetpack/viewpager2/viewpager2-loop.gif" style="max-width:280px" />

今回の検証に用いたコードは、[satoshun/ViewPager2](https://github.com/satoshun-android-example/ViewPager2/tree/master/app/src/main/java/com/github/satoshun/example/infinite)にあります😃

では、コードを説明していきます。今回は、1つのViewTypeのみを扱っていきます。

## 実際のコード

まず、`RecyclerView.Adapter`のサイズを決める`getItemCount`メソッドから、限りなく大きい値(`Int.MAX_VALUE`)を返します。

```kotlin
override fun getItemCount(): Int = Int.MAX_VALUE
```

次に、`onBindViewHolder`メソッドを次のように実装します。

```kotlin
private val itemData: List<Data>

override fun onBindViewHolder(holder: InfiniteViewHolder, position: Int) {
  val data = itemData[position % itemData.size]
  ...
}
```

`itemData`にはViewの生成に必要な実際のデータが入っています。`position % itemData.size`とindexを取ることで、ループ中の位置を特定することが出来ます。

最後にViewPager2を次のようにセットアップします。

```kotlin
with(binding.viewpager) {
  val infiniteAdapter = Adapter(itemData = mockAdapterData)
  adapter = infiniteAdapter
  val center = Int.MAX_VALUE / 2
  val start = center - (center % mockAdapterData.size)
  setCurrentItem(start, false)
}

private val mockAdapterData = (0..2).map {
  InfiniteData(title = ('a' + it).toString())
}
```

`setCurrentItem`は0ではなく、中心をセットしてあげます。そうすることで、最初の状態でも左にスクロールして、最後の要素を出すことが出来ます。

これで完成です！キャッシュとか効かせたい場合や、複数のViewTypeを扱いたいときは、もう少し工夫が必要になると思いますが、基本はこれだけで動かすことが出来ます。

## 注意点

- 複数のViewTypeを扱うときは、もう少し工夫が必要
- `Int.MAX_VALUE / 2`回スクロールすると、先頭にたどり着くので無限スクロールではない
    - とはいえ、普通そんなにスクロールしないし、無視しても問題ないと思います
