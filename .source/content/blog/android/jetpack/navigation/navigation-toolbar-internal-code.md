+++
date = "Sat Nov 30 06:14:19 UTC 2019"
title = "メモ Android: Navigation Component + Toolbar(ActionBar)周りのコードを読んで見る"
tags = ["android", "jetpack", "navigation"]
blogimport = true
type = "post"
draft = false
+++

Navigation Component + Toolbarのデフォルトの挙動をカスタマイズしたかったので、その周辺のコードを読んでみたメモブログになります。

この記事のコードは、次のライセンスに従います。

```text
/*
 * Copyright 2018 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```

## 2行で

- OnDestinationChangedListenerから、Toolbar(ActionBar)をいじっている
- ということは、OnDestinationChangedListenerをカスタマイズすれば、挙動をいじれる

## 前提/基本

NavigationにActionbarを紐付けるのは、次のようにします。

```kotlin
val navController = findNavController(R.id.nav_host_fragment)
val configuration = AppBarConfiguration(navController.graph)
setupActionBarWithNavController(navController, configuration)
```

これが、どんな感じで、処理されていくかを見ていきます。


## AppBarConfigurationクラス

naivgation graphなどの情報から、AppBarがどのように振る舞うかを決めるクラスです。
ここでは、navigate upが失敗した時のハンドリング、DrawerLayoutを設定できます。

```kotlin
AppBarConfiguration.Builder
    setDrawerLayout
    setFallbackOnNavigateUpListener
    build
```

登録した`FallbackOnNavigateUpListener`が、どこで発火するかを、コードを読んで確認します。

FallbackOnNavigateUpListenerのドキュメントを読むと、navigateUpに失敗した時と書いてあるので、navigateUpメソッドを見に行きます。

```java
public static boolean navigateUp(@NonNull NavController navController,
        @NonNull AppBarConfiguration configuration) {
    DrawerLayout drawerLayout = configuration.getDrawerLayout();
    NavDestination currentDestination = navController.getCurrentDestination();
    Set<Integer> topLevelDestinations = configuration.getTopLevelDestinations();
    if (drawerLayout != null && currentDestination != null
        && matchDestinations(currentDestination, topLevelDestinations)) {
        drawerLayout.openDrawer(GravityCompat.START);
        return true;
    } else {
        if (navController.navigateUp()) {
            return true;
        // ここで発火する
        } else if (configuration.getFallbackOnNavigateUpListener() != null) {
            return configuration.getFallbackOnNavigateUpListener().onNavigateUp();
        } else {
            return false;
        }
    }
}
```

navController.navigateUpがfalseを返すときに、`configuration.getFallbackOnNavigateUpListener().onNavigateUp()`が実行され、登録しておいたコールバックが発火します。

## setupActionBarWithNavControllerメソッド

次に、setupActionBarWithNavControllerメソッドを見ていきます。

このメソッドの中身は、次のようになっています。

```java
fun AppCompatActivity.setupActionBarWithNavController(
    navController: NavController,
    configuration: AppBarConfiguration = AppBarConfiguration(navController.graph)
) {
    NavigationUI.setupActionBarWithNavController(this, navController, configuration)
}

public static void setupActionBarWithNavController(@NonNull AppCompatActivity activity,
    @NonNull NavController navController,
    @NonNull AppBarConfiguration configuration) {
    navController.addOnDestinationChangedListener(
        new ActionBarOnDestinationChangedListener(activity, configuration));
}
```

`addOnDestinationChangedListener`から、`ActionBarOnDestinationChangedListener`を設定しています。
`addOnDestinationChangedListener`は、destinationが変わった時に発火するコールバックなので、ページが切り替わったタイミングなどで`ActionBarOnDestinationChangedListener`が発火することが分かります。

---

次に、`ActionBarOnDestinationChangedListener`の中身は、以下のようになっています。

```java
class ActionBarOnDestinationChangedListener extends
        AbstractAppBarOnDestinationChangedListener {
    private final AppCompatActivity mActivity;

    ActionBarOnDestinationChangedListener(@NonNull AppCompatActivity activity,
            @NonNull AppBarConfiguration configuration) {
        super(activity.getDrawerToggleDelegate().getActionBarThemedContext(), configuration);
        mActivity = activity;
    }

    @Override
    protected void setTitle(CharSequence title) {
        ActionBar actionBar = mActivity.getSupportActionBar();
        actionBar.setTitle(title);
    }

    @Override
    protected void setNavigationIcon(Drawable icon,
            @StringRes int contentDescription) {
        ActionBar actionBar = mActivity.getSupportActionBar();
        if (icon == null) {
            actionBar.setDisplayHomeAsUpEnabled(false);
        } else {
            actionBar.setDisplayHomeAsUpEnabled(true);
            ActionBarDrawerToggle.Delegate delegate = mActivity.getDrawerToggleDelegate();
            delegate.setActionBarUpIndicator(icon, contentDescription);
        }
    }
}
```

AbstractAppBarOnDestinationChangedListenerクラスを継承しており、setTitle、setNavigationメソッドがそれぞれ実装されています。

まず、setTitleメソッドで、actionBarにタイトルをセットしています。

次に、setNavigationIconメソッドでは、引数のiconがnullなら、`setDisplayHomeAsUpEnabled`がfalseになり、
iconがあるなら`setDisplayHomeAsUpEnabled`がtrueになり、upボタンが表示されます。

---

次に、基底クラスの`AbstractAppBarOnDestinationChangedListener`を見ていきます。

`AbstractAppBarOnDestinationChangedListener`は長いので、重要な部分だけを抜粋します。

```java
abstract class AbstractAppBarOnDestinationChangedListener
        implements NavController.OnDestinationChangedListener {
 　　...

    @Override
    public void onDestinationChanged(@NonNull NavController controller,
            @NonNull NavDestination destination, @Nullable Bundle arguments) {
        ...
        CharSequence label = destination.getLabel();
        if (!TextUtils.isEmpty(label)) {
            // Fill in the data pattern with the args to build a valid URI
            StringBuffer title = new StringBuffer();
            Pattern fillInPattern = Pattern.compile("\\{(.+?)\\}");
            Matcher matcher = fillInPattern.matcher(label);
            while (matcher.find()) {
                String argName = matcher.group(1);
                if (arguments != null && arguments.containsKey(argName)) {
                    matcher.appendReplacement(title, "");
                    //noinspection ConstantConditions
                    title.append(arguments.get(argName).toString());
                } else {
                    throw new IllegalArgumentException("Could not find " + argName + " in "
                            + arguments + " to fill label " + label);
                }
            }
            matcher.appendTail(title);
            setTitle(title);
        }
        boolean isTopLevelDestination = NavigationUI.matchDestinations(destination,
                mTopLevelDestinations);
        if (drawerLayout == null && isTopLevelDestination) {
            setNavigationIcon(null, 0);
        } else {
            setActionBarUpIndicator(drawerLayout != null && isTopLevelDestination);
        }
    }

    private void setActionBarUpIndicator(boolean showAsDrawerIndicator) {
        boolean animate = true;
        if (mArrowDrawable == null) {
            mArrowDrawable = new DrawerArrowDrawable(mContext);
            // We're setting the initial state, so skip the animation
            animate = false;
        }
        setNavigationIcon(mArrowDrawable, showAsDrawerIndicator
                ? R.string.nav_app_bar_open_drawer_description
                : R.string.nav_app_bar_navigate_up_description);
        float endValue = showAsDrawerIndicator ? 0f : 1f;
        if (animate) {
            float startValue = mArrowDrawable.getProgress();
            if (mAnimator != null) {
                mAnimator.cancel();
            }
            mAnimator = ObjectAnimator.ofFloat(mArrowDrawable, "progress",
                    startValue, endValue);
            mAnimator.start();
        } else {
            mArrowDrawable.setProgress(endValue);
        }
    }
}
```

1. destinationが変わったら、onDestinationChangedメソッドからコールバックされる
2. 設定したlabelから、Actionbarのタイトルを変更する
    - fragmentタグから、labelをセットすることが出来るので、その値がセットされます
3. トップのdestinationの場合、iconがnullになる。そうでない場合は、DrawerArrowDrawableを表示する
    - トップの場合、Toolbar上に戻るボタンが表示されません

という感じになってます。また、DrawerLayoutがある場合、挙動が変わることが分かります。

## 以上を踏まえて、Toolbarの挙動を変えるには

独自に、`NavController.OnDestinationChangedListener`を実装すれば良いことが分かります。

例えば、タイトルの一番最後に動的に文字を追加したい場合は、次のようにします。

```kotlin
var suffix = "suffix" // この値は動的に変わるものとする

navController.addOnDestinationChangedListener { _, _, _ ->
    // ここで文字を追加してあげる
    supportActionBar?.title = "${supportActionBar?.title} $suffix"
}
```

## まとめ: 2行で

- OnDestinationChangedListenerから、Toolbar(ActionBar)をいじっている
- OnDestinationChangedListenerをカスタマイズすれば、挙動をいじれる
