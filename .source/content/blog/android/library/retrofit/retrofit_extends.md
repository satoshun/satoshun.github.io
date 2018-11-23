+++
date = "2018-12-01T00:00:00Z"
title = "Retrofitインターフェースの継承について"
tags = ["android", "retrofit"]
blogimport = true
type = "post"
draft = true
+++

RetrofitはインターフェースにAPIの定義をすることで、
HTTP通信を行うことができるHTTPクライアントライブラリです。

この記事では、Retrofitのインターフェースの継承についてまとめます。

Retrofitではインタフェースの継承を意図的に禁止しています。

```java
// Prevent API interfaces from extending other interfaces. This not only avoids a bug in
// Android (http://b.android.com/58753) but it forces composition of API declarations which is
// the recommended pattern.
if (service.getInterfaces().length > 0) {
    throw new IllegalArgumentException("API interfaces must not extend other interfaces.");
}
```

禁止している理由は2つです。

1. 特定のAndroidバージョンでは動かない
2. 継承ではなくコンポジションを推奨している

1はバグなので禁止している理由としては妥当だと思います。

次に2についてです。

詳細はこの[issue/504](https://github.com/square/retrofit/issues/504)にまとまっています。

- Generic型を使いたい

```java
interface GenericAPI<R extends GenericResponse> {
   void obtainData(String name, Callback<R> callback)
}

interface GetUsersAPI extends GenericAPI<UsersResponse> {
   @GET("/dataAPI/{version}/{name}")
   void obtainData(@Path("name") String name, Callback<UsersResponse> callback)
}
```

- Loginユーザ、非ログインユーザなど状況に応じてAPIの向き先を変えたい

```java
interface LoginService {
  void login(String name)
}

interface GuestLoginService extends LoginService {
  @GET("guest/login")
  @Override
  void login(String name)
}

interface LoginedInService extends LoginService {
  @GET("logged_in/login")
  @Override
  void login(String name)
}
```

確かにこういうユースケースはありそうです。

しかし、これがRetofitがカバーするべきユースケースかというと、著者の意向に次第だと思います。
現状では、これはRetrofitがカバーすべきユースケースではないと判断しているようです。
ただ、これは確定ではなく、慎重に検証したいという立場のようです。
今後このユースケースが非常に一般的で、かつRetrofitでカバーしたほうが良いと判断されれば、十分に実装される可能性がありそうです。
