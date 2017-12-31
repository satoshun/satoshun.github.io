+++
date = "2015-02-22T02:40:23Z"
title = "tips: 気軽にローカルにサーバを立てる"
tags = ["tips", "python"]
blogimport = true
type = "post"
+++

サクッとローカルサーバを立てるためのTipsを紹介します.

下記コマンドは, Pythonが入っていれば, 問題なく動きます.

```shell
$ python -m CGIHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

僕は, 上記コマンドをaliasに登録しています.

```shell
alias ser='python -m CGIHTTPServer'
```

これで, serでサーバが立つようになりました.

ちなみに, Portを指定することも出来ます.

```shell
$ ser
Serving HTTP on 0.0.0.0 port 8000 ...

$ ser 8001
Serving HTTP on 0.0.0.0 port 8001 ...
```

簡単にローカルサーバを立てることが出来るようになりました.
