+++
date = "Sat Mar  2 13:21:05 UTC 2019"
title = "RxAndroidにPull Requestを出した話"
tags = ["android", "rxandroid", "rxjava", "github", "pr"]
blogimport = true
type = "post"
draft = false
+++

RxAndroidにIssueを立てて、PRを出した話です。広く使われているOSSプロジェクトに対して、Issueを立てて、テストもセットでPRを出したことがなかったので、それの記念記事になります。

## Issueの内容

実際のIssueは [HandlerScheduler.scheduleDirect supports async option?](https://github.com/ReactiveX/RxAndroid/issues/441) になります。

Issueの概要は、RxAndroidは2.1.0でasync messageに対応しました。しかし、これは`Worker.schedule`のスケジュールからのみのサポートでした。RxJavaでは`Scheduler.scheduleDirect`でもスケジュールされるので、こちらも対応したほうが良いのでは？と思いIssueを立てました。

## そもそもこの問題に気づいたきっかけ

RxAndroidが2.1.0でasync messageに対応した時に、サンプルで効果を測定したところ、いくつかのオペレータではパフォーマンスの向上が見られないことに気づきました。このときは、サンプルが悪いのか、それとも環境がおかしいのか、またまたこれが意図した挙動なのかが分かりませんでした。とりあえず、自分のタスク管理をしているtodoistに「良く分からないけどパフォーマンスが向上しないパターンがある」みたいなタスクを作って、あとで調べることにしました。

## 調べ方

クラッシュするわけでもないので、パフォーマンスが向上するパターンと、向上しないパターンでスケジュールのされかたに違いがないかをデバッガーを使い、地道にコードを追いかけました。

結果、`Observable.observeOn`では`Worker.schedule`メソッドでタスクのスケジューリングをし、`Maybe.observeOn`では`Scheduler.scheduleDirect`メソッドでスケジューリングする違いがあることが分かりました。後は、それぞれのパスでのasync messageの挙動の違いを特定し、修正するだけです。

## その他・感想

### JakeさんとZacさんにレビューをしてもらった

二人のコードは良く読んでいて、尊敬しているAndroidエンジニアなので、その2人にレビューをしてもらえたのは嬉しかったです😃
