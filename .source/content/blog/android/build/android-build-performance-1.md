+++
date = "Sat Sep 17 03:43:41 UTC 2022"
title = "Androidビルド速度改善雑記 その1"
tags = ["android", "build", "gradle"]
blogimport = true
type = "post"
draft = true
+++

今回はGradleビルドキャッシュについて話します。
ビルド速度（incremental build）を改善するためには、ビルドキャッシュがいい感じに効いている必要があります。

キャッシュが効いているかを確認する方法を紹介します。
まず、最初に同じコマンドを実行します。

```text
> ./gradlew assembleDebug
> ./gradlew assembleDebug
```

2回目の実行のときに、タスクの実行数が0ならおkです。
実行数は、コマンド実行の最後に出力されます。

```text
> ./gradlew assembleDebug
...
39 actionable tasks: 16 executed, 23 up-to-date
```

この場合は、16タスクが実行されて、23タスクが最新（キャッシュ）となっており、23タスクがキャッシュを使っています。
もう一度同じコマンドを実行したときに、すべてのタスクがキャッシュから使われていれば、いい感じです。

```
> ./gradlew assembleDebug
...
39 actionable tasks: 39 up-to-date
```

これが理想ケースになります。こうなっていない場合には、Gradle Build Scanを使って調査するのが良いと思います。

Build Scanの使い方は簡単で、引数に --scan をつけるだけです。

```shell
> ./gradlew assembleDebug --scan
...
```

これを実行すると、URLが発行されるので、アクセスしてアクティベートします。

アクティベートされると、ビルド情報の詳細が見れます。今回は、再実行されているタスクが知りたいので、 PerformanceタブのTask execution を確認します。

![](https://paper-attachments.dropbox.com/s_0DA7A96A63A1068E4AF158145FFCE4A33C8DE6821B7117A6CF39EAF404C4F867_1663061755425_Screen+Shot+2022-09-13+at+18.34.04.png)

このTasks executedが実行されているタスクになるので、これを確認して、問題があるタスクを調査する感じです。

メンテされていないプラグインであったり、自前のプラグインを使っている場合、キャッシュが考慮されておらず、タスクが再実行されてしまうなどの不都合がありがちなので、注意してください。
また、version codeなど、BuildConfig系の値をビルド毎に更新している場合にも、再ビルドが発生してしまうので、注意が必要です。
