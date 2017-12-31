+++
date = "2015-03-18T00:00:00Z"
title = "Android: FragmentPagerAdapterでハマった話"
tags = ["android", "java"]
blogimport = true
type = "post"
+++


ハマったのでメモ. 以下のことをしたかった.

- データ取得するまで, Fragment内でProgress Barを出力
- ネットワークからデータを取得し, Fragment内にあるAdapterのデータ更新
- UIに反映


## notifyDataSetChangedメソッドが効かない

データが更新した後に FragmentPagerAdapter#notifyDataSetChangedメソッドを叩けばFramentが再生成されるんでしょ?と思っていた時期が僕にもありました.
notifyDataSetChangedメソッドを叩いても, データがUIに反映されず, ProgressBarが表示されたままでした.

FragmentPagerAdapterでは, 基本的に一度作られたFragmentは削除されず, notifyDataSetChangedメソッドでデータを更新したよーと知らせても, Fragmentを再生成してくれません.(仕様通り)

これはどうしたものかと思っていろいろ調べたところ, FragmentStatePagerAdapterクラスにいきつきました.


## FragmentStatePagerAdapterを使う

FragmentPagerAdapterでなくて, FragmentStatePagerAdapterを使えばUIにデータが反映されました.

以下, 実装例になります. まずは, ダメパターンです. FragmentPagerAdapterクラスを使い, Activityでデータを受け取ったら, refreshメソッドを叩くようになっています.

```java
public class PagerAdapter extends FragmentPagerAdapter {
    ...
    ...

    public void refresh() {
        notifyDataSetChanged();
    }
}
```

FragmentPagerAdapterがスーパークラスになっているため, notifyDataSetChangedを何度叩いても, Fragmentが再生成されることはありません. つまり, データを更新した後にUIにデータが反映されません.

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

getItemPositionは, Fragmentの状態を管理しているメソッドです. ここで, `POSITION_NONE` を返すとFragmentを再生成してくれます. つまり, データを更新した後に `POSITION_NONE` を返すように実装すれば良いわけです.


## まとめ

FragmentPagerAdapterはFragmentの再生成をしない代わりに, パフォーマンスが高くなっています. 場合によって使い分ける必要がありそうです.
