+++
date = "Fri May 15 13:25:48 UTC 2020"
title = "Android: ConstraintLayoutの子にRecyclerViewを配置して、Match Constraintsを設定すると良くない挙動をする"
tags = ["android", "recyclerview", "jetpack"]
blogimport = true
type = "post"
draft = true
+++

備忘録です。

次のようなレイアウトは良くないぞという話です。

```xml
<androidx.constraintlayout.widget.ConstraintLayout
  android:layout_width="match_parent"
  android:layout_height="wrap_content">

  <androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recycler"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

(今回の例ではLinearLayoutManagerをLayoutManagerとして使っています。他のLayoutManagerだとどういう挙動をするか分かりません)

## どんな挙動をするか?

このレイアウトはファーストビューのタイミングで、**すべてのアイテム**をバインドしようとします。

例えば500個のRecyclerViewアイテムがあったときに、画面に収まるかどうかに関わらず500個のバインドが走ります。

```kotlin
with(recycler) {
  // 横方向のLinearLayoutManager
  layoutManager = LinearLayoutManager(
    this@ConstraintMatchConstraintsActivity,
    RecyclerView.HORIZONTAL,
    false
  )

  // 500個のアイテムを生成
  adapter = SampleAdapter().apply {
    submitList((0..500).map { "$index $it" })
  }
}

private class SampleAdapter : ListAdapter<String, RecyclerView.ViewHolder>(...) {
  ...
  override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
    println(getItem(position)) // ここが、0 ~ 500まで表示される
  }
}
```

## なんでか?

LinearLayoutManagerでは、内部で`mInfinite` っていうフィールドを持っていて、これは次の関数によって生成されます。

```java
boolean resolveIsInfinite() {
  return mOrientationHelper.getMode() == View.MeasureSpec.UNSPECIFIED
      && mOrientationHelper.getEnd() == 0;
  }
```

細かいところまで追えてないのですが、今回のレイアウトの場合、上記の関数がtrueを返す挙動をしていました。

この値がtrueだと、どういうことが起こるかっていうと、RecyclerViewにアイテムを詰めるタイミングで一気に全部アイテムを詰めようと試みます。

```java
int fill(RecyclerView.Recycler recycler, LayoutState layoutState,
    RecyclerView.State state, boolean stopOnFocusable) {
  ...
  // mInfintieがtrueなので、このwhileが最後まで行くことが可能になる
  while ((layoutState.mInfinite || remainingSpace > 0) && layoutState.hasMore(state)) {
    ...
  }
}
```

mInfiniteがtrueだと、whileが`remainingSpace`に値に関わらず、最後まで行くことが可能になります。
結果、今回の場合では全アイテムをファーストビューのタイミングでバインドします。

## どう直すか?

直し方としては

1. ConstraintLayoutをRecyclerViewの親にしない
2. wrap_contentを使う

```xml
<androidx.constraintlayout.widget.ConstraintLayout
  android:layout_width="match_parent"
  android:layout_height="wrap_content">

  <androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recycler3"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

3. match_parentを使う

```xml
<androidx.constraintlayout.widget.ConstraintLayout
  android:layout_width="match_parent"
  android:layout_height="wrap_content">

  <androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recycler1"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```


で良いかなと思います😃
