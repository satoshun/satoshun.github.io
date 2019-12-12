+++
date = "Thu Dec 12 12:25:11 UTC 2019"
title = "GitHub Actionsでエミュレータテストをする"
tags = ["ci", "githubactions"]
blogimport = true
type = "post"
draft = false
+++

GitHub Actionsでエミュレータでテストを回したかったので調べてみました。

結論、AutoDisposeでエミュレータでテストを行っていたので、それをアレンジする感じがいいと思います。詳しくは [AutoDispose/ci.yml](https://github.com/uber/AutoDispose/blob/master/.github/workflows/ci.yml) を見てください。

## エミュレータテストをするスクリプト

特にキャッシュとかを考えないなら、以下のスクリプトで動きます。

```
name: CI

on: [push, pull_request]

jobs:
  build:
    name: JDK 1.8
    runs-on: macOS-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v1
      - name: Install JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - name: Install Android SDK
        uses: malinskiy/action-android/install-sdk@release/0.0.5
      - name: Build project
        run: ./gradlew assemble --stacktrace
      - name: Run instrumentation tests
        uses: malinskiy/action-android/emulator-run-cmd@release/0.0.5
        with:
          cmd: ./gradlew connectedCheck --stacktrace
          api: 25
          tag: default
          abi: x86
```

[malinskiy/action-android](https://github.com/Malinskiy/action-android)という、便利GitHub Actionがあるので、それを呼び出すだけでおｋです。

あとは、`cmd: ./gradlew connectedCheck --stacktrace`みたいな感じで、自分のプロジェクトに適したテストコマンドをコールするだけでいけます。stacktraceを出しておくと、デバッグのときなどに便利です。

[結果](https://github.com/satoshun-kotlin-example/CoroutineCatalog/commit/0b8336d8ff2c7cc37510857c816e10c3ea31b295/checks?check_suite_id=355095099) はこんな感じになります。

{{< figure src="/blog/ci/github-result.png" width="600" >}}

## まとめ

- GitHub Actionsはいいぞ〜
