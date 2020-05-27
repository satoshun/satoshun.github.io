+++
date = "Wed May 27 03:52:50 UTC 2020"
title = "avocadoとGitHub Actionsを使って、Vector Drawableを最適化する"
tags = ["android", "ci", "githubactions"]
blogimport = true
type = "post"
draft = false
+++

Vector Drawableを最適化する [avocado](https://github.com/alexjlockwood/avocado)と、GitHub Actions組み合わせた話です。

## 🥑avocado🥑?

avocadoを使うことで、Vector Drawableを最適化することができます。使い方は簡単です。

```shell
avocado [file name]
```

Vector Drawableによっては、結構最適化されます。

## GitHub Actionsに組み込む

1週間単位でavocadoを実行して、PRを作るGitHub Actionsになります。

```yaml
name: avocado

on:
  schedule:
  - cron:  '0 0 * * 0'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup node
      uses: actions/setup-node@v1
      with:
        node-version: '12'

    - name: Setup avocado
      run: npm install -g avocado

    - name: Run avocado
      run: find . -type f -name *.xml | xargs grep "</vector>" -rl | avocado

    - name: PR
      id: cpr
      uses: peter-evans/create-pull-request@v2
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        commit-message: execute avocado
        title: 'title'
        branch: your_branch
```

Vector Drawableなファイル一覧を `find . -type f -name *.xml | xargs grep "</vector>" -rl | avocado` で取得して（もっと効率の良い書き方がありそう)、差分があれば、[peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request) でPRを作ります。
また、週に1度実行するようにScheduleを指定しています。Scheduleの記法の確認には [crontab guru](https://crontab.guru/#0_0_*_*_0) が便利です。
