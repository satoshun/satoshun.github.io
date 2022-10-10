+++
date = "Mon Oct 10 07:49:35 UTC 2022"
title = "Gradle: Javaのバージョン差異によるビルド速度の変化"
tags = ["gradle", "android"]
blogimport = true
type = "post"
draft = false
+++

Javaのバージョンを変更することで、Androidプロジェクトのビルド速度がどのくらい変化するかについて確認してみました。

環境は次のようにしました。

- Mac M1 Max
- Gradle: 7.5.1
- Java:
  - 11.0.16.1
  - 17.0.4

この環境で、Java11と17でそれぞれ10回 `./gradlew assembleDebug --rerun-tasks`を実行します。

Java11の場合

```text
BUILD SUCCESSFUL in 2m 36s
BUILD SUCCESSFUL in 2m 29s
BUILD SUCCESSFUL in 2m 30s
BUILD SUCCESSFUL in 2m 28s
BUILD SUCCESSFUL in 2m 30s
BUILD SUCCESSFUL in 2m 30s
BUILD SUCCESSFUL in 2m 30s
BUILD SUCCESSFUL in 2m 34s
BUILD SUCCESSFUL in 2m 31s
BUILD SUCCESSFUL in 2m 32s
```

Java17の場合

```text
BUILD SUCCESSFUL in 2m 13s
BUILD SUCCESSFUL in 2m 15s
BUILD SUCCESSFUL in 2m 13s
BUILD SUCCESSFUL in 2m 12s
BUILD SUCCESSFUL in 2m 16s
BUILD SUCCESSFUL in 2m 13s
BUILD SUCCESSFUL in 2m 18s
BUILD SUCCESSFUL in 2m 15s
BUILD SUCCESSFUL in 2m 20s
BUILD SUCCESSFUL in 2m 13s
```

僕の環境の場合には、Java17だとちょっと早くなりました。
使っているPCや、Gradleのバージョン、Androidプロジェクトの構成によって、結果は異なる可能性はありますが、少しでもビルドを早くしたい人は試してみてください。


## Java17のインストール方法について

sdkmanを使うと、Javaのバージョンを管理しやすいのでおすすめです

- https://sdkman.io/
