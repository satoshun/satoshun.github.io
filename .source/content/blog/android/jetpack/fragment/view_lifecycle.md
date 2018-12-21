+++
date = "Fri Dec 21 07:08:34 UTC 2018"
title = "FragmentとgetViewLifecycleの話"
tags = ["android", "jetpack", "fragment", "livedata", "lifecycle"]
blogimport = true
type = "post"
+++

この記事ではFragmentでLiveDataにObserverを登録するときは`Fragment#getViewLifecycle`を使うと良いという話をします。

まず、Fragmentのおおまかなライフサイクルは次のようになっています。

- onAttach
- onCreate
    - onCreateView
    - onViewCreated
        - ...
    - onDestoryView
- onDestroy
- onDetach

ここで重要なのは、`onDestroy`が呼ばれることなく、複数回`onCreateView`が呼ばれる可能性がある点です。

例えば、次のコードは間違っている可能性があります。

```kotlin
class MainFragment: Fragment() {
    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.data.observe(this, Observer {
            ...
        })
    }
}
```

なぜなら、LiveDataに渡したthis（LifecycleOwner）は、自身のライフサイクルに駆動するためです。
このObserverが開放されるタイミングは、Fragment#onDestroyがコールされたタイミングになります。
しかし前述したとおり、Fragment#onDestroyがコールされずに、複数回onCreateViewがコールされる可能性があるため、前のObserverが開放されずに残ってしまいます。

前述のコードのObserverはFragment本体のLifecycleに駆動されるのではなく、FragmentのViewに駆動するため、この問題が起こります。
よって、FragmentにはView用のLifecycleが用意されています。それが、`Fragment#getViewLifecycle`です。

前述のコードは次のように書くことが出来ます。

```kotlin
class MainFragment: Fragment() {
    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // ViewのLifecycleOwnerを渡す
        viewModel.data.observe(viewLifecycleOwner, Observer {
            ...
        })
    }
}
```

このように書くことで、ObserverがViewのライフサイクルに駆動するため、複数のObserverが登録される問題を回避することが出来ます！！

## 補足1

Observerが開放されるタイミングはonDestroyがコールされるタイミングとは別にもう1つあります。
それは、Observerへの参照がなくなったタイミングです。内部的にObserverはWeakReferenceで保持されており、参照が無くなったタイミングでGCされます。

## 補足2

Observerの重複登録問題はattach/detachを繰り返す場合におこります。
サンプルコードは次のようになります。

```kotlin
while (true) {
    delay(3000)
    supportFragmentManager.commitNow {
        if (fragment.isDetached) attach(fragment)
        else detach(fragment)
　  }
}
```
