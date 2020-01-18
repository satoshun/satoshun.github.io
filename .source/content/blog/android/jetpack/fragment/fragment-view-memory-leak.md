+++
date = "Sat Jan 18 13:09:08 UTC 2020"
title = "FragmentでViewの参照を持つとメモリリークする話と実装"
tags = ["android", "jetpack", "fragment", "lifecycle"]
blogimport = true
type = "post"
draft = false
+++

View Bindingのドキュメントが更新され、onDestroyViewのタイミングで保持しているBindingの参照を解放する節が追記されました。

[Use view binding in fragments](https://developer.android.com/topic/libraries/view-binding#fragments)

Fragment自体のライフサイクルのほうが、FragmentのViewのライフサイクルより長いので、FragmentでBindingの参照を保持するとリークしてしまうためです。

この記事では、メモリリークをしないために、どのような実装が考えられるかを紹介していきます。

## 1. onDestoryViewで解放する

公式ドキュメントに載っている方法です。

```kotlin
// onCreatedViewで初期化
private var _binding: ResultProfileBinding? = null
private val binding get() = _binding!!

override fun onDestroyView() {
  _binding = null
}
```

onDestroyViewで参照を解放するコードを書きます。シンプルですが、冗長なのかなと思います。

## 2. AACサンプルで使っているAutoClearedValueを使う

takahiromさんにTwitterで教えてもらったんですが、AACサンプルではDelegationを使って、自動で参照を解放しているようです。

---

<blockquote class="twitter-tweet" data-conversation="none" data-theme="dark"><p lang="ja" dir="ltr">DroiKaigiでは、Adapterとか持ちたい場合もあるので、AACのサンプルにあるAutoCleardValueにしてみました <a href="https://t.co/IUNmeQLzfB">https://t.co/IUNmeQLzfB</a></p>&mdash; takahirom (@new_runnable) <a href="https://twitter.com/new_runnable/status/1217980925008465920?ref_src=twsrc%5Etfw">January 17, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

---

次のように使います。

```kotlin
// onCreatedViewで初期化する
var binding by autoCleared<RepoFragmentBinding>()
var adapter by autoCleared<RepoFragmentAdapter>()
```

AutoClearedValueは、`viewLifecycleOwnerLiveData`を購読しており、onDestroyViewのタイミングで、自動的に参照を解放してくれます。また、ReycyclerView.Adapterでも同様に使うことが出来ます。

詳細な実装は[AutoClearedValue.kt](https://github.com/android/architecture-components-samples/blob/master/GithubBrowserSample/app/src/main/java/com/android/example/github/util/AutoClearedValue.kt)を見てください。

## 3. DataBinding-Ktxを使う

[DataBinding-ktx](https://github.com/wada811/DataBinding-ktx)というライブラリを使うことで、valで定義することが可能になります。

```kotlin
// onCreatedViewで初期化しないのでも済む
private val binding: ViewBindingFragmentBinding by viewBinding()
```

内部で、リフレクションを用いており、ViewBindingの場合でも自動的にBindingを生成してくれます。
また、AutoClearedValueと同様に、viewLifecycleOwnerLiveDataを購読しており、自動で参照を解放してくれます。

## 4. View.setTag, getTagを使った実装を使う

自動的に解放する部分の実装の話なんですが、setTag、getTagを使った実装でも参照を解放することが可能です。

```kotlin
class MainFragment : Fragment(R.layout.main_frag2) {
  private val binding: MainFrag2Binding by viewBinding()
}

// ViewDataBinding.kt
fun <T : ViewDataBinding> Fragment.viewBinding(): ReadOnlyProperty<Fragment, T> =
  object : ReadOnlyProperty<Fragment, T> {
    override fun getValue(thisRef: Fragment, property: KProperty<*>): T {
      val view = thisRef.view!!
      var binding = view.getTag(R.id.fragment_binding) as? T
      if (binding == null) {
        binding = DataBindingUtil.bind(view)
        view.setTag(R.id.fragment_binding, binding)
      }
      return binding!!
    }
  }
```

[Gist/サンプルコード](https://gist.github.com/satoshun/0185c4231983016f6afa4d8f8c423cd9)

viewLifecycleOwnerLiveDataを使わないパターンの実装になります。また、このコードではFragmentのコンストラクタからレイアウトIDを渡すことを想定しています。

この例では、DataBindingを想定していますが、ViewBindingで使う場合には、リフレクションを用いるか、もしくはファクトリを渡す必要があります。

リフレクションを使う場合は、次のようになります。

```kotlin
inline fun <reified T : ViewBinding> Fragment.viewBinding(): ReadOnlyProperty<Fragment, T> =
  object : ReadOnlyProperty<Fragment, T> {
    override fun getValue(thisRef: Fragment, property: KProperty<*>): T {
      val view = thisRef.view!!
      var binding = view.getTag(R.id.fragment_binding) as? T
      if (binding == null) {
        val method = T::class.java.getMethod("bind", View::class.java)
        binding = method.invoke(null, view) as T
        view.setTag(R.id.fragment_binding, binding)
      }
      return binding
    }
  }
```

## 個人的な感想

AACサンプルで使っているAutoClearedValueを使うのが良いのではと思っています。なぜかっていうと、RecyclerView.AdapterなどのBinding以外でも使うことが出来るからです。より汎用的だと思います。

とはいえ、bindingを保持する変数をvalにしたいよねって話もあると思うので、そのときは3、4の方法を参考にするのが良いと思います。

## まとめ

- FragmentでViewの参照を持つBindingを保持するとメモリリークする
  - 解放しておくとより丁寧
- AutoClearedValueが汎用的に使える
- Bindingの参照をvalにしたいなら、DataBinding-Ktxか、4の方法を参考にすると良さそう
