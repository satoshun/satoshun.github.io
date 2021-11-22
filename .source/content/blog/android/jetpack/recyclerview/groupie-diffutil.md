+++
date = "Thu Nov 21 12:35:25 UTC 2019"
title = "Android: Groupieの内部でやっている差分更新周りの話"
tags = ["android", "recyclerview", "groupie"]
blogimport = true
type = "post"
draft = false
+++

Groupieには`GroupAdapter#update`メソッドという便利なメソッドがあります。

この記事では`update`メソッドをコールした時に、どのように差分更新されるかを、簡単な説明と実際に動かしてみて見ていきます。

## そもそもRecyclerViewの差分更新って何よ

RecyclerViewでは、DiffUtil.Callbackクラスを実装することで、前のAdapterの状態と、新しいAdapterの状態の差分を計算することが出来ます。その計算結果をもとに、RecyclerViewでは効率的にViewを更新してくれます。またいい感じにアニメーションを実行してくれます。

Groupieでは、内部で`DiffUtil.Callback`を実装した`DiffCallback`クラスがあり、そのクラスをもとに差分更新が行われます。

## DiffCallbackクラスの実装を見ていく

まずは`areItemsTheSame`メソッドから。`areItemsTheSame`メソッドは、Itemが同一かどうかを判定します。

```java
@Override
public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
    Item oldItem = GroupUtils.getItem(oldGroups, oldItemPosition);
    Item newItem = GroupUtils.getItem(newGroups, newItemPosition);
    return newItem.isSameAs(oldItem);
}
```

まず、1つ前のItemと新しいItemを取ってきて、`newItem.isSameAs(oldItem);`をコールしています。

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

Idは、Itemクラスのコンストラクタから与えることが出来ます。

```java
protected Item(long id) {
    this.id = id;
}
```

なので、GroupAdapterがupdateされる可能性があるなら、適切なIdを渡すのが良いです。
例えば、Userクラス的なものがあって、運良くUserを一意に判定できるidが定義されていたら、それを渡すと良いと思います。

---

次に、`areContentsTheSame`メソッド。`areContentsTheSame`メソッドは、このItemの内容が同じかどうかを判定します。

```java
@Override
public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
    Item oldItem = GroupUtils.getItem(oldGroups, oldItemPosition);
    Item newItem = GroupUtils.getItem(newGroups, newItemPosition);
    return newItem.hasSameContentAs(oldItem);
}
```

1つ前のItemと新しいItemを取ってきて、`newItem.hasSameContentAs(oldItem);`をコールしています。

```java
public boolean hasSameContentAs(Item other) {
    return this.equals(other);
}
```

デフォルトでは、`Object#equals`メソッドをコールしているので、同一のインスタンスかどうかで、Itemの内容が同じかどうかを判定しています。
Itemのインスタンスが変わったらfalseを返し、Itemの内容が変わっても、同一インスタンスならtrueを返します。

なので、基本的には`hasSameContentAs`ないし、`equals`メソッドをoverrideしたほうが良いです。

Kotlinなら、data classで定義するのも手です。equalsを自動で実装してくれるからです。

```kotlin
data class HogeItem(model, id) : Item<...>(id) ...
```

---

最後に、`getChangePayload`メソッド。`getChangePayload`メソッドは、Itemの差分を計算して、payloadを求めます。

```java
public Object getChangePayload(int oldItemPosition, int newItemPosition) {
    Item oldItem = GroupUtils.getItem(oldGroups, oldItemPosition);
    Item newItem = GroupUtils.getItem(newGroups, newItemPosition);
    return oldItem.getChangePayload(newItem);
}
```

1つ前のItemと新しいItemを取ってきて、`newItem.getChangePayload(oldItem);`をコールしています。

```java
public Object getChangePayload(Item newItem) {
    return null;
}
```

デフォルトでは、nullを返しており、payloadの計算は行われません。ここでいい感じにpayloadを計算してあげると、効率よく差分更新が出来ます。

## ケースごとに動作を見てみる

今までのを踏まえて、いろいろなケースでどのように動作するかを見ていきます。

### 違うid && インスタンスを毎回生成する

```kotlin
class Adapter : GroupAdapter<GroupieViewHolder>() {
  fun update() {
    update(listOf(BasicItem())) // 毎回新しく生成する && idは毎回違う
  }
}

class BasicItem : Item<GroupieViewHolder>()
```

{{< figure src="/blog/android/jetpack/recyclerview/device-2019-11-21-111037.gif" width="30%" height="250" style="object-fit: cover;object-position: 100% 0;" >}}

Itemが点滅しているのが分かります。これは、Idが毎回異なるので、`areItemsTheSame`がfalseを返すためです。この場合、Viewは再利用されません。

### 同じid && インスタンスを毎回生成する && equalsなどを実装しない

```kotlin
class Adapter : GroupAdapter<GroupieViewHolder>() {
  fun update() {
    update(listOf(BasicItem(3))) // 毎回新しく生成する && idは固定
  }
}

class BasicItem(id: Long) : Item<GroupieViewHolder>(id)
```

{{< figure src="/blog/android/jetpack/recyclerview/device-2019-11-21-112018.gif" width="30%" height="250" style="object-fit: cover;object-position: 100% 0;" >}}

この場合も、Itemが点滅しているのが分かります。これは、equalsメソッドなどを実装していないので、`areContentsTheSame`がfalseを返すためです。
`areContentsTheSame`がfalse かつ payloads周りの実装がない かつ RecyclerView.ItemAnimator周りの設定を変えていない場合は、1つ前のViewが再利用されません。


### 同じid && インスタンスを毎回生成する && equalsを実装して、trueを返す

```
class Adapter : GroupAdapter<GroupieViewHolder>() {
  fun update() {
    update(listOf(BasicItem(3))) // 毎回新しく生成する && idは固定
  }
}

// dataクラスなのでequalsが実装される
data class BasicItem(private val id: Long) : Item<GroupieViewHolder>(id)
```

{{< figure src="/blog/android/jetpack/recyclerview/device-2019-11-21-113425.gif" width="30%" height="250" style="object-fit: cover;object-position: 100% 0;" >}}

この場合は、Itemが点滅していないのが分かります。これは、`areItemsTheSame`と`areContentsTheSame`がtrueを返すので、Viewが再利用されるためです。またItemにはbindメソッドがあるんですが、bindメソッドもコールされません。なぜなら、`areContentsTheSame`がtrueなので、Viewの中身を更新する必要がないためです。

多くの処理をスキップ、再利用することが出来ます。

### 同じidを渡す && インスタンスを毎回生成する && getChangePayloadを実装する

```kotlin
class Adapter : GroupAdapter<GroupieViewHolder>() {
  fun update() {
    update(listOf(BasicItem(3))) // 毎回新しく生成する && idは固定
  }
}

class BasicItem(id: Long) : Item<GroupieViewHolder>(id) {
  ...
  override fun getChangePayload(newItem: Item<*>): Any? {
    return newItem.id // 今回はサンプルなので適当な実装
  }
}
```

{{< figure src="/blog/android/jetpack/recyclerview/device-2019-11-21-114407.gif" width="30%" height="250" style="object-fit: cover;object-position: 100% 0;" >}}

この場合も、Itemが点滅していないのが分かります。これは、`areContentsTheSame`はfalseを返しているのですが、payload（以前との差分）を計算しているため、1つ前のViewが再利用されるためです。この場合、bindメソッドはコールされます。なぜなら、`areContentsTheSame`がfalseなので、Viewの中身を更新する必要があるためです。

## まとめ

- Groupie（RecyclerView）で効率的に差分更新をしたいなら、以下の3点を考慮する必要があります
    - `areItemsTheSame`メソッド: Itemのidに一意な値を渡す
    - `areContentsTheSame`メソッド: data classにする または equalsメソッドの実装 または `hasSameContentAs`メソッドを実装する
    - `getChangePayload`メソッド: 古いItemと新しいItemのどこが差分なのかを計算して、適切なpayloadを返す
- 本文中では触れてないんですが、RecyclerViewがデフォルトで使う、DefaultItemAnimatorの設定を変えることで、ここらへんの挙動を変えることが出来ます
    - 例えば、supportsChangeAnimationsをfalseすると、`getChangePayload`を実装していなくても、Viewが再利用されるようになります
