+++
date = "2016-11-23"
title = "Android: Repository層についてあれこれ"
tags = ["android", "architecture", "repository"]
blogimport = true
type = "post"
draft = false
+++

参考: https://github.com/googlesamples/android-architecture/tree/todo-mvp


## Repository層とは?

data層を抽象する層. Repositoryではアプリのキャッシュポリシーを担当する.
キャッシュはメモリ, ディスクのどちらかに保存するかに依存しないようにする. そこの実装はdata層に任せる.

具体的には, Repositoryは, remote dataオブジェクトと, local dataオブジェクトの2つのオブジェクトに依存するようにし,
どこにキャッシュされるかはlocal dataオブジェクトに従う.
こうすることで, Repositoryはどこにデータが保存されているかを気にしなくて良い. あくまでRepositoryはどこのdataオブジェクトからデータを取ってくるのかを決定する.

また, Repositoryは基本的にApplicationのライフサイクルに連動し, SingletonでActivity間でデータの共有を効率的に行える.
例えば, Activity Aで取得したデータはRepositoryのキャッシュポリシーに従いキャッシュをし, Activity Bでは, キャッシュポリシーに従い即座にデータを取得できる可能性がある.(ポリシーによっては毎回remoteからデータ取得する可能性もある)


## RxJavaとともに

RepositoryはRxJava(Observable)との相性が良いと思っています. 例えば以下のようにコールバック関数無く書けるからです.

```java
@NonNull
@Override public Observable<User> getUser(int id) {
return localDataSource.getUser(id)
        .onErrorResumeNext(remoteDataSource.getUser(id))
        .doOnNext(new Action1<User>() {
            @Override public void call(User user) {
            localDataSource.saveUser(user);
            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
}
```

上のソースコードは, localDataSourceからデータを取得できなければ, remoteDataSourceに取得しに行くコードになります.
クライアントでは, local, remoteの違いを気にしなくて良いというのが一番の利点です.


## まとめ

ざっくりと書きました. キャッシュの抽象化(Repository層) + RxJavaは非常に強力で, とても良いパターンだと思っています.
クライアント(ContrllerやPresenter)のコードもきれいになり, またActivity間のデータの共有もスムーズに行うことができるので(最悪キャッシュがリリースされていてもRemoteから取得してくれる),
Androidの開発に向いているパターンでもあると思います.
