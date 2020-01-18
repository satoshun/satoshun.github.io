+++
date = "Sat Jan 18 11:56:31 UTC 2020"
title = "FragmentでViewの参照を持つとメモリリークする話と対策"
tags = ["android", "jetpack", "fragment", "lifecycle"]
blogimport = true
type = "post"
draft = false
+++

View Bindingのドキュメントが更新され、onDestroyViewのタイミングで保持しているBindingの参照を解放するよう文が追記されました。

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

onDestroyViewで参照を解放するコードを書きます。

## 2. AACサンプルで使っているAutoClearedValueを使う

takahiromさんにTwitterで教えてもらったんですが、AACサンプルではDelegationを使って、自動で参照を解放しているようです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">DroiKaigiでは、Adapterとか持ちたい場合もあるので、AACのサンプルにあるAutoCleardValueにしてみました <a href="https://t.co/IUNmeQLzfB">https://t.co/IUNmeQLzfB</a></p>&mdash; takahirom (@new_runnable) <a href="https://twitter.com/new_runnable/status/1217980925008465920?ref_src=twsrc%5Etfw">January 17, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

次のように実装します。

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

内部で、リフレクションを用いており、自動的に生成を行ってくれます。また、AutoClearedValueと同様に、viewLifecycleOwnerLiveDataを購読しており、自動で参照を解放してくれます。

## 4. View.setTag, getTagを使った実装を使う

自動的に解放する部分の実装の話なんですが、setTag、getTagを使った実装でも参照を解放することが可能です。

<script src="https://gist.github.com/satoshun/0185c4231983016f6afa4d8f8c423cd9.js"></script>

viewLifecycleOwnerLiveDataを使わないパターンの実装になります。

## 個人的な感想

AACサンプルで使っているAutoClearedValueを使うのが良いのではと思っています。なぜかっていうと、RecyclerView.AdapterなどのBinding以外でも使うことが出来るからです。より汎用的だと思います。

とはいえ、bindingを保持する変数をvalにしたいよねって話もあると思うので、そのときは3、4の方法を参考にするのが良いと思います。

## まとめ

- FragmentでViewの参照を持つBindingを保持するとメモリリークする
  - 解放しておくとより丁寧
