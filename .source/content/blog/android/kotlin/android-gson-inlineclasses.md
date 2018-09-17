+++
date = "2018-09-17"
title = "Inline classesとGsonでprimitive型をクラスで受けとる"
tags = ["android", "kotlin","gson"]
blogimport = true
type = "post"
+++

（この記事は1.3-M2を使っています。）

Kotlin 1.3でInline classesが入ります。これはパフォーマンスに影響を与えずに、値のラッパークラスを作成することが出来ます。

例えば、次のように書くことが出来ます。

```kotlin
inline class UserId(val id: String) {
    val url get() = "http://$id"
}

val userId = UserId("user-dayo")
println(userId.url)
```

このコードは一見、UserIdクラスが生成されそうです。しかし、コンパイルされたコードではUserIdクラスは生成されません。

```java
public static final class UserId$Erased {
    ...

    @NotNull
    public static final String getUrl(String $this) {
        return "http://" + $this;
    }

    ...
}

String userId = "user-dayo";
String var1 = UserId$Erased.getUrl(userId);
System.out.println(var1);
```

UserIdのインスタンスを作らずに、Stringをそのまま使っていることが分かります。そして自動生成された`UserId$Erased`クラスにあるstaticメソッドを実行しています。Inline classesでは、インスタンスを生成せずにstaticメソッドをコールすることで、インスタンス生成のコストを抑えています。

ここからが本題です。
Inline classesがAndroid開発のどこで役立つのかを考えたときに、 Gsonなどのライブラリによってdeserialize/serializeされるクラスで有効使えると思いました。

例えば、次のコードがあったとします。

```kotlin
data class Response(
    @SerializedName("user_id") val userId: String,
    @SerializedName("friend_id") val friendId: String
)
```

これはuserIdとfriendIdをStringで受け取っており、このStringが何のStringかの情報が欠落しています。型による分類が出来てない状態です。

これにInline classesを使ってみます。

```kotlin
inline class UserId(private val id: String) {
    val url get() = "http://my/$id"
}

inline class FriendId(private val id: String) {
    val url get() = "http://friend/$id"
}

data class Response(
    @SerializedName("user_id") val userId: UserId,
    @SerializedName("friend_id") val friendId: FriendId
)
```

UserIdとFriendIdクラスで受け取ることが出来ます。Inline classesがコンパイルされると、wrapされた値の型が展開されるためです。UserId、FriendIdのwrapされた値はStringなので、コンパイル後はStringが展開され、Stringでdeserialize/serializeされます。しかし、コンパイルされる前までは型安全の恩恵を受け取ることが出来ます。

Inline classesを使えば、Adapterを作成することなく、カスタムのクラスで受け取れるため非常に便利です(・o・)
