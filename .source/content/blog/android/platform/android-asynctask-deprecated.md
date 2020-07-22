+++
date = "Wed Jul 22 02:58:19 UTC 2020"
title = "Android: RIP AsyncTask"
tags = ["android"]
blogimport = true
type = "post"
draft = false
+++

javadocコメントとコミットログは次のようになっています。

> AsyncTask was intended to enable proper and easy use of the UI thread. However, the most common use case was for integrating into UI, and that would cause Context leaks, missed callbacks, or crashes on configuration changes. It also has inconsistent behavior on different versions of the platform, swallows exceptions from {@code doInBackground}, and does not provide much utility over using {@link Executor}s directly.

[https://android-review.googlesource.com/c/platform/frameworks/base/+/1156409](https://android-review.googlesource.com/c/platform/frameworks/base/+/1156409)

AsyncTaskの目的は、UIスレッドを簡単に扱うためのものだけど、Contextのリークや、configuration changeなどでクラッシュしてしまうのでそもそも目的を達成していない。また、Androidのバージョンごとに振る舞いに一貫性がないのも問題だと。ただでさえ、非同期プログラミングは難しいのに、バージョンごとに振る舞いが異なっていたらしんどさ倍増って話だと言う話です。

もう少し詳しく見ていきます。

### Contextのリークや、configuration changeなどでクラッシュしてしまう

この状態は、AsyncTaskを正しく使った場合は起こらないと認識しています。AsyncTaskには、RxJavaはKotlin Coroutineと同様に、タスクをcancelするためのメソッドがあります。このメソッドを `onDestory` などの適切なタイミングでコールしてあげればリーク、クラッシュを防ぐごとが出来ます。この問題は、他のフレームワークでも同様に起こるので、AsyncTaskの問題というよりは、非同期プログラミングの問題なのかなと思っています。ただ、Coroutineにはstructured concurrency、RxJavaにはRxAutoDisposeがあるなど、これらのミスが起きにくい環境にはなっています。

### 3つのgenerics型がしんどかった

AsyncTaskのクラス宣言は次のようになっています。

```java
public abstract class AsyncTask<Params, Progress, Result>
```

個人的には、3つのgenerics型が何となく難しいと感じていました。当時を思い返すと、よく順番が分からなくなっていた気がします。

### RxJavaなどの他のエコシステムが栄えた

AsyncTaskとかLoaderとかそこらへんのが難しいなぁとなっているところに、RxJavaのようなイケてるフレームワークが出てきて、銀の弾丸だぞっていう感じで広告されたことで、一気に端に追いやられてしまったのかなと思っています。他には、RxJavaはJava8のラムダ式と相性が良い、当時retrolambdaが流行っていたなどの理由もあると思います。