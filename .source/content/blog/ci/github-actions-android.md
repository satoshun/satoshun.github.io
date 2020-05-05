+++
date = "Tue May  5 06:11:48 UTC 2020"
title = "GitHub Actions: AndroidプロジェクトのGradle周りの設定メモ"
tags = ["ci", "githubactions"]
blogimport = true
type = "post"
draft = false
+++

GitHub Actionのスペック

- 7Gメモリの2 CPU

## こんな感じが良さそう

```groovy
org.gradle.daemon=false
org.gradle.parallel=true
org.gradle.jvmargs=-Xmx5120m
org.gradle.workers.max=2

kotlin.incremental=false
kotlin.compiler.execution.strategy=in-process

kapt.use.worker.api=false
kapt.incremental.apt=false

android.databinding.incremental=false
android.lifecycleProcessor.incremental=false
```

- CatchUpアプリを参考にして、上の設定をCI実行時に `~/gradle/gradle.properites` に配置するようにした
    - プロジェクト直下の `gradle.properties` より優先度が高いのでそうした
- kaptの設定はいらないかもしれない
    - worker使わないほうが安定するんじゃないかなーと思って、falseにしてみた
- incremental系は、全部offで良いと思っている


## 考慮したほうが良さそうなプロパティ

- org.gradle.jvmarg
    - Xmx
    - -XX:MaxPermSize
    - -XX:+HeapDumpOnOutOfMemoryError
- org.gradle.daemon
- org.gradle.parallel
- org.gradle.workers.max
- org.gradle.caching
- org.gradle.configureondemand
- kotlin.incremental
- kotlin.compiler.execution.strategy
- kotlin.parallel.tasks.in.project
- kapt.use.worker.api
- kapt.incremental.apt
- プロジェクト固有のやつ
    - android.databinding.incremental
    - android.lifecycleProcessor.incremental=false
    - firebasePerformanceInstrumentationEnabled


## 参考プロジェクト

### insetterライブラリ

https://github.com/chrisbanes/insetter/tree/master/.github

```yml
JAVA_TOOL_OPTIONS: -Xmx5120m
GRADLE_OPTS: -Dorg.gradle.daemon=false -Dorg.gradle.workers.max=2 -Dkotlin.incremental=false -Dkotlin.compiler.execution.strategy=in-process
```

### Catchupアプリ

https://github.com/ZacSweers/CatchUp/tree/master/.github

```groovy
org.gradle.daemon=false
org.gradle.parallel=true
org.gradle.jvmargs=-Xmx5120m
org.gradle.workers.max=2

kotlin.incremental=false
kotlin.compiler.execution.strategy=in-process
```
