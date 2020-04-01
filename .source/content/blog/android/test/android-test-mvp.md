+++
date = "2016-07-31T00:00:00Z"
title = "Android: MVPのPresenterの非同期周りのユニットテストの書き方"
tags = ["android", "test"]
blogimport = true
type = "post"
draft = false
+++

MVP(Model-View-Presenter)アーキテクチャが流行ってますね!

MVPはテスタブルであることが1つのウリとなっていますが, 実際にPresenterのユニットテストをどのようにして書くかを紹介したいと思います.

今回のコードは https://github.com/satoshun-example/AndroidTestSample に上げてあるので, 良かったら見て下さい.


## 使うツール/ライブラリ

- JUnit4
- Mockito


## 今回, テスト書きたい箇所

今回は, `MainPresenter`クラスをテストします.

```java
public class MainPresenter implements MainContract.Presenter {

    @NonNull
    private final UserDataSource dataSource;

    @NonNull
    private final MainContract.View view;

    public MainPresenter(@NonNull UserDataSource dataSource, @NonNull MainContract.View view) {
        this.dataSource = dataSource;
        this.view = view;
    }

    @Override
    public void start() {
        view.showLoadingIndicator();

        dataSource.getUser(new UserDataSource.Callback() {
            @Override
            public void onUserLoaded(@NonNull User user) {
                view.hideLoadingIndicator();
                view.showUser(user);
            }

            @Override
            public void onNotAvailable() {
                view.hideLoadingIndicator();
                view.showNoUser();
            }
        });
    }
}
```

```java
public interface MainContract {

    interface View {
        void showUser(@NonNull User user);

        void showNoUser();

        void showLoadingIndicator();

        void hideLoadingIndicator();
    }

    interface Presenter {
        void start();
    }
}
```

```java
public interface UserDataSource {
    interface Callback {
        void onUserLoaded(@NonNull User user);

        void onNotAvailable();
    }

    void getUser(@NonNull Callback callback);
}
```

`UserDataSource`はモデルだと思って下さい.

`UserPresenter`は, `UserDataSource`, `MainContract.View`をコンストラクタから受け取り, 2つのクラスを適切にコミュニケーションさせるクラスとなります.

今回の場合, `UserPresenter#start`がコールされた時, `UserDataSource`からデータを非同期に取得し, `View`に値を返すクラスとなっています.


## 非同期なAPIをどうユニットテストで表現するか

僕の言語能力では説明が難しいので, 最初にテストコードを紹介します.

```java
public class MainPresenterTest {

    @Rule
    public MockitoRule mockitoRule = MockitoJUnit.rule();

    @Mock
    private UserDataSource dataSource;

    @Mock
    private MainContract.View view;

    @Captor
    private ArgumentCaptor<UserDataSource.Callback> callbackCaptor;

    @Test
    public void showUser_successNetworkAccess() throws Exception {
        MainPresenter presenter = new MainPresenter(dataSource, view);
        User dummyUser = new User("dummy", "dummy");

        presenter.start();

        verify(dataSource).getUser(callbackCaptor.capture());
        callbackCaptor.getValue().onUserLoaded(dummyUser);

        verify(view, times(1)).showUser(dummyUser);
    }

    @Test
    public void showNoUser_failureNetworkAccess() throws Exception {
        MainPresenter presenter = new MainPresenter(dataSource, view);

        presenter.start();

        verify(dataSource).getUser(callbackCaptor.capture());
        callbackCaptor.getValue().onNotAvailable();

        verify(view, times(1)).showNoUser();
    }
}
```

※ JUnit4, Mockitoを使っています.

`verify(dataSource).getUser(callbackCaptor.capture());`でCallbackオブジェクトを取得し, 取得したCallbackオブジェクトのonUserLoaded or onNotAvailableメソッドをコールすることで, 非同期APIが成功 or 失敗を表現します.

そして, 成功した場合はshowUserメソッドがコールされるはずなので, `verify(view, times(1)).showUser(dummyUser);`と表現します. (times(1)はなくても大丈夫です. 僕は1回しか呼ばれていないことを強調するために書くのが好ましいと思っています)

これで, MainPresenterの非同期API周りのテストが書けました!


## まとめ

今回は, Presenterのユニットテストの書き方について紹介しました.
ここをこうしたほうが良いよとか, お前のユニットテストの書き方は間違っている! というのがありましたらぜひ教えて下さい(/ω・＼)
