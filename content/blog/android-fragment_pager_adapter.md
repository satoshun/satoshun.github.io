+++
date = "2015-02-22T04:08:23Z"
title = "Android: FragmentPagerAdapterを使った時の話"
tags = ["android", "java"]
blogimport = true
type = "post"
+++


ハマったのでメモ. 以下のことをしたかった.

- データ取得するまでFragmentでは, Progress Barを出力
- ネットワークからデータを取得し, Fragmentのデータ更新
- FramentのListViewにデータ表示

上記を実装した話しになります.


## notifyDataSetChangedメソッドが効かない

データが更新されたらFragmentPagerAdapter#notifyDataSetChangedメソッドを叩けばFramentが再生成されるんでしょ?と思っていた時期が僕にもありました.

notifyDataSetChangedメソッドを叩いても, Fragmentが再生成されず, ProgressBarが表示されたままでした.

FragmentPagerAdapterでは, 基本的に一度作られたFragmentは削除されず,
notifyDataSetChangedメソッドでデータを更新したよーと知らせても, Fragmentを再生成してくれません.(これは仕様通りです)

これはどうしたものかと思っていろいろ調べたところ, FragmentStatePagerAdapterクラスにいきつきました.


## FragmentStatePagerAdapterを使う

FragmentPagerAdapterでなくて, FragmentStatePagerAdapterを使えば動作しました.

以下, 実装例になります.


## 実装例: ダメパターン

まずは, ダメパターンです. FragmentPagerAdapterクラスを使っています.
Activityでデータを受け取ったら, refreshメソッドを叩くようになっています.

```java
public class PagerAdapter extends FragmentPagerAdapter {
    ...
    ...

    public void refresh() {
        notifyDataSetChanged();
    }
}
```

FragmentPagerAdapterがスーパークラスになっているので,
notifyDataSetChangedを何度叩いても, Fragmentが再生成されることはありません.


## 実装例: OKパターン

次にOKパターンです. FragmentStatePagerAdapterを使い, getItemPositionをOverrideするのがポイントです.

```java
public class PagerAdapter extends FragmentStatePagerAdapter {
    private List<Fragment> mFragments;

    ...
    ...

    @Override
    public int getItemPosition(Object object) {
        Fragment target = (Fragment) object;
        if (mFragments.contains(target)) {
            return POSITION_UNCHANGED;
        }

        return POSITION_NONE;
    }

    public void refresh() {
        mFragments.clear();
        notifyDataSetChanged();
    }
}
```

getItemPositionは, Fragmentの状態を管理しているメソッドです.
ここで, POSITION_NONE を返すと, Fragmentを再生成してくれます.

refreshメソッドで, mFragmentsを空にして, notifyDataSetChangedを叩くとFragmentが再生成されます.


## まとめ

FragmentPagerAdapterはFragmentの再生成をしない代わりに, パフォーマンスが高くなっています. 場合によって使い分ける必要がありそうです.
