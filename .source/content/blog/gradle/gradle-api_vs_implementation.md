+++
date = "2018-12-08"
title = "Gradle: Daggerでapiとimplementationどちらを使うか議論されている"
tags = ["gradle", "dagger", "android"]
blogimport = true
type = "post"
draft = true
+++

Gradleで`compile`がDeprecatedになり、implementationまたはapiを使うことが推奨されています。
それに合わせて多くのライブラリのREADMEのcompileがimplementation or apiに置き換わりました。

Daggerでもcompileをimplementationに置き換えるPRが出されました。https://github.com/google/dagger/pull/1130

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
Daggerをライブラリで使ったときに、その中でComponentやModuleを使います。ライブラリ側ではDaggerのAPIを使うことになります。
しかし、クライアント側ではComponentとModuleなどのDaggerのAPIを使う必要は必ずしもありません。ただDaggerではLazy、ProviderなどのAPIも定義されており、これらのAPIはクライアント側で使うことが予想されます。

なので、結局このライブラリを依存に持つようなモジュールはLazy、Providerを使うことになるので、結局Daggerに依存することが確定しているなら、implementationではなく、apiで指定したほうが良いのではという話です。

また、最終的なこの議論の着地として、非推奨になったcompileはapiとほぼ同等の意味を持ちます。なので、単純にREADMEを更新するならapiのほうがふさわしいんじゃないか、安全じゃないかという感じでまとまりました。安全に振った感じです。
