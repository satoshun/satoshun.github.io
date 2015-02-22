+++
date = "2015-02-22T02:08:23Z"
title = "AnsibleでGhostアプリをデプロイ"
tags = ["ansible", "system"]
blogimport = true
type = "post"
+++


VPSに[Ghost](http://ghost.org)を, Ansibleでデプロイしている話.

サーバー側で使っているソフトウェアは, supervisor, nodeです.


## GitHubでソース管理

CSS, HTMLを少し弄りたいので, Ghostをforkし, それを編集してデプロイしています.(https://github.com/satoshun/ghost)

このリポジトリを, デプロイしていきます.


## デプロイ時の手順

サーバで以下のタスクを行います.

- Gitリポジトリを最新の状態にする(git pull origin master)
- npm moduleの更新(npm install)
- gruntの実行(grunt init && grunt prod)
- ghostプロセスの再起動(supervisorctl restart ghost)

Playbookは以下になります.

```yml
---
- hosts: all
  user: "{{ user }}"
  tasks:
    - include: update_source.yml
    - include: update_package.yml
    - include: restart.yml
```

各タスクについて説明していきます.


### Git pullする

Gitモジュールがあるので, それを使います.

```yml
- name: Update Git repository
  git: repo=<git url> dest=<path to project>
```

### npm moduleの更新

npmモジュールがあるので, それを使います.

```yml
- name: Update npm module
  npm: path=<path to project>
```

### gruntの実行

commandモジュールを使います.

```yml
- name: Run grunt init
  command: "grunt init chdir=<path to project>"

- name: Run grunt prod
  command: "grunt prod chdir=<path to project>"
```


### Ghost再起動

supervisorで管理しているので, supervisorctlモジュールを使い再起動を行います.

```yml
- name: Restart Ghost
  sudo: yes
  supervisorctl: name=ghost state=restarted
```
