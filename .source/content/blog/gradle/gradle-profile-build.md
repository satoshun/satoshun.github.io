+++
date = "Sun Feb  6 03:39:04 UTC 2022"
title = "Gradle Profilerを使って、ビルドを最適化する"
tags = ["gradle", "android"]
blogimport = true
type = "post"
draft = false
+++

開発を高速化するために、ビルド速度を改善することは重要です。
Gradleの設定を見直すことで、ビルド速度を改善できる可能性があります。

この記事では、Gradle Profilerを使って、自分の環境において最適そうな設定を見つける方法を紹介します。

## Gradle Profilerとは?

[Gradle Profiler](https://github.com/gradle/gradle-profiler) は、Gradleのプロファイリング、ベンチマークを計測してくれるツールです。

Gradle Profilerを使うことで、最適なGradleの設定を見つけだすことが出来ます。

## 良さそうな設定を見つける 

まず、最初にシナリオファイルを作成します。これは、Gradle Profilerを実行する際に必要なファイルで、記述したシナリオ通りに実行してくれます。

今回は、次のようなシナリオを作りました。

```txt
# scenario.txt
clean_build_2gb_2workers {
    tasks = [":app:assembleDebug"]
    gradle-args = ["--max-workers=2"]
    jvm-args = ["-Xmx2048m"]
    cleanup-tasks = ["clean"]
}

clean_build_4gb_4workers {
    tasks = [":app:assembleDebug"]
    gradle-args = ["--max-workers=4"]
    jvm-args = ["-Xmx4096m"]
    cleanup-tasks = ["clean"]
}
```

このシナリオでは、worker数と、メモリの値をそれぞれ設定しています。

このシナリオは、Gradle Profilerコマンドを使って、次のように実行できます。

```
gradle-profiler --benchmark --project-dir . --scenario-file scenarios.txt
```

これを実行すると、ビルドにどれくらい時間が掛かったかの出力してくれます。

---

{{< figure src="/blog/gradle/gradle_profiler_scenario1.png" width="800">}}

---

この結果から、僕の環境 + このプロジェクトの場合、2GB、2workerよりも4GB、4workerで設定したほうが良さそうなことが分かります。

このようにして、Gradle Profilerを使うことで、自分の環境において、最適そうな設定を探すことが出来ます。

## まとめ

今回は、Gradle Profilerを使って、ビルドの高速化をする方法を紹介しました。
今回は単純なシナリオを使いましたが、他にもGCの設定を変えるとか、細かい設定があると思うので、そこのあたりもシナリオに入れて計測してみると良いと思います。

また、シナリオの実行にはめちゃくちゃ時間が掛かるので、夜とか、パソコンを触らない時間に実行すると良いと思います😃

## 参考

- https://developer.android.com/studio/build/profile-your-build
- https://github.com/gradle/gradle-profiler
