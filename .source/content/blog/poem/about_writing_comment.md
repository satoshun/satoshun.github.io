+++
date = "2018-04-05T00:10:00Z"
title = "ActivityやFragmentにコメントを書くことについて"
tags = ["poem", "android"]
blogimport = true
type = "post"
+++

ポエムです。最近(2018年4月現在)思っていることなので、今後考えが変わる可能性は大いにあります。

例えば以下のようなActivityがあるとします。

```kotlin
class HogeActivity : Activity() {
    ...
    ...

    private fun isHeavyUser(loginCount: Int, firstAccess: Boolean) : Boolean() {
        return firstAccess || (loginCount >= 10 && loginCount <= 100)
    }
}
```

これを見た時、「なんで `loginCount <= 100`にしているの? 100回以上ログインしてるんだからヘビーユーザやん?」 って思う可能性があるのでコメントを追加したくなります。

```kotlin
class HogeActivity : Activity() {
    private fun isHeavyUser(loginCount: Int, firstAccess: Boolean) : Boolean() {
         // 100回以上ログインした場合は超ヘビーユーザなので100以上はheavy userではない
        return firstAccess || (loginCount >= 10 && loginCount <= 100)
    }
}
```

「超ヘビーユーザっていうのがいて、それにヘビーユーザは含まれていないのね。」というのがコメントから理解できます。

ただ、自分の考えでは上記のコードは根本的に間違っていると思っていて、そもそもActivityでコメントが必要なほど複雑なことをしているのが問題だと思います。
なんでActivityで複雑なことをしてはいけないかというと、ActivtyはContextにアクセスできたりと、なんでも出来るからです。なんでも出来る層でいろいろやってしまうと、
いわゆるfat activity問題が起こってしまいます。

なので上記のコードだと、例えばUserモデル(データ)クラスのようなものを作ってそこにロジックを書いて、必要に応じてコメントを付加するのが良いと思います。

```kotlin
data class User(private val loginCount: Int, private val firstAccess: Boolean) {
    private fun isHeavyUser() : Boolean() {
         // 100回以上ログインした場合は超heavy userなので100以上はheavy userではない
        return firstAccess || (loginCount >= 10 && loginCount <= 100)
    }

    private fun hyperHeavyUser(): Boolean() { /** */ }
}
```

人間に理解し難いビジネスロジックのコメントは、モデルクラス(世の中的にはドメインモデルとか言われているのカナ?)に書くのが良いと思います。


## まとめ

Activityにコメントを書きたくなったら、クラスを分割したほうが良い


## 補足

- とはいえActivityにコメントを書く正しいケースもあると思うので、そこらへんは柔軟にオナシャス
- コメントはコードと違い、コンピュータがコンパイルして正当性を確かめてくれるわけでないので、正しく運用するのがコードより難しいと思う
