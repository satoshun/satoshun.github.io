+++
date = "2015-02-22T04:09:23Z"
title = "Android: Pull to Refreshの実装(SwipeRefreshLayout)"
tags = ["android", "java"]
blogimport = true
type = "post"
+++

AndroidでPull to Refreshの実装方法です. ListViewなどを下方向に引っ張ると, データを更新するように出来ます. Gmailとかで使われているあれです.

具体的には, `SwipeRefreshLayout`を使って実装します. 以下でコードで説明していきます.


## XML側の記述

ListViewに覆いかぶさるように定義します.

```xml
<android.support.v4.widget.SwipeRefreshLayout
    android:id="@+id/refresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</android.support.v4.widget.SwipeRefreshLayout>
```

XML側はこれで完了です.
これで, ListViewを引っ張ると「Pull to Refresh」のアニメーションが起こります.


## Activity側の記述

Pull to Refreshをした時に, イベントが発生するのでListenerを記述します.

実装例です. `setOnRefreshListener`でListenerを登録します.

```java

private SwipeRefreshLayout mSwipe;

@Override
protected void onCreate(Bundle savedInstanceState) {
  ...
  mSwipe = (SwipeRefreshLayout) findViewById(R.id.refresh);
  // Callback登録
  mSwipe.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
      @Override
      public void onRefresh() {
          /* ここに適当な処理を書く */
          mSwipe.setRefreshing(false);
      }
  });
}
```

ListViewを引っ張ると, `setOnRefreshListener`メソッドがコールされます.
`setOnRefreshListener`の最後に, `setRefreshing(false)`をコールします. これは, 更新アニメーションをOffにするものです.


## アニメーションの色を変える

上のバーに出てくるアニメーションの色を変えることが出来ます.

`setColorScheme`を使います.

```java
mSwipe.setColorScheme(android.R.color.holo_blue_bright,
    android.R.color.holo_green_light,
    android.R.color.holo_orange_light,
    android.R.color.holo_red_light);
```

これで, 4色でアニメーションが行われます.


## まとめ

簡単にGmailライクなリフレッシュアニメーションが作れました.
