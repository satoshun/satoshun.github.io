+++
date = "Sun Aug 29 13:18:37 UTC 2021"
title = "GitHub Actions: Composite Run Stepsの簡単な使い方"
tags = ["ci", "githubactions", "android"]
blogimport = true
type = "post"
draft = true
+++

Composite Run Stepsを使うと、GitHub Actionsでactionを再利用する事ができます。

この記事では、同一リポジトリ内で使い回すパターンの、Composite Run Stepsについて紹介します。

最終的には、こんな感じになります

- [composite run stepの定義側](https://github.com/satoshun-android-example/Template/blob/main/.github/actions/setup-java/action.yml)
- [composite run stepを使う側](https://github.com/satoshun-android-example/Template/blob/main/.github/workflows/android.yml#L17)

## ローカルcomposite run stepsの定義

今回は、Java11のセットアップをする、setup-java ステップを作ってみたいと思います。

最初に、次のパスにaction.ymlを作ります。

`.github/actions/setup-java/action.yml`

action.ymlは次のようになります。

```yml
name: 'Java Setup'
description: 'Installs Java'

runs:
  using: 'composite'
  steps:
    - name: Install Java 11
      uses: actions/setup-java@v2
      with:
        distribution: 'adopt'
        java-version: '11'
```

今回は1つのstepしか使っていませんが、stepsには複数stepを定義することが出来るので、処理を1箇所にまとめることが可能です。

## setup-javaをworkflowから使う

次に、定義したsetup-javaを使ってみます。

```yml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup Java
      uses: ./.github/actions/setup-java

    ...
```

usesでパスを指定するだけです。これで、stepを再利用することが出来ます。

## まとめ

Composite Run Stepsを使うことで、複数のstepをまとめ、再利用が可能になります。同じようなstepを記述しているworkflowが多い場合は導入すると幸せになると思います。

## 参考

- [GitHub Actions: Composite Run Steps](https://github.blog/changelog/2020-08-07-github-actions-composite-run-steps/)
