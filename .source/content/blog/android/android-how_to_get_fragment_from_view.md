+++
date = "2018-01-28T00:00:00Z"
title = "Android: ViewがどのFragmentに属しているかをViewから取得する"
tags = ["android"]
blogimport = true
type = "post"
+++

ViewがどのFragmentに属しているかを取得する方法の紹介になります。
前々からtag、id以外の仕組みで取得出来ないかなと考えていたら、GlideでViewからFragmentを取得するコードがありました。

```java
@Nullable
private Fragment findSupportFragment(@NonNull View target, @NonNull FragmentActivity activity) {
  tempViewToSupportFragment.clear();
  findAllSupportFragmentsWithViews(
      activity.getSupportFragmentManager().getFragments(), tempViewToSupportFragment);
  Fragment result = null;
  View activityRoot = activity.findViewById(android.R.id.content);
  View current = target;
  while (!current.equals(activityRoot)) {
    result = tempViewToSupportFragment.get(current);
    if (result != null) {
      break;
    }
    if (current.getParent() instanceof View) {
      current = (View) current.getParent();
    } else {
      break;
    }
  }

  tempViewToSupportFragment.clear();
  return result;
}

private static void findAllSupportFragmentsWithViews(
     @Nullable Collection<Fragment> topLevelFragments,
     @NonNull Map<View, Fragment> result) {
   if (topLevelFragments == null) {
     return;
   }
   for (Fragment fragment : topLevelFragments) {
    // getFragment()s in the support FragmentManager may contain null values, see #1991.
    if (fragment == null || fragment.getView() == null) {
      continue;
    }
    result.put(fragment.getView(), fragment);
    findAllSupportFragmentsWithViews(fragment.getChildFragmentManager().getFragments(), result);
  }
}
```

https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/manager/RequestManagerRetriever.java#L192

説明すると、ターゲットのViewが所属しているactivity(`View#getContext`から取得できる)ので、

1. `activity.getSupportFragmentManager().getFragments()`と`fragment.getChildFragmentManager().getFragments()`から、Activityが保持している全Fragmentを取得する
2. それらのFragmentのTopのViewを取得する
3. ターゲットのViewのgetParentを辿っていき、2で取得したViewと一致するViewを探す
4. 一致したViewが所属するFragment == ターゲットのViewが所属するFragment

という流れになっています。

Glideのコードは効率的なコードなのでやや複雑ですが、Kotlinで効率性を考えずにに書くなら以下のようにも書けます。

```kotlin
fun View.findAttachFragment(): Fragment? {
  val activity = context as? FragmentActivity ?: return null
  val allFragments = findAllFragments(activity.supportFragmentManager.fragments)

  val root = activity.findViewById<View>(android.R.id.content)
  var result: Fragment? = null
  var current = this
  while (current != root) {
    result = allFragments.firstOrNull { it.view == current }
    if (result != null) break
    current = current.parent as? View ?: break
  }
  return result
}

private fun findAllFragments(
    fragments: List<Fragment?>?
): List<Fragment> {
  if (fragments == null || fragments.isEmpty()) return emptyList()
  val fragments = fragments.filter { it?.view != null }.filterNotNull()
  return fragments + findAllFragments(fragments.map { it.childFragmentManager.fragments }.flatten())
}
```

https://github.com/satoshun-example/GetViewFragmentSample


## まとめ

- ViewからFragmentを取得できる方法は正攻法では無いと思っていたんですが、それなりに簡潔な方法で取得できる方法がありました
