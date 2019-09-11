+++
date = "Wed Sep 11 02:14:03 UTC 2019"
title = "ISUCON9 予選ログ"
tags = ["isucon", "server"]
blogimport = true
type = "post"
draft = false
thumbnail = "https://satoshun.github.io/blog/contest/isucon.png"
+++

ISUCON9にチームSsstohで参加してきました。最終スコアは11,860で、無事に本線に出場出来ることになりました:D

僕がやったことをつらつらと反省とともに、ログに残して置こうと想います。

## 僕がやったこと

- 朝
    - 寝坊しました

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ISUCON起床失敗</p>&mdash; Sato Shun🧁 (@stsn_jp) <a href="https://twitter.com/stsn_jp/status/1170138234036207616?ref_src=twsrc%5Etfw">September 7, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 12:00くらいまで
    - アプリケーションコード読んだり、alpの結果見たり、スロークエリログに対してIndexを適当に貼ってみたりしてた
        - スコアほぼ変わらず。それはそう
- 12:00から
    - `/users/transactions.json`が遅そうだったので、そこを潰す
        - USERの取得などがN+1クエリだったので、それを `IN`句に置き換え
        - 道中、外部APIを叩いてるところがボトルネックっぽいことに気づく
    - ここらへんの修正で、確かスコアが5000~6000くらい（他の人の修正も入っているので、これだけではないです）
- 16:30くらい
    - `/new_items/:root_category_id.json`、`/new_items.json`のN+1クエリログを倒す
        - ここらへんで確か8000~9000くらい
- 17:30くらい
    - 他の人のPRみたり、静的ファイルをNginxから返すようにしてた
        - ここらへんで10000~くらい
- 最後らへん
    - ログを出力しないようにしたり修正
        - ここらへんで最終スコアの11,860くらい

こんな感じ

## 反省点

- 3台構成に出来なかった
    - 練習不足で時間が足りなかったんやなって
- キャンペーンモードを忘れていた
    - READMEはちゃんと読もうな
- ログインにボトルネックがあることに気づかなかった
    - pprofしていこうな
- SQLの書き方が記憶から失われていた
    - 頑張っていこうな
- 寝坊しない
    - それは無理かもしれない

## まとめ

- 本線頑張るぞ〜😃
