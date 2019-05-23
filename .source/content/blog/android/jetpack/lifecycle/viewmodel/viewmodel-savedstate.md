+++
date = "Tue May 21 23:59:53 UTC 2019"
title = "雑メモ: ViewModel SavedStateのコードリーディング"
tags = ["android", "jetpack", "viewmodel", "savedstate"]
blogimport = true
type = "post"
draft = false
+++

ViewModelのSavedStateがどのように実現しているのか、内部でどのように動作しているのか気になったので、ソースコードを読んでみました。

この記事のソースコードは全て、下記のライセンスに従います。

```xml
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
```

## まずsavedstate本体のコード

主な登場クラスは以下です。

### SavedStateRegistry.SavedStateProvider

```java
/**
 * This interface marks a component that contributes to saved state.
 */
public interface SavedStateProvider {
   /**
    * Called to retrieve a state from a component before being killed
    * so later the state can be received from {@link #consumeRestoredStateForKey(String)}
    *
    * @return S with your saved state.
    */
    @NonNull
    Bundle saveState();
}
```

`onSaveInstanceState`のタイミングでコールされ、`outState`に追加で、保存したい値をBundle型で返却します。

### SavedStateRegistry

```java
/**
 * An interface for plugging components that consumes and contributes to the saved state.
 *
 * <p>This objects lifetime is bound to the lifecycle of owning component: when activity or
 * fragment is recreated, new instance of the object is created as well.
 */
public final class SavedStateRegistry {
    public Bundle consumeRestoredStateForKey(@NonNull String key)
    public void registerSavedStateProvider(@NonNull String key,
            @NonNull SavedStateProvider provider)
    public void unregisterSavedStateProvider(@NonNull String key)
    public boolean isRestored()
    public void runOnNextRecreation(@NonNull Class<? extends AutoRecreated> clazz)

    void performRestore(@NonNull Lifecycle lifecycle, @Nullable Bundle savedState)
    void performSave(@NonNull Bundle outBundle)
}
```

先程のSavedStateProviderを`registerSavedStateProvider`メソッドを通して集約し、`performRestore`、`performSave`によって、savedStateから読み込み、outStateに保存します。

### SavedStateRegistryOwner

```java
/**
 * A scope that owns {@link SavedStateRegistry}
 */
public interface SavedStateRegistryOwner extends LifecycleOwner {
    /**
     * Returns owned {@link SavedStateRegistry}
     *
     * @return a {@link SavedStateRegistry}
     */
    @NonNull
    SavedStateRegistry getSavedStateRegistry();
}
```

SavedStateRegistryOwnerは先程の`SavedStateRegistry`のOwnerになります。これは、`ComponentActivity`や`Fragment`が実装しています。LifecycleOwner、Lifecycleのような実装です。

### SavedStateRegistryController

```java
/**
 * An API for {@link SavedStateRegistryOwner} implementations to control {@link SavedStateRegistry}.
 * <p>
 * {@code SavedStateRegistryOwner} should call {@link #performRestore(Bundle)} to restore state of
 * {@link SavedStateRegistry} and {@link #performSave(Bundle)} to gather SavedState from it.
 */
public final class SavedStateRegistryController {
    public SavedStateRegistry getSavedStateRegistry()
    public void performRestore(@Nullable Bundle savedState)
    public void performSave(@NonNull Bundle outBundle)

    public static SavedStateRegistryController create(@NonNull SavedStateRegistryOwner owner)
}
```

`SavedStateRegistryOwner`のための実装です。ComponentActivityやFragmentではこのクラスを介して、Bundleから値を復元/restoreしたり、保存/saveします。

これがSavedStateで使われている主なクラスになります。

次に、ViewModelのsaveの実行の流れを見てみます。

## ViewModelのsaveの実行の流れ

ViewModelのSavedStateのsaveの実行の流れを見ながら、コードリーディングをしていきます。

### 1. ViewModelを生成する

ViewModelとSavedStateを一緒に扱い時は、`SavedStateVMFactory`を使います。
これは、ViewModelインスタンスに`SavedStateHandle`インスタンスを渡すために必要なFactoryです。

```kotlin
// thisはFragmentActivity
val vm = ViewModelProvider(this, SavedStateVMFactory(this))
    .get(MyViewModel::class.java)
```

こう書くことで、MyViewModelで`SavedStateHandle`を受け取ることが出来ます。

SavedStateVMFactoryでは以下の処理を行っています。

- `SavedStateHandle`インスタンスの生成
    - SavedStateHandleでは、`SavedStateProvider`の実装と、outStateに保存/saveしたい状態を保持している
- 生成したSavedStateHandleインスタンスをViewModelに渡す
- `SavedStateRegistry`に、SavedStateHandleで保持しているSavedStateProviderを登録する

### 2. `ComponentActivity#onSaveInstanceState`

```java
public class ComponentActivity ... {
    private final SavedStateRegistryController mSavedStateRegistryController =
            SavedStateRegistryController.create(this);

    protected void onSaveInstanceState(@NonNull Bundle outState) {
        ...
        mSavedStateRegistryController.performSave(outState);
    }
}
```

まずは、SavedStateRegistryControllerに、`outState`に保存を頼みます。

### 3. `SavedStateRegistryController#performSave`

```java
public final class SavedStateRegistryController {
    private final SavedStateRegistry mRegistry;

    ...

    public void performSave(@NonNull Bundle outBundle) {
        mRegistry.performSave(outBundle);
    }
}
```

`SavedStateRegistry`に処理を委譲します。

### 4. `SavedStateRegistry#performSave`

```java
public final class SavedStateRegistry {
    private SafeIterableMap<String, SavedStateProvider> mComponents =
            new SafeIterableMap<>();
    private Bundle mRestoredState;

    ...

    void performSave(@NonNull Bundle outBundle) {
        Bundle components = new Bundle();
        if (mRestoredState != null) {
            components.putAll(mRestoredState);
        }
        for (Iterator<Map.Entry<String, SavedStateProvider>> it =
                mComponents.iteratorWithAdditions(); it.hasNext(); ) {
            Map.Entry<String, SavedStateProvider> entry1 = it.next();
            components.putBundle(entry1.getKey(), entry1.getValue().saveState());
        }
        outBundle.putBundle(SAVED_COMPONENTS_KEY, components);
    }
}
```

`outBundle#putBundle`を通して、保存を行います。今で、直接Activityの`onSaveInstanceState`をoverrideして書いていた処理がここに移ったイメージです。

### 5. SavedStateHandleの`SavedStateProvider`の実装

実際に保存される内容を決めるのはSavedStateHandleのSavedStateProviderの実装/中身になります。

実装は次のようになっています。

```java
final Map<String, Object> mRegular;

private static final String VALUES = "values";
private static final String KEYS = "keys";

private final SavedStateProvider mSavedStateProvider = new SavedStateProvider() {
    @SuppressWarnings("unchecked")
    @NonNull
    @Overrides
    public Bundle saveState() {
        Set<String> keySet = mRegular.keySet();
        ArrayList keys = new ArrayList(keySet.size());
        ArrayList value = new ArrayList(keys.size());
        for (String key : keySet) {
            keys.add(key);
            value.add(mRegular.get(key));
        }

        Bundle res = new Bundle();
        // "parcelable" arraylists - lol
        res.putParcelableArrayList("keys", keys);
        res.putParcelableArrayList("values", value);
        return res;
    }
};
```

Mapに保存したい値を保持しておいて、それをBundleに書き出すだけです。

ざっくりと保存の流れはこんな感じです:D

## まとめ

- LifecycleOwnerのような感じで実装されていた
- SavedStateProviderを実装して、registerSavedStateProviderに渡せば、誰でもカスタムのSavedStateが書ける😃
