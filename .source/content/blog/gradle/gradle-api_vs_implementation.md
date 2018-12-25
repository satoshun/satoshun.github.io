+++
date = "2018-12-08"
title = "Gradle: Dagger2でapiとimplementationどちらを使うか議論されている"
tags = ["gradle", "dagger", "android"]
blogimport = true
type = "post"
draft = false
+++

**注意** この記事はapiとimplementationの説明をする類の記事ではありません。

Gradleで`compile`がDeprecatedになり、implementationまたはapiを使うことが推奨されています。
それに合わせて多くのライブラリのREADMEのcompileがimplementationまたはapiに置き換わりました。

Dagger2でもcompileをimplementationに置き換えるPRが出されました。https://github.com/google/dagger/pull/1130

内容が興味深かったのでまとめてみようと思います。

まず最初のPRは次の差分になっていました。compileをimplementationに置き換えています。

```
// Add Dagger dependencies
dependencies {
  - compile 'com.google.dagger:dagger:2.x'
  + implementation 'com.google.dagger:dagger:2.x'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.x'
}

- compile 'com.google.dagger:dagger-android:2.x'
- compile 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
+ implementation 'com.google.dagger:dagger-android:2.x'
+ implementation 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
```

ここでjakeさんが

> I think this one is somewhat debatable, but you're likely to mark it as implementation in every module that contains the compiler so it's probably a non-problem.

とコメントしました。「Daggerコンパイラの依存はすべてのモジュールで明示的に書くだろうから、多分implementationで問題ないけど、議論の余地はある。」と。

この議論を生んだ要因として、Daggerライブラリの性質があります。
Daggerをライブラリで使ったときに、そのライブラリ内ではComponentやModuleを使うため、当然DaggerのAPIを使うことになります。
しかし、クライアント側ではComponentとModuleなどのDaggerのAPIを使う必要は必ずしもありません。ただ、DaggerではLazy、ProviderなどのAPIも定義されており、これらのAPIはクライアント側で使うことが予想されます。
なので、結局このライブラリを依存に持つようなモジュールはLazy、Providerを使うことになるので、最終的にDaggerに依存することが確定しているなら、implementationではなく、apiで指定したほうが良いのではという話です。ただし、プロジェクトのモジュール構成によってはimplementationのほうがふさわしい場合もあるので、その場合はimplementationに置き換えることが期待されています。

最終的なこの議論の着地として、implementationを使うかapiを使うかはプロジェクトに強く依存するのでどちらが正しいかはない。そのため、今回置き換えたいcompileはapiとほぼ同等の意味を持つので、置き換えるならapiのほうがふさわしいんじゃないか、今までと同じ動作をするので安全じゃないかという感じでまとまりました。

現状の差分は以下です。

```
-  compile 'com.google.dagger:dagger:2.x'
+  api 'com.google.dagger:dagger:2.x'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.x'
}

+ For more information on api vs implementation please see https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration?utm_source=android-studio#new_configurations

- compile 'com.google.dagger:dagger-android:2.x'
- compile 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
+ implementation 'com.google.dagger:dagger-android:2.x'
+ implementation 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
```

dagger-androidがimplementationのままなのは、こちらにはLazyやProviderなどのクライアントライブラリが使うであろうAPIが含まれていないためだと思います。

## まとめ

- READMEを更新するのにかなり熱い議論をしているのがすごい印象的でした
  - 手を抜かないって大事なんだなって
- より詳しい議論は、[ISSUE](https://github.com/google/dagger/pull/1130)を見て下さい😃
