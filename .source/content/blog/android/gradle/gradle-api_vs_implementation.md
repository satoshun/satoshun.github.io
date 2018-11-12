+++
date = "2018-11-12"
title = "Gradle: Daggerでapiとimplementationどちらを使うか議論されている"
tags = ["dagger", "android"]
blogimport = true
type = "post"
draft = true
+++

GradleでcompileがDeprecatedになり、implementationまたはapiを使うことが推奨されています。
その変更に合わせて多くのライブラリのREADMEのcompileが置き換わりました。

Daggerでもcompileをimplementationに置き換えるPRが出されました。https://github.com/google/dagger/pull/1130
内容が興味深かったのでまとめてみようと思います。

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

まず最初のPRは上記のような差分になっていました。compileをimplementationに置き換えています。

ここでjakeさんが

> I think this one is somewhat debatable, but you're likely to mark it as implementation in every module that contains the compiler so it's probably a non-problem.

とコメントしました。「Daggerコンパイラの依存はすべてのモジュールで明示的に書くだろうから、多分implementationで問題ないけど、議論の余地はある。」と。

この議論を生んだ要因として、Daggerライブラリの性質があります。
Daggerをライブラリで使ったときに、その中でComponentやModuleを使います。クライアント側ではComponentとModuleを使う必要はありません。
しかし、クライアント側では`Lazy`、`Provider`などのAPIは使います。これらのAPIを使うためにはDaggerを依存に指定しておく必要があります。
なので、このライブラリを依存に持つようなモジュールは`Lazy`、`Provider`を使うことになるので、結局Daggerに依存することが確定しているなら`api`で指定したほうが良いのではという話です。
