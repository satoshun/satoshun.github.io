+++
date = "Wed Nov 20 11:35:17 UTC 2019"
title = "Android: Groupieの内部でやっている差分更新周りの話"
tags = ["android", "recyclerview", "groupie"]
blogimport = true
type = "post"
draft = true
+++

Groupieには`GroupAdapter#update`メソッドという便利なメソッドがあります。

この記事では`update`メソッドをコールした時に、どのように差分更新されるかを説明します。

## そもそもRecyclerViewの差分更新って何よ

RecyclerViewでは、DiffUtil.Callbackクラスを実装することで、前の状態と、新しい状態の差分を計算することが出来ます。
その計算結果をもとに、RecyclerViewでは効率的にViewを更新してくれます。また設定したアニメーションを実行してくれます。

Groupieでは、内部で`DiffUtil.Callback`を実装した`DiffCallback`クラスがあり、そのクラスをもとに差分更新が行われます。

## 実装を見てみる

まずは`areItemsTheSame`メソッドから。`areItemsTheSame`メソッドは、このItemが同一のものか判定します。

```java
@Override
public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
    Item oldItem = GroupUtils.getItem(oldGroups, oldItemPosition);
    Item newItem = GroupUtils.getItem(newGroups, newItemPosition);
    return newItem.isSameAs(oldItem);
}
```

まず、前のItemと新しいItemを取ってきて、`newItem.isSameAs(oldItem);`をコールしています。

`isSameAs`メソッドは次の定義になっています。

```java
public boolean isSameAs(Item other) {
    if (getViewType() != other.getViewType()) {
        return false;
    }
    return getId() == other.getId();
}
```

ItemのviewTypeが等しい かつ Idが等しい場合にtrueを返します。

次に、`areContentsTheSame`メソッド。`areContentsTheSame`メソッドは、このItemの内容が同じかどうかを判定します。

```java
@Override
public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
    Item oldItem = GroupUtils.getItem(oldGroups, oldItemPosition);
    Item newItem = GroupUtils.getItem(newGroups, newItemPosition);
    return newItem.hasSameContentAs(oldItem);
}
```

前のItemと新しいItemを取ってきて、`newItem.hasSameContentAs(oldItem);`をコールしています。

```java
public boolean hasSameContentAs(Item other) {
    return this.equals(other);
}
```

デフォルトでは、`Object.equals`メソッドをコールしているので、同一のインスタンスかどうかを判定しています。

最後に、`getChangePayload`メソッド。`getChangePayload`メソッドは、Itemの差分を計算して、payloadを求めます。

```java
public Object getChangePayload(int oldItemPosition, int newItemPosition) {
    Item oldItem = GroupUtils.getItem(oldGroups, oldItemPosition);
    Item newItem = GroupUtils.getItem(newGroups, newItemPosition);
    return oldItem.getChangePayload(newItem);
}
```

前のItemと新しいItemを取ってきて、`newItem.getChangePayload(oldItem);`をコールしています。

```java
public Object getChangePayload(Item newItem) {
    return null;
}
```

デフォルトでは、nullを返しており、payloadの計算は行われません。

