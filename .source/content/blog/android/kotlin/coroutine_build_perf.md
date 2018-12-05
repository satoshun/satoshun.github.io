+++
date = "2018-12-05"
title = "Kotlin Coroutineを導入したらどれだけビルドタイムが増えそうか検証した"
tags = ["android", "kotlin", "coroutine"]
blogimport = true
type = "post"
draft = false
+++

**注意**: 計測方法は実環境に全く即していないので意味がない可能性が高いです。

### 環境

- AGP3.4.0-alpha06
- Kotlin 1.3.10
- Kotlinx.coroutine 1.0.1
- Gradle 5.0

### 比較対象

- kotlinx.coroutineを使ったサンプル
  - クラス数 300
  - 各クラスは7つのメソッドを持ち、その中でcoroutine builderやsuspend関数をコールしている
    - 300 * 7の2100箇所がCoroutine関連のコードになります
- kotlinx.coroutineを使わないサンプル
  - クラス数 300
  - 各クラスは7つのメソッドを持ち、その中で適当なメソッド（`Handler().post {}`）をコールしている
- [サンプルコード](https://github.com/satoshun-android-example/CoroutineBuildPerfExample)

### 計測コマンド

Build Scanを使って計測します。その際。build-cacheはoffにします。

```command
./gradlew clean
./gradlew build --no-build-cache --scan
```

両サンプルのクラス数、メソッド数を合わせただけなので、全く正当な比較でないことを留意ください。
また試行回数は10回程度で、最終結果のみを以下に掲載します。

## kotlinx.coroutineを使う

```
Time spent executing tasks 1m 16.034s
    All tasks	207	2m 42.833s
    Tasks avoided	12 (09.7%)	0.062s
    From cache	0 (00.0%)	0.000s
    Up-to-date	12 (09.7%)	0.062s
    Tasks executed	112 (90.3%)	2m 42.739s
    Cacheable	0 (00.0%)	0.000s
    Not cacheable	112 (90.3%)	2m 42.739s
    Lifecycle	45	0.023s
    No source	38	0.009s
    Skipped	0	0.000s
```

最終実行時間は`1m 16.034s`になります。

## kotlinx.coroutineを使わない

次に使わない例です。

```
Time spent executing tasks 1m 1.520s
    All tasks	207	1m 57.416s
    Tasks avoided	12 (09.7%)	0.015s
    From cache	0 (00.0%)	0.000s
    Up-to-date	12 (09.7%)	0.015s
    Tasks executed	112 (90.3%)	1m 57.356s
    Cacheable	0 (00.0%)	0.000s
    Not cacheable	112 (90.3%)	1m 57.356s
    Lifecycle	45	0.021s
    No source	38	0.024s
    Skipped	0	0.000s
```

最終実行時間は`1m 1.520s`になります。

## まとめ

- 上記のような結果になりました。しかし、繰り返しになりますが、この比較は実環境に即して無い、そもそもサンプルコードが同等でないため正当じゃないです
  - ただ、Coroutineを入れてある程度の規模まで行くと、フルビルド時のビルド時間の増加は顕著になるかもしれません
  - iMac Proを買ってもらいましょう💻s

もっとこういうふうに比較してほしいであったり、間違っている部分があればご指摘いただければ幸いです😃

[サンプルコードはここにあります](https://github.com/satoshun-android-example/CoroutineBuildPerfExample)😃
