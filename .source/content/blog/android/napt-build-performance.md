+++
date = "Sun Dec  4 09:40:27 UTC 2022"
title = "naptを使って、ビルドを高速化する"
tags = ["android", "build", "napt"]
blogimport = true
type = "post"
draft = true
+++

kaptの代替ライブラリの[napt](https://github.com/sergei-lapin/napt)がDagger hiltに対応したので、弊アプリでどれくらいビルド速度が上がるかを簡単に検証しました。
naptについて簡単に説明すると、ビルドを高速化するけど、機能が制限されているkaptっていう感じです。

実際に、`assembleDebug --rerun-tasks`コマンドを10回程度実行してそれぞれ比較してみました。

```text
// Before
2m 6s
2m 46s
2m 4s
2m 47s
2m 20s
2m 47s
2m 3s
2m 46s
2m 5s
2m 14s
```

```text
// After
1m 48s
1m 46s
1m 48s
1m 49s
1m 46s
2m 2s
2m 29s
1m 51s
1m 46s
1m 48s
```

ややビルドが早くなりました。kapt -> naptへの置き換えもほぼ苦なく出来ました。
それなりに大規模なプロジェクトの場合には、導入を検討してみるのも良さそうです。

## 補足

- 弊アプリの場合には一部napt置き換えが出来なかったので、kaptを使っている箇所があります
- まだ、本番にリリースなどしていないので、実際には何か問題がある可能性があります
- kaptに比べると、一部機能落ちしているのでそこには注意が必要です
