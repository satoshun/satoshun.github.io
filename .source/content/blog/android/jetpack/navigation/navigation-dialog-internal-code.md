+++
date = "Sat Nov  9 06:23:13 UTC 2019"
title = "Android: Navigationのdialogタグ周りのコードちょっと読んでみた"
tags = ["android", "jetpack", "navigation"]
blogimport = true
type = "post"
draft = false
+++

Navigation Component 2.1.0からdialogタグが使えるようになりました。
どんな感じで処理をしているのか気になったので、ざっくりとメモ書き。

## 1. abstract Navigatorクラス

このクラスは最終的にどのように、対象クラスをnavigateされるかを決めるクラスです。
Activityなら、ActivityNavigator。FragmentならFragmentNavigatorが使われます。
dialogの場合は、DialogFragmentNavigator経由でdialogが発火するようになっています。

## 2. NavInlaterでdialogタグの場合、DialogFragmentNavigatorを使うようにしている

NavInflaterは単純なXMLパーサーになっています。<dialog ...> を発見したら、DialogFragmentNavigatorからNavDestinationを作成するようになっています。
このNavDestinationは実際にコールされるときに、DialogFragmentNavigatorを呼び出すような仕組みなっています。

## 3. DialogFragmentNavigatorの中身

一部分を抜粋します。

```java
/*
 * Copyright 2019 The Android Open Source Project
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
...
@Nullable
@Override
public NavDestination navigate(@NonNull final Destination destination, @Nullable Bundle args,
        @Nullable NavOptions navOptions, @Nullable Navigator.Extras navigatorExtras) {
    ...

    final Fragment frag = mFragmentManager.getFragmentFactory().instantiate(
            mContext.getClassLoader(), className);
    if (!DialogFragment.class.isAssignableFrom(frag.getClass())) {
        throw new IllegalArgumentException("Dialog destination " + destination.getClassName()
                + " is not an instance of DialogFragment");
    }
    final DialogFragment dialogFragment = (DialogFragment) frag;
    dialogFragment.setArguments(args);
    dialogFragment.getLifecycle().addObserver(mObserver);

    dialogFragment.show(mFragmentManager, DIALOG_TAG + mDialogCount++);

    return destination;
}
...
```

やっていることは単純で、

1. FragmentFactory経由で対象のDialogFragmentを生成する
2. argumentsのセット
3. ダイアログが終了したかどうかをハンドリングするために、ライフサイクルに登録
4. DialogFragment#showをコールして、ダイアログを出す

これでDialogFragmentを起動しています。

## まとめ

- とっても雑に書きました
- Navigatorっていう抽象クラスがあって、そこを拡張ポイントにして、実装されていた
