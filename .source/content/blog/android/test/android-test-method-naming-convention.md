+++
date = "2016-02-12T00:00:00Z"
title = "Android: テストメソッドの命名規則について"
tags = ["android", "test"]
blogimport = true
type = "post"
draft = false
+++

ソフトウェアの世界ではAndroidに限らず, テストを書くというのは非常に重要です.

ユニットテストの場合は, メソッドレベルでリグレッションが起きていないことを,
機能/受け入れテストの場合は, ユーザの行動レベルでリグレッションが起きていないことを保証するために必要になります.

今回の記事では, Androidにおけるユニットテスト(各テストメソッド)の命名規則の話をします. しかし, 他の言語/プラットフォームの場合でも同様の考え方を出来ると思います.


## 用語定義

アプリコード: アプリケーション本体のコード


## なぜテストケースの命名規則を気にする必要があるか?

Androidアプリを作っていて, アプリコードはリファクタリングや, 新機能の追加などで頻繁に目を通しますが,
テストコードは1度書いたら, そのテストが失敗するまで放置するパターンが多いなと感じています(何にしてもアプリコードほどは目を通さない)

数週間前に書いたテストコードなんて忘れてしまうので, テストコードにはいろいろな情報をコメントなどで残しておく必要があります.


## まず, テストケースの構成を考える

本題に入る前に, テストケースがどのような要素で構成されているかを説明をします.

一般的なテストケースは次の4つの要素に分割できます.

1. 事前準備(setup)
2. 実行(exercise)
3. 宣言(assertion)
4. 後処理(teardown)

これだと分かりにくいので, コードで説明すると,

```java
public class HogeTest {
  List<String> lists;

  @Before
  public void setup() {
    // 事前準備(setup)
    lists = new ArrayList<>();
    lists.add("1");
    lists.add("123");
  }

  @After
  public void teardown() {
    // 後処理(teardown)
  }

  @Test
  public void testGet() {
    // 実行(exercise)
    String actual = lists.get(0);

    // 宣言(assertion)
    assertThat(actual).isEqualTo("1");
  }
}
```

という感じになります. JUnit4ではこのようにテストを構築します.


## テストケース`testGet`という名前

前述の例では, テストケースの名前を`testGet`にしています. 果たして, これで意味が伝わるでしょうか?
`testGet`から分かることは, 「getメソッドをテストしている」ことだけであって, getメソッドの**何を**テストしているのかが伝わってきません.
なので, その情報をメソッド名, もしくはコメントに含める必要があります.

では, どのような情報を含めれば良いのか? それを次の例で説明していきます.


## やや複雑な例

`Request`クラスの内容に応じて, HTTPでデータを取ってくる`HTTPConnection`クラスを考えます.


```java
public class HTTPConnectionTest {
  HTTPConnection httpConnection;

  @Before
  public void setup() {
    // 事前準備(setup)
    httpConnection = new HTTPConnection();
  }

  @After
  public void teardown() {
    // 後処理(teardown)
  }

  @Test
  public void execute() {
    // 事前準備(setup)
    Request validRequest = new Request("http://valid-url.com")
    // 実行(exercise)
    Response response = httpConnection.execute(validRequest);
    // 宣言(assertion)
    assertThat(response.isOk()).isTrue();

    // 事前準備(setup)
    Request inValidRequest = new Request("http://invalid-url.com")
    // 実行(exercise)
    response = httpConnection.execute(inValidRequest);
    // 宣言(assertion)
    assertThat(response.isOk()).isFalse();
  }
}
```

上記テストで問題なのは, `execute`で正常, 異常の2つのことをテストしている所です.
一般的に, 1つのテストケースでは, 1つのことをテストすべきなので, 修正します.


```java
  @Test
  public void execute_ok() {
    // 事前準備(setup)
    Request validRequest = new Request("http://valid-url.com")
    // 実行(exercise)
    Response response = httpConnection.execute(validRequest);
    // 宣言(assertion)
    assertThat(response.isOk()).isTrue();
  }

  @Test
  public void execute_ng() {
    // 事前準備(setup)
    Request inValidRequest = new Request("http://invalid-url.com")
    // 実行(exercise)
    response = httpConnection.execute(inValidRequest);
    // 宣言(assertion)
    assertThat(response.isOk()).isFalse();
  }
```

これで, 1つのテストケースで1つのことをテストするようになり, 大分綺麗になりました.


## ここでテストのメソッド名を考える

このトピックがこの記事で一番伝えたい事です.

前述の例では, `execute_ok`, `execute_ng`の2つのテストメソッドを作成しました.
この2つのテストメソッドは, executeメソッドの, 正常, 異常系をテストしていることは伝わります.
しかし, **どういう条件下でこの振る舞いをするか**の情報が抜けています. よって, その情報を付与する必要があります.


```java
  @Test
  public void execute_ok_200URL() {
    // 事前準備(setup)
    Request validRequest = new Request("http://valid-url.com")
    // 実行(exercise)
    Response response = httpConnection.execute(validRequest);
    // 宣言(assertion)
    assertThat(response.isOk()).isTrue();
  }

  @Test
  public void execute_ng_404URL() {
    // 事前準備(setup)
    Request notFoundRequest = new Request("http://invalid-url.com")
    // 実行(exercise)
    response = httpConnection.execute(notFoundRequest);
    // 宣言(assertion)
    assertThat(response.isOk()).isFalse();
  }
```

`execute_ok_200URL`, `execute_ng_404URL`とし, 前提条件をテストメソッド名に含めました.
これで, 「200のURLならOK」, 「404のURLならNG」を返すことが, メソッド名から推測することが出来ます.

これを一般化すると. `テスト対象メソッド_期待される振る舞い_前提条件`という命名規則になります. これを, ルール化することで, テストコードの可読性が上がり, 全体的に統一感のあるテストが作成できると思います.


## まとめ

今回はテストメソッドの命名規則について紹介しました. 個人的に開発をして思うのは, アプリコードはともかくとして, テストコードを読むのは結構苦痛だなということです.
特に, 読みにくいテストコード(数ヶ月前の自分が書いたテストコード)は全部消したくなる衝動に駆られます. しかし, そうは言っても当然, 保守/運用/改善はしていかないといけないため, 何からしらの対策を立てなければいけません.

今回の記事のように, `テスト対象メソッド_期待される振る舞い_前提条件` のようなルールを決めることで, テストコードの統一性を保つことが出来れば, 少し幸せになれるのかなと思います.
