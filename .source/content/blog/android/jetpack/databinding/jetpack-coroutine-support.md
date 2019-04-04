+++
date = "Thu Apr  4 00:46:00 UTC 2019"
title = "DataBindingとActivityのコンストラクタ Layout Id指定を一緒に使う"
tags = ["android", "jetpack", "databinding", "ktx"]
blogimport = true
type = "post"
draft = false
+++

JetpackのActivityとFragmentのコンストラクタに、Layout Idが指定できるようになりました。

- [Activity Version 1.0.0-alpha06](https://developer.android.com/jetpack/androidx/releases/activity#1.0.0-alpha06)
- [Fragment Version 1.1.0-alpha06](https://developer.android.com/jetpack/androidx/releases/fragment#1.1.0-alpha06)

これは、次のように使うことが出来ます。

```kotlin
class MainActivity : AppCompatActivity(R.layout.main_act)

class MainFragment : Fragment(R.layout.main_frag)
```

Activityの場合は、setConentViewが。Fragmentの場合はonCreateViewがそれぞれ省略することが出来ます。

ここからが本題です。これをDataBindingと一緒に使うなら、次のようになるかなと思います。

### Activityの場合

まずはActvityの例です。

```kotlin
// 拡張関数を定義しておく
fun <T : ViewDataBinding> ComponentActivity.bindView(): T =
  DataBindingUtil.bind(getContentView())!!

private fun Activity.getContentView(): View =
  findViewById<ViewGroup>(android.R.id.content)[0]


// MainActivity.kt
class MainActivity : AppCompatActivity(R.layout.main_act) {
  private lateinit var binding: MainActBinding

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = bindView()
    ...
  }
}
```

また、Activityの場合に限り、by lazyと組み合わせることも可能です。

```kotlin
class MainActivity : AppCompatActivity(R.layout.main_act) {
  private val binding by lazy { bindView<MainActBinding>() }

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    ...
  }
}
```

### Fragmentの場合

次にFragmentの例になります。

```kotlin
// 拡張関数を定義しておく
fun <T : ViewDataBinding> Fragment.bindView(): T = DataBindingUtil.bind(view!!)!!


// MainFragment.kt
class MainFragment : Fragment(R.layout.main_frag) {
  private lateinit var binding: MainFragBinding

  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    binding = bindView()
    ...
  }
}
```

Kotlinの拡張関数でやるなら、こんな感じになると思ってます。

## まとめ

- コンストラクタにLayoud Idが指定できるようになり、それを使いたいなら、DataBindingの取得の仕方が少し変わりそう
  - Layout Id指定は必須ではないので、必ずしも使う必要はないと思います
- KotlinのDelegationを使えば、もっといい感じに書けるかもしれない
- サンプルは[satoshun/DataBindingContentLayoutIdExample](https://github.com/satoshun-android-example/DataBindingContentLayoutIdExample)にあります😃

---

内容におかしい点や、もっとこうしたほうがいいよって！！いうのがあればTwitterなどから教えてもらえればとても嬉しいです😊
