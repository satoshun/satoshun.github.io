+++
date = "2018-08-08T00:00:00Z"
title = "Android: Dagger 2.17のfastInitを試してみた"
tags = ["android", "dagger"]
blogimport = true
type = "post"
+++

Dagge 2.17でfastInitオプションが追加されました。
https://google.github.io/dagger/compiler-options

これは、startup timeを改善するための機能です。どれくらい差があるかを担当アプリで実際に調べてみました。

確認に使用した端末はAndroid8系のGalaxyと、Android7系のXperiaの計2台になります。

## 確認に使用したシェルスクリプト

adb shellコマンドから起動時間を調べるコマンドです。計11回startup timeを確認する事ができます。

```shell
for i in {0..10}
do
    adb shell am start -S -W jp.hoge/.ui.main.MainActivity -c android.intent.category.LAUNCHER -a android.intent.action.MAIN >> hoge.txt
    sleep 10
done
```

下記を参考にしました。

- https://developer.android.com/topic/performance/vitals/launch-time

## fastInit有効の場合

下記をbuild.gradleに追加します。

```groovy
kapt {
    javacOptions {
        option("-Adagger.fastInit=enabled")
    }
}
```

結果:

```
平均: 1609ms
```

## fastInit無効の場合

結果:

```
平均: 1607ms
```

## まとめ

ほぼ変わらない数字が出てきてしまいました。悲しい。
Dagger生成コードを見る限りだと、最初のComponentのcreateのタイミングでComponentが持っているフィールドの初期化が行われていなかったので、早くなりそうだなと思ったんですが、実際にはほぼ変わりませんでした。

芳しくない結果になった推測として

- 担当アプリのDaggerの書き方が正しくないからこの結果になった?
  - 要調査、しかし一般的なAndroid-Daggerを使った書き方をしているので正しいはず
- 確認に使用したコマンドが良くないのかも?

なにか分かったら追記します、もしくは間違っている点があればご指摘いただければ幸いです😊
