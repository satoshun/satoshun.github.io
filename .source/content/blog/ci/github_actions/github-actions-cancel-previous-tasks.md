+++
date = "Sat Apr  3 03:00:25 UTC 2021"
title = "GitHub Actions: 前のGitHub Actionの実行をキャンセルする"
tags = ["ci", "githubactions"]
blogimport = true
type = "post"
draft = false
+++

Pushを複数回した場合などに、1つ前のGitHub Actionsの実行を止める方法の紹介です。

[styfle/cancel-workflow-action](https://github.com/styfle/cancel-workflow-action)を使うことで実現できます。
READMEにも書いてあるのですが、以下のstepを追加するだけです。

```yml
steps:
- name: Cancel Previous Runs
  uses: styfle/cancel-workflow-action@0.8.0
  with:
    access_token: ${{ github.token }}
```

GitHub Actionsは従量課金なので、上記のキャンセルをやっておくと、多少の節約になると思います。
