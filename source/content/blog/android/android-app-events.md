+++
date = "2015-09-06T00:00:00Z"
title = "Android: MVP, Dagger2, Retrofitなどなどでアプリを作りました"
tags = ["android"]
blogimport = true
type = "post"
+++

イベントを検索するAndroidアプリを作成したので, 使った技術のまとめです.

アプリの技術的機能, 特徴は以下になります.

- HTTPを介してイベントのデータを取得する
  - 検索対象のサーバは複数あるため(今回は3つ), どこかでそれらのレスポンスデータを同期する必要がある
    - それら複数のサーバーはJSONを返すが, 微妙にJSONの構造が異なる
- Data-Bindingライブラリを使う
- MVP(Model-View-Presenter)パターンを使う
  - Activity(Fragment)に機能が集中しないようにしたい!


上記を中心にどのように実装をしたかを説明をしていきます.

フルソースコードはここにあります. https://github.com/satoshun/AndroidEvents


## HTTPを介してイベントのデータを取得する

connpass, Atnd, Zusaarの3つのAPIを使うことにしました. [Retrofit](http://square.github.io/retrofit/)でAPIを定義し,
JSONのパースには[Gson](https://github.com/google/gson), データの処理には[RxJava](https://github.com/ReactiveX/RxJava)を使いました.

例えば, connpass APIは以下のように定義しました.

```java
/** Get data from Conpass  */
public interface Connpass {
    @GET("/v1/event")
    Observable<ConnpassResponse> search(
            @Query("ymd") List<String> ymds);

    @GET("/v1/event")
    Observable<ConnpassResponse> search(
            @Query("keyword") String keyword,
            @Query("ymd") List<String> ymds);
}
```

keywordとymdを指定してリクエストを生成します. 「2015/12/31のAndroidのイベント」のように使う想定です.
Retrofitを使うと, このようにinterfaceとして, APIを定義することが出来ます.
(https://github.com/satoshun/AndroidEvents/blob/master/app/src/main/java/com/github/satoshun/events/network/Connpass.java#L10)

次に, RxJavaの説明をします. RxJava(Observable)を使うと, 非同期にデータが扱いやすくなります. またデータのマージ(merge)や, フィルタリング(filter)などを簡単に行うことが出来ます.
今回は, 3つのAPI(connpass, Atnd, Zusaar)が終わるのを待ってから処理を開始したかったので, 以下のように書きました.

```java
Observable.merge(
  connpass.search(keyword, generateYmd()),
  atnd.search(keyword, generateYmd()),
  zusaar.search(keyword, generateYmd()))
  .subscribe(...)
```

`Observable.merge`は複数のObservableを1つのObservableにまとめるAPIです. これで, 3つのAPIが終了するまでwaitすることが出来ます.
あとは, これをsubscribeして, データを取得します.(https://github.com/satoshun/AndroidEvents/blob/master/app/src/main/java/com/github/satoshun/events/ui/domain/EventInteractor.java#L41)


## Data-Bindingライブラリを使う

Data-BindingはXMLレイアウトにbindしたいデータ(インスタンス)を記述することで, よしなにデータを出力してくれる機能です.
AngularJSのデータバインディングをイメージして貰えると良いと思います.
書くコード量が減り, とても便利なライブラリでした. (https://github.com/satoshun/AndroidEvents/blob/master/app/src/main/res/layout/adapter_event.xml)

また, 推奨された使い方かどうかはわからないですが, ViewHolderパターンとして使うことも出来ます.
ViewHolderパターンは, Adapter#getViewでコストが掛かる処理(View#findViewByIdなど)をcacheするパターンです.

```java
public class EventAdapter extends BaseAdapter {
    /*
      .. ....
     */

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        View view = convertView;
        if (view == null) {
            AdapterEventBinding binding = AdapterEventBinding.inflate(inflater, parent, false);
            view = binding.getRoot();
            view.setTag(binding);
        }

        AdapterEventBinding binding = (AdapterEventBinding) view.getTag();
        Event event = getItem(position);
        binding.setEvent(event);
        binding.setDateFormat(DATE_FORMAT);

        return view;
    }
}
```

AdapterEventBindingをViewHolderの代わりに使っています.
(https://github.com/satoshun/AndroidEvents/blob/master/app/src/main/java/com/github/satoshun/events/ui/adapter/EventAdapter.java#L34)
なかなか良いと思います.


## MVP(Model-View-Presenter)パターンを使う

MVPパターンとは, MVCの親戚?のようなパターンで, 責務をModel, View, Presenterにそれぞれ分割するパターンです.

Android開発は, Activity(Fragment)の責務が大きくなりがちです. 具体的にはActivityは以下のよう責務を持ちます.

- ユーザからのイベントハンドリング
  - クリック, ロングクリック, ...
- システムからのイベントハンドリング
  - 画面回転, アプリ終了, ...
- データの取得する際の非同期処理
  - HTTP(network), SQLite, SharedPreferences, ...
- 取得したデータをパースしてViewにパースしたデータを割り当てる

これらを全て1つのActivityで処理をすると, どうしてもFat-Activityになってしまいます. (１つのAcitvityが1000行ありますみたいな)

そこでMVPです.

- Presenter
  - Modelから(非同期に)データを取得し, Viewに取得したデータをどのようにに表示するかを指定する(ビューロジック)
- Model
  - データを取得してアプリで使いやすい形にデータをパースする. いわゆるビジネスロジック.
    - Retrofitを叩く
    - SQLiteにQueryを発行する
- View(Activity)
  - ユーザからのクリックイベントなど, イベントのハンドリングをする(onClickとか)
    - イベントの処理はPresenterに任せる
  - 必要なデータをPresenterから受け取り, 画面に表示する

このように責務を分割することで, Activityの責務が薄くなります.
今回のコードも今まで自分が書いたコードと比較すると, 大分良くなった気がします(当人比)


## その他, メモ

### Dagger2

DI(Dependency Injection)をするためのツールで, 実装をデバッグ時, 本番時, テスト時に切り替えられるライブラリです.
デバッグ時はデータの取得先を変えたい, テスト時にはネットワークアクセスしないで欲しい, などといった時に力を発揮します.


## まとめ

MVPパターンの話をしましたが, MVPが絶対良いという話ではないです.
しかし, 何もパターンがないとFat-Activityになってしまったり, 無秩序なコードになってしまいがちなので, そのような場合は, MVPのようなパターンを導入したほうが良いと思います.

まだアプリ自体はまだ付けたい機能があるため, 公開していません. 近々しようと思います.


## references

- [MVP for Android: how to organize the presentation layer](http://antonioleiva.com/mvp-android/)
