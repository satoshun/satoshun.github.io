+++
date = "2015-03-14T00:00:00Z"
title = "本: HTTPの教科書"
tags = ["book", "web", "api"]
blogimport = true
type = "post"
draft = true
+++

この記事には, 僕の主観, 解釈が入っています. 本のまとめというよりかは, 感想的なものです.


## TCP/IP

4階層に分かれている. 個々の変更が全体に影響しないように
- リンク層
- ネットワーク層: IP
- トランスポート層: TCP, UDP
- アプリケーション層: HTTP, FTP, SSH

トランスポート層で, パケットに分割, 相手に届いたかどうかを確認
ネットワーク層で, パケットを相手ホストに届ける.


## HTTP

持続的接続
- TCPコネクションはオーバーヘッドが高い. request毎にコネクションをcloseしないようにする
  - サーバの負荷軽減, レスポンスタイムの削減
- pipeライン化により, imageのrequestなどを同時に行える. responseの順番は保証されない

cookie
- HTTPはステートレスなプロトコル
  - cookieをブラウザに保存してもらい, それをサーバに送ってもらうようにする

multipart/form-data
- 1つのrequestに画像, textfileなど複数のデータが送れる. MIMEと似たような拡張

コンテンツ・ネゴシエーション
- 同じURIだが, requestに含まれるフィールドを見て, 出すコンテンツを出し分ける

HTTP status code
- 2XX: success: ok, ...
- 3XX: redirection
- 4XX: client error: bad request, ...
- 5XX: server error: internal server error, ...


## HTTP header field

Cache-control
- private, public: 他の人にcacheを見せるか
- max-age: 秒指定
- ...

Connection
- それ以上転送しないヘッダーフィールドを指定: Connection: upgradeなら, upgradeフィールドを転送しない
- HTTP1.1以前で, Connection: Keep-Aliveを指定すると, Connectionを使い回せる

Upgrade
- TLS/1.0など, 他のプロトコルを使いたいときに指定する. OKなら, 101 Switching Protocolsを返す

Accept
- text/htmlなど, 欲しい拡張子を指定する

Accept-Charset
- 文字コードを指定

Accept-Encoding
- コンテンツコーディングを指定
