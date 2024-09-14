+++
date = "Sat Sep 14 14:41:37 UTC 2024"
title = "Android at Scale(with Circuit)の感想"
tags = ["android", "architecture"]
blogimport = true
type = "post"
draft = false
+++

John Buhananさんの、[Android at Scale with Circuit](https://www.johnbuhanan.dev/android-at-scale-with-circuit/) がとても良い記事だったので雑記。 

上記の記事では、大規模なプロジェクトにおいて、どのようにマルチモジュールを構成するかについての解説をしています。

その中で、個人的に、

- 各featureモジュール単位でアプリが実行できるようにする
- feature(library)モジュールは、public/internal/appで構築されて、各モジュールはpublicのみに依存するようにする

の部分が刺さったので、それについて雑感を述べていきます。また、本記事では、技術詳細は省いているので、より詳細は元記事を読んでもらえると嬉しいです。

## 各featureモジュール単位でアプリが実行できるようにする

一般的なプロジェクトでは、全機能が入ったアプリを作ることしか出来ないと思うんですが、ここでは1部の機能だけを持ったアプリを作れるようにすることを提案しています。

割と前から、この考え自体はandroid界隈で囁かれていたんですが、個人的にはコード複雑になりそうだと感じたので、導入するのはしんどそうだなと思っていました。しかし、記事内ではCircuitライブラリ、Dagger Hilt、モジュール構成などを工夫することで、シンプルな構成での1部機能アプリの作成を可能にしています。

詳細は [元記事](https://www.johnbuhanan.dev/android-at-scale-with-circuit/) 内に書かれているんですが、DI周りを少し説明すると、Daggerのマルチバンディングを使うことで実現可能です。例えば、Account機能の宣言イメージは次のようになります。

```kotlin
// CircuitライブラリのDIコード
@Module
@InstallIn(SingletonComponent::class)
abstract class AccountFactoryModule {
  @Binds
  @IntoSet
  abstract fun bindAccountFactory(accountFactory: AccountFactory): Ui.Factory
}
```

Account機能を使いたいときは、上記の宣言を依存に含めてあげれば、Account機能が注入されます。逆に、この宣言を依存に含めなければ、Account機能を外すことができるため、「Accountモジュールの依存を外せる = Accountモジュールのビルドを省略」することが出来ます。

例えば、10個の機能があるアプリだったら、9個の機能モジュールの依存を外せて、さらにその9個が依存しているモジュールの依存も省略できるので、かなりのビルド短縮が見込めます。

さらに、dropbox/focusライブラリを使うことで、読み込むモジュール数を減らすことが出来ます。これは不必要なモジュールをアンロードしてくれるライブラリで、イメージとしてはsetting.gradleのincludeを自動生成してくれるようなライブラリになっています。これにより、Gradle Configurationの改善や、IDEのパフォーマンス改善が期待できます。

まとめると、モジュール構成などを工夫して機能ごとにアプリを作成出来るようにすることで、ビルド速度が改善できる。dropbox/focusを使うことで、IDEのパフォーマンスなども改善ができる。ツールやライブラリも整ってきたので、コードも複雑にならない。っていう感じです。

## 各featureモジュールは、public/internal/appで構築されて、各モジュールはpublicのみに依存する

じゃあ実際にどのようにモジュールを構成すれば良いのかっていう話なんですが、

- public: CircuitのScreenクラスのみが入っている (navigationコードだけ)
- internal: CircuitのPresenter、UIなど、実画面の実装と、publicで定義したScreenと結びつける用のDIが入っている
- app: 起動に必要なDIが定義されている。これを含めることで、この機能をエントリーポイントとして実行することができる

っていう感じで構成されています。画面間を遷移するためにはnavigationをいい感じに実装する必要があるんですが、Circuitでは、Screen (Intentみたいなイメージ) を使ってnavigationします。各画面から遷移するときは、Screenの情報があれば良いので、Screenのみが含まれたpublicモジュールを機能ごとに作って、各featureモジュールがpublicのみに依存するようにすることで、最低限のモジュール間依存でnavigationを構築することができます。

そして、実行時にテストしたい機能のみに依存することで、最小限の依存で機能単位で実行することが可能になります。例えば、Account機能のテストをしたかったら、feature:account:app モジュールを使うような感じです。

まとめると、各機能モジュール間の依存を最小限にすることで、機能ごとの実行を容易に出来て、そのためにpublic/internal/appっていう粒度でモジュールを分割するといいよっていう感じです。

また、記事内ではCircuitを使っているんですが、Circuitを使わずに、何かしらの抽象的なnavigationシステムを構築すれば、似たようなことが実現できると思います。

## 最後に

色々と書いたのですが、ざっくりとした話しかしていないので、細かい部分については、ぜひ元記事の方を読んで頂けると幸いです。

- [https://www.johnbuhanan.dev/android-at-scale-with-circuit/](https://www.johnbuhanan.dev/android-at-scale-with-circuit/)
- [https://github.com/JamesBuhanan/Songify](https://github.com/JamesBuhanan/Songify)
