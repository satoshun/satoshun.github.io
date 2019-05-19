+++
date = "Sun May 19 22:11:08 UTC 2019"
title = "雑メモ: ViewModel SavedStateのコードリーディング"
tags = ["android", "jetpack", "viewmodel", "savedstate"]
blogimport = true
type = "post"
draft = true
+++

ViewModelのSavedStateがどのように実現しているのか、内部でどのように動作しているのか気になったので、ソースコードを読んでみました。

この記事のソースコードは全て、下記のライセンスに従います。

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

`onSaveInstanceState`などのタイミングでコールされ、`outState`に保存したいBundleを返り値として返却します。

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

SavedStateProviderを集約し、`performRestore`、`performSave`によって、savedState、outStateから読み込み、書き込みを行う。

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

SavedStateRegistryOwnerは`ComponentActivity`や`Fragment`が実装しています。なので、LifecycleOwnerのように、これらのコンポーネントが本体となります。

コードリーディングするときは、`ComponentActivity`または`Fragment`を起点にすると実行の流れが掴みやすいです。

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

`SavedStateRegistryOwner`のための実装です。ComponentActivityやFragmentではこのクラスを介して、saved stateを扱います。

これがざっくりとした、全体像です。

## 次にViewModelのsaveの流れ

ViewModel saved stateのsave時の実行の流れを見ながら、コードリーディングをしていきます。

1. ViewModelの生成

ViewModel + saved stateのときは、`SavedStateVMFactory`を使います。
これは、ViewModelインスタンスに`SavedStateHandle`を渡すために必要なFactoryです。

SavedStateVMFactoryでは、

- `SavedStateHandle`の生成
    - SavedStateHandleでは、`SavedStateProvider`と、outStateに保存したい状態を保持している
- SavedStateHandleをViewModelに渡す
- `SavedStateRegistry`に、SavedStateHandleで保持しているSavedStateProviderを登録する

2. `ComponentActivity#onSaveInstanceState`

```java
public class ComponentActivity ... {
    //
    private final SavedStateRegistryController mSavedStateRegistryController =
            SavedStateRegistryController.create(this);

    protected void onSaveInstanceState(@NonNull Bundle outState) {
        ...
        mSavedStateRegistryController.performSave(outState);
    }
}
```

まずは、SavedStateRegistryControllerに、`outState`に書き込み/保存を頼みます。

3. `SavedStateRegistryController#performSave`

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

4. `SavedStateRegistry#performSave`

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

Lifecycleに準拠したcomponentを**lifecycle aware**といいますが、SavedStateに準拠したcomponentは**savedstate aware**と表現されるかもしれません。
