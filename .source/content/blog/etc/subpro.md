+++
date = "2014-05-08T00:00:00Z"
title = "subproコマンド: Sublime Projectのプロジェクト管理"
tags = ["sublimetext"]
blogimport = true
type = "post"
+++

## subproコマンド

Sublime Textのプロジェクトをterminalで管理するためのコマンドを作りました．

結構便利だと思うので，使ってくれたら嬉しいです． https://github.com/satoshun/subpro


### インストール

homebrewでインストール出来ます．

```shell
$ brew tap satoshun/commands
$ brew install subpro
$ brew install subpro-completion
```

.bashrcに下のコードを追加してください．

```shell
if [ -f `brew --prefix`/etc/bash_completion.d/subpro_completion ]; then
    source `brew --prefix`/etc/bash_completion.d/subpro_completion
fi
```

### 使い方

#### プロジェクト作成

```shell
# 基本構文
subpro create [group] [プロジェクトのディレクトリ]

# 例
$ ls
martini
$ subpro create golang martini
## create martini project into golang directory and open project.
```

#### 作成したプロジェクトを開く

```shell
# 基本構文
subpro [プロジェクト名]

# 例
$ subpro martini
## open martini project with sublime text
```

#### プロジェクト一覧

```shell
# 例
$ subpro [tab]
Display all 131 possibilities? (y or n)
Amalgam                                 beaker                                  ghost                                   roboguice
Android-Universal-Image-Loader          bitcoin                                 gist_css                                routes
```

#### プロジェクトを削除

```shell
# 基本構文
subpro delete [プロジェクト名]

# 例
$ subpro delete martini
## delete martini project
```
