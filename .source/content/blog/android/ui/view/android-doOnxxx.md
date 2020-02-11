+++
date = "Tue Feb 11 14:02:02 UTC 2020"
title = "doOnNextLayout、doOnLayout、doOnPreDrawの違いと、Coroutineでこれらを動かしてみる"
tags = ["android", "ui", "view"]
blogimport = true
type = "post"
draft = false
+++

タイトルにあるメソッドはJetpack core-ktxに定義されています。

- [doOnNextLayout](https://developer.android.com/reference/kotlin/androidx/core/view/package-summary#doonnextlayout)
- [doOnLayout](https://developer.android.com/reference/kotlin/androidx/core/view/package-summary#doonlayout)
- [doOnPreDraw](https://developer.android.com/reference/kotlin/androidx/core/view/package-summary#doonpredraw)

これらを雰囲気で使っていたので、軽く調べてみました。

## doOnNextLayout

これは、指定したViewがレイアウトされたときに実行されます。
なので、measure、layoutの後にコールバックされます。

注意としては、「既にレイアウト済み かつ 再レイアウトが行われない時」はコールバックされません。

## doOnLayout

doOnNextLayoutと似ているのですが、異なる点は、「レイアウト済み かつ 再レイアウトの要求がない」場合には、即時実行されます。

```kotlin
inline fun View.doOnLayout(crossinline action: (view: View) -> Unit) {
    if (ViewCompat.isLaidOut(this) && !isLayoutRequested) {
        action(this)
    } else {
        doOnNextLayout {
            action(it)
        }
    }
}
```

## doOnPreDraw

描画される前に実行されます。よって、このタイミングではmeasure、layoutは完了していて、描画するぞっていうタイミングでコールバックされます。

`doOnNextLayout`と違うところは、goneでもコールされる点なのかなと思います。
Viewがgoneの場合、`doOnNextLayout`はコールされないですが、`doOnPreDraw`ではコールされます。

## コルーチンと一緒に使う

[chrisbanes/tivi](https://github.com/chrisbanes/tivi)が凄く参考になります。

PreDrawをCoroutineと協調して動くようにしたいなら、次のようになります。

```kotlin
/*
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
suspend fun View.awaitPreDraw() = suspendCancellableCoroutine<Unit> { cont ->
    val listener = OneShotPreDrawListener.add(this) {
        cont.resume(Unit)
    }
    // If the coroutine is cancelled, remove the listener
    cont.invokeOnCancellation {
        listener.removeListener()
    }
}
```

View周りのコードってどうしてもコールバック地獄になりがちだと思うんですが、Coroutineを使うことでフラットに保つことが期待できます。

他にもいろいろ定義してあります。[ViewExtensions.kt](https://github.com/chrisbanes/tivi/blob/master/common-ui-view/src/main/java/app/tivi/extensions/ViewExtensions.kt)


## まとめ

- Coroutine便利〜

## 参考

- [chrisbanes/tivi](https://github.com/chrisbanes/tivi)
- [ViewExtensions.kt](https://github.com/chrisbanes/tivi/blob/master/common-ui-view/src/main/java/app/tivi/extensions/ViewExtensions.kt)
