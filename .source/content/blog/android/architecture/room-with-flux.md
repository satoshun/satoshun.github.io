+++
date = "Thu Jan 24 00:43:33 UTC 2019"
title = "Room + Fluxは冗長なコードが多くなるので良くなさそう"
tags = ["android", "flux", "room", "jetpack", "architecture"]
blogimport = true
type = "post"
+++

FluxのDispatcherをRoom in memoryで実装すれば最高なのでは?と思い、ちょっと試してみました。

結論から申しますと冗長なコードが多く、良くないと感じました。EventBusや、他のライブラリを使って実装したほうが良いと思います😂

また、オリジナルFluxは「Dispatcherがアプリ内で1つだけ存在する」という原則があったと思いますが、それを破っています。Fluxですらない可能性があります。

## Room in memory?

Roomではin memoryでデータベースを作ることが出来ます。正確に言えば、SQLiteの機能をRoomのAPIとして開放しています。

使い方は次のようになります。

```kotlin
Room
  .inMemoryDatabaseBuilder(context, MyDatabase::class.java)
  .build()
```

in memoryを使う理由としては、

- ディスパッチするアクションを永続化する必要はないだろう
- マイグレーションが必要ない

になります。

## 実装に入っていく

では、実装の説明をしていきます。

まずはActionをRoomのEntityとして定義します。

```kotlin
sealed class AuthorAction

@Entity(tableName = "author1")
data class Author1(
  @PrimaryKey val _id: Long = 0, // always 0
  val name: String,
  val age: Int
) : AuthorAction()

@Entity(tableName = "author2")
data class Author2(
  @PrimaryKey val _id: Long = 0, // always 0
  val name: String,
  val age: Int
) : AuthorAction()
```

Primary keyは常に一定にして、アクションは0 or 1つしか存在しないようにしておきます。仮にアクションの履歴が欲しいなら、`@PrimaryKey(autoGenerate = true)`を使っても良いと思います。

次にDaoを定義します。これはFluxでいうところのDispatcherになります。

```
@Dao
interface AuthorDispatcher {
  @Insert(onConflict = OnConflictStrategy.REPLACE) fun dispatch(author: Author1)
  @Insert(onConflict = OnConflictStrategy.REPLACE) fun dispatch(author: Author2)

  // Storeに相当する
  @Query("select * FROM author1 WHERE _id = 0")
  fun author(): LiveData<Author1?>

  // Storeに相当する
  @Query(
    """
    select author1.name as name1, author2.name as name2
    FROM author1
     INNER JOIN author2
    WHERE author1._id = 0 AND author2._id = 0
    """
  )
  fun mappedAuthor(): LiveData<MappedAuthor?>
}
```

となります。`@Insert`でアクションをdispatchメソッドを、`@Query`でsubscribeメソッドを実装しています。

これで、FluxのDispatcherに似た何かをRoomで表現することが出来ます！

## まとめ/考察

RoomでDispatcher的なのを作る方法と、EventBusなどを使ったアプローチの違いは以下になります。

- Storeに書いていたロジックをSQLに任せることが出来る
  - Transactionを上手く使えば、マルチスレッド環境でもそれっぽく動きそう
- ほげほげDispactherクラスがアプリ内に蔓延する
- コードが冗長😂

工夫すれば、もう少しキレイに書けるとは思いますが、ただEventBusや、Coroutineを使ったアプローチには敵わないと思っています。
FluxのDispatcherの代替としては辛いですが、他の用途、例えばRepositoryの実装などには使える余地があると思うので、そういったところで思い出していただけたら幸いです😊

検証に用いたサンプルコードは[RoomDispatcherExample](https://github.com/satoshun-android-example/RoomDispatcherExample)にあります。
