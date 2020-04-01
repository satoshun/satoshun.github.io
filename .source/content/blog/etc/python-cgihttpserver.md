+++
date = "2016-02-16T00:00:00Z"
title = "お手軽にローカルにサーバを立てる"
tags = ["python", "tips"]
blogimport = true
type = "post"
draft = false
+++

サクッとローカルにサーバを立てるためのコマンドを紹介します.
Python2系が入っていれば, 下記コマンドでローカルにサーバを立てることが出来ます.

```shell
$ python -m CGIHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

ブラウザで, `http://localhost:8000`にアクセス出来るようになりました.

ちょっとコマンドが長いので, 僕は, 上記コマンドをaliasに登録しています.

```shell
alias ser='python -m CGIHTTPServer'
```

これで, `ser`でサーバが立つようになりました.

ちなみに, デフォルトではポート8000でサーバが立つのですが, 指定することも出来ます.

```shell
$ ser 10000
Serving HTTP on 0.0.0.0 port 10000 ...
```

これで, いつでも簡単に`ser`でローカルにサーバを立てることが出来るようになりました.
