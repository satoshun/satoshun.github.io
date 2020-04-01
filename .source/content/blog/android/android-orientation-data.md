+++
date = "2016-10-09"
title = "Android: Fragmentを使って、画面の向きの変更時にデータを保持する"
tags = ["android", "rxjava"]
blogimport = true
type = "post"
draft = false
+++

`onSaveInstanceState`, `onRestoreInstanceState`を使う方法もありますが, 今回はFragmentを使う方法を紹介します.

サンプルコードは https://github.com/satoshun-example/AndroidRetainData にあります.

## ユースケース

ネットワークはコストが掛かるので, 1度データを取得したら, 画面回転でアクティビティが再生成されたとしてもデータを使いまわしたい, というケースがあるとします.


## やり方

まず, データを保持するFragmentを定義します.

最初にonCreateで, setRetainInstance(true)をコールします.

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setRetainInstance(true);

    refresh();
}
```

次に, refreshでデータを取得します. ここではRxJava2 + Retrofitを使っています.

```java
void refresh() {
    disposables.clear();
    disposables.add(provideGithubAPI()
            .getRepositories("satoshun")
            .subscribeOn(Schedulers.computation())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                    data -> subject.onNext(data),
                    e -> subject.onError(e)));
}
```

APIコールをして, 取得したデータをBehaviorSubject(subject)に与えます.

最後にobservableメソッドを定義し, データソース(subject)を他のオブジェクトからアクセス出来るようにします.

```java
Observable<List<Repo>> observable() {
    return subject;
}
```

Fragmentのフルソースコードは[ここ](https://github.com/satoshun-example/AndroidRetainData/blob/master/app/src/main/java/com/github/satoshun/example/retain/RetainFragment.java)にあります.

次に, Activityの準備をします. ActivityのonCreateでデータFragmentを使うようにするだけです.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    dataFrag = (RetainFragment) getSupportFragmentManager().findFragmentByTag("data");
    if (dataFrag == null) {
        dataFrag = RetainFragment.newInstance();
        getSupportFragmentManager()
                .beginTransaction()
                .add(dataFrag, "data")
                .commit();
    }
    // ここで, データFragmentからデータを取得する!!
    disposables.add(dataFrag.observable()
                .subscribe(
                        data -> {
                            binding.swipe.setRefreshing(false);
                            repoAdapter.addRepos(data);
                        },
                        e -> {
                            binding.swipe.setRefreshing(false);
                            Log.e("onDataSource", e.getMessage());
                        }));
}
```

Activityのフルソースコードは[ここ](https://github.com/satoshun-example/AndroidRetainData/blob/master/app/src/main/java/com/github/satoshun/example/retain/MainActivity.java)にあります.

これで, データFragmentで取得したデータを使いまわせます!


## まとめ

FragmentはUIだけでなく, データオブジェクトとしても有能.
