+++
date = "2017-11-26"
title = "Android: Roomにおけるデータ変更通知の仕組みについて(InvalidationTracker)"
tags = ["android", "jetpack", "room"]
blogimport = true
type = "post"
draft = false
+++

こんにちはʕ•ӫ̫͡•ʔ

この記事ではどのようにしてRoomがテーブルの変更を検知し、LiveDataなどからデータの変更をオブザーバーに通知しているかについて説明します。


## 前置き/前提知識

[Room](https://developer.android.com/topic/libraries/architecture/room.html#db-migration)はGoogleが提供しているSQLiteをいい感じに扱うためのライブラリです。

RoomにはDao(Data access object)と呼ばれるインターフェース/抽象クラスがあり、そこを起点にクエリを実行します。

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getUsers(): Flowable<List<User>>
}
```

返り値として、observableデータ型を取ることができ、RoomではLiveDataとRxJava2の型(Flowableとか)などのobservable型サポートしています。

そして、LiveDataまたはFlowableで返り値を定義すると対象のテーブルがInsertなどで変更されたときに、変更通知を受け取ることが出来ます。

```kotlin
// クライアント側のコード
userDao.getUsers()
  .subscribeOn(Schedulers.io)
  .subscribe {
     println(it) // ここが対象のテーブルが変更されるたびに評価される。
  }
```

テーブルの変更検知はRoomの内部クラスの`InvalidationTracker`クラスで行っています。
これから`InvalidationTracker`の内部実装を見ていきます。


## InvalidationTrackerの流れ

1. InvalidationTracker#startTrackingTableメソッドでTrigger(SQL)を生成
2. obsevableが生成されたら、対応するInvalidationTracker.Observer(変更通知用コールバック)をInvalidationTrackerに登録
3. RoomData#endTransactionのタイミングで各テーブルが更新されたかを確認
4. テーブルが更新されていたら変更通知コールバックを実行

というのが大まかな流れになります。2はobserver(変更通知用コールバック)を登録するだけ、4は登録したコールバックにデータを流すだけです。なので1, 3について詳しく見ていきます。


### InvalidationTracker#startTrackingTableメソッドによりTriggerを生成する

最初に各テーブルのバージョンを管理する `room_table_modification_log`が生成されます。

```sql
CREATE TEMP TABLE room_table_modification_log (
    version INTEGER PRIMARY KEY AUTOINCREMENT,
    table_id INTEGER
)
```

このクエリの意味することは、テーブル操作があったときに、対象のテーブルのversionを更新するようにすることで、version変更されていたら更新を検知することが出来ます。このversionを更新するためにRoomではSQLにある機能の1つTriggerを使っています。具体的には次のTriggerを利用しています。

```sql
CREATE TEMP TRIGGER IF NOT EXISTS room_table_modification_trigger_users_UPDATE AFTER UPDATE ON users
BEGIN
    INSERT OR REPLACE INTO room_table_modification_log VALUES (null, 0); # 0はtable_id
END
```

これは一見複雑に見えますがものすごい単純で、usersテーブルがUPDATEされたら上記のクエリがフックされて実行されるだけです。上のクエリ`INSERT OR REPLACE INTO room_table_modification_log VALUES (null, 0); `は `room_table_modification_log`テーブルに最新のid + 1のrowを追加/更新するものになります。

上のTriggerはUPDATEですが、その他にDELETE、INSERT用のTriggerも実行されます。これらのTriggerによりテーブルの変更(Update, Insert, Delete)を検知し、前回のversionを新しいものに更新します。


### RoomData#endTransactionのタイミングで各テーブルが更新されたかを確認する

Triggerによりversionを更新できたので、次にその更新したテーブルを前回の状態と比較し、差分があるなら発火してあげる必要があります。それは `RoomData#endTransaction`のタイミングで行っています。

endTransactionのタイミングで`InvalidationTracker#refreshVersionsAsync`をコールし、`room_table_modification_log`テーブルに変更があったら、InvalidationTrackerに登録にした`InvalidationTracker.Observer`に対してデータ変更のコールバックをします。これにより生成されたobservable(FlowableとかLiveDataとかとか)に対して変更通知をすることが出来ます。

これがRoomにおけるおおまかな変更検知/通知の仕組みになります。


## まとめ

- SQLのTriggerを使っているところが個人的には意外というか、こんな実装があるんだと感心した
- 上手く説明できた気がしないので、より詳細はInvalidationTrackerの実装を読んで下さい(゜レ゜)
