+++
date = "2015-02-22T01:08:23Z"
title = "AnsibleでGoアプリをデプロイ"
tags = ["ansible", "golang"]
blogimport = true
type = "post"
+++


ローカルでバイナリを作成し, デプロイするような, Ansible Playbookを書きました. Supervisorでプロセスの管理を行っています.

下の手順でデプロイを行っています.

- Macでコンパイルして, Linux-amd64用のバイナリ生成(**[注意1](#caution1)**)
- バイナリをサーバにデプロイ(コピー)
- アプリのリスタート(supervisorで再起動)

実際のPlaybookは以下のようになります.

deploy.yml

```
---
- hosts: local
  connection: local
  tasks:
    - include: build.yml


- hosts: app
  user: "{{ user }}"
  tasks:
    - include: update_binary.yml
    - include: restart.yml
```

各タスクについて, 説明してきます.


## Linux-amd64用のバイナリ生成

Goはデフォルトで, クロスコンパイル出来る機能を持っているので, その機能を使います.

やり方はいろいろあると思うのですが, Makefileを作成して, それをAnsibleから叩くようにして実装しました. Makefileを作る必要ないと思います.

Makefile

```make
get:
  go get -v

build-amd64: get
  GOOS=linux GOARCH=amd64 go build .
```

build.yml

```yml
---
- name: build src
  command: make build-amd64 chdir={{ local_home }}
```

local_homeは変数で, Makefileがあるディレクトリを定義しています.


## バイナリをデプロイ

ファイルをサーバーにコピーする時は, copyモジュールを使います.

update_binary.yml

```yml
---
- name: update binary
  copy:
    src={{ local_home }}/{{ project_name }}
    dest={{ go_bin }}/{{ project_name }}
```

srcにバイナリのpath, destにバイナリをデプロイするpathを指定します.


## アプリのリスタート

supervisorctlモジュールを使います.

restart.yml

```yml
---
- name: restart binary
  sudo: yes
  supervisorctl: name={{ project_name }} state=restarted
```

ちなみに, supervisor confファイルは以下のようにしています.

```ini
[program:{{ project_name }}]
user={{ user }}
command={{ go_bin }}/{{ project_name }}
autostart=true
autorestart=true
stdout_logfile = /var/log/supervisor/%(program_name)s.log
stdout_logfile_maxbytes = 10MB
stdout_logfile_backups = 10
stderr_logfile = /var/log/supervisor/%(program_name)s-error.log
stopsignal=INT
```

commandにGoのバイナリを指定して, あとはいつものおまじないです.


## まとめ

デプロイスクリプトを作っておくことは, 非常によいことだと思います. Ansibleはshell scriptくらいの気軽さで書け, エラーハンドリングなどをよしなにやってくれるのでさくっと書けてお勧めです.


### <a name="caution1"></a>注意1

HomebrewでGoをクロスプラットフォームにビルド出来るようにするには,
--cross-compile-allオプションを付けてあげます.

```shell
$ brew install go --cross-compile-all
```
