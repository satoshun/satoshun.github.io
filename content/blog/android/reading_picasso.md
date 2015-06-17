+++
date = "2015-06-12T00:00:00Z"
title = "Android: Picassoで使われているデザインパターン"
tags = ["android", "design_pattern"]
blogimport = true
type = "post"
+++

[Picasso](https://github.com/square/picasso)で使われているデザインパターンを紹介する記事になります.


## Singletonパターン

Singletonパターンは, インスタンスの生成を1つに制限するパターンになります.

https://github.com/square/picasso/blob/d35058278cff55874d133cfd63286dd0f1ff0d50/picasso/src/main/java/com/squareup/picasso/Picasso.java#L672


```java
public static Picasso with(Context context) {
  if (singleton == null) {
    synchronized (Picasso.class) {
      if (singleton == null) {
        singleton = new Builder(context).build();
      }
    }
  }
  return singleton;
}
```

Picasso#withは, すでにPicassoのインスタンス `singleton` が生成されていればそれを返し,
生成されていなければ, インスタンスを生成して返します.

このパターンのメリットは, インスタンスを高々1つしか作らないのでメモリ的に有利な点です.
しかし, singletonなインスタンスは, 複数のクラスから使われる可能性があるので, スレッドセーフでなければいけません.


## Builderパターン

Builderパターンはインスタンス生成時に多数のパラメータが必要なときに便利なパターンです.

https://github.com/square/picasso/blob/d35058278cff55874d133cfd63286dd0f1ff0d50/picasso/src/main/java/com/squareup/picasso/Picasso.java#L702

```java
public static class Builder {
  private final Context context;
  private Downloader downloader;
  private ExecutorService service;
  private Cache cache;
  private Listener listener;
  private RequestTransformer transformer;
  private List<RequestHandler> requestHandlers;
  private Bitmap.Config defaultBitmapConfig;

  private boolean indicatorsEnabled;
  private boolean loggingEnabled;

  public Builder(Context context) {
    if (context == null) {
      throw new IllegalArgumentException("Context must not be null.");
    }
    this.context = context.getApplicationContext();
  }

  ...
  ...

  public Picasso build() {
    Context context = this.context;

    if (downloader == null) {
      downloader = Utils.createDefaultDownloader(context);
    }
    if (cache == null) {
      cache = new LruCache(context);
    }
    if (service == null) {
      service = new PicassoExecutorService();
    }
    if (transformer == null) {
      transformer = RequestTransformer.IDENTITY;
    }

    Stats stats = new Stats(cache);

    Dispatcher dispatcher = new Dispatcher(context, service, HANDLER, downloader, cache, stats);

    return new Picasso(context, dispatcher, cache, listener, transformer, requestHandlers, stats,
        defaultBitmapConfig, indicatorsEnabled, loggingEnabled);
  }
```

必ず必要なパラメータContextはコンストラクタ引数として渡し, オプション的なパラメータは必要に応じてsetします.
パラメータのセットが完了したら, buildメソッドをコールします.

```
new Builder(context) // 必ず必要なパラメータ
    .debugging(true) // debuggingをtrueに
    .memoryCache(memoryCacheInstance) // 専用のmemoryCacheを使う
    .build(); // パラメータに異常がなければインスタンスを返す
```

Builderパターンを使うことで, コンストラクタの数を減らすことが出来ます.
また, Hoge(int, int, int)の時, 与えるintの順番を間違える可能性がありますが,
Builderパターンだと名前付きメソッドで値を指定できるので, 間違える可能性を下げることが出来ます.


## static factoryパターン

static factoryパターンは,　コンストラクタの代わりに, クラスのインスタンスを返すstatic methodを使用するパターンです.

```java
public static Picasso with(Context context) {
  if (singleton == null) {
    synchronized (Picasso.class) {
      if (singleton == null) {
        singleton = new Builder(context).build();
      }
    }
  }
  return singleton;
}
```

Picasso#with(Context)は, Picassoのインスタンスを返します. static factoryメソッドを使うことで, コンストラクタ以上の柔軟性を提供することが出来ます.
上の例で言うと, Picasso#withは, シングルトンパターンにより, 毎回インスタンスを生成する必要がありません. コンストラクタを使う場合は, 毎回インスタンスを生成する必要があります.
さらに, static factoryは自分自身だけでなく, サブクラス, インターフェースの実装を返すことも可能です.
上の例で言うとPicasso#withはPicassoのサブクラスを返しても問題なく動作します(もちろんサブクラスにバグがなければ).


## 早期リターンパターン(early return pattern)

早期リターンパターンは, **メソッドの先頭** で, 何もせずにメソッドを終了するか, 例外をスローするパターンです.

https://github.com/square/picasso/blob/d35058278cff55874d133cfd63286dd0f1ff0d50/picasso/src/main/java/com/squareup/picasso/RequestCreator.java#L519


```
public void into(Target target) {
  long started = System.nanoTime();

  // こっから例外などの判定
  checkMain();

  if (target == null) {
    throw new IllegalArgumentException("Target must not be null.");
  }
  if (deferred) {
    throw new IllegalStateException("Fit cannot be used with a Target.");
  }

  if (!data.hasImage()) {
    picasso.cancelRequest(target);
    target.onPrepareLoad(setPlaceholder ? getPlaceholderDrawable() : null);
    return;
  }
  // 例外などの判定修了

  // こっからメインロジック
  Request request = createRequest(started);
  String requestKey = createKey(request);

  if (shouldReadFromMemoryCache(memoryPolicy)) {
    Bitmap bitmap = picasso.quickMemoryCacheCheck(requestKey);
    if (bitmap != null) {
      picasso.cancelRequest(target);
      target.onBitmapLoaded(bitmap, MEMORY);
      return;
    }
  }

  target.onPrepareLoad(setPlaceholder ? getPlaceholderDrawable() : null);

  Action action =
      new TargetAction(picasso, target, request, memoryPolicy, networkPolicy, errorDrawable,
          requestKey, tag, errorResId);
  picasso.enqueueAndSubmit(action);
}
```

1. 変数targetがnullなら例外をスロー
2. 変数deferredがtrueなら例外をスロー
3. data.hasImage()がfalseなら, cancelRequestをコールしてreturn
4. メインのロジックの実行

メソッドのエラー処理の部分をメソッドの最初に, メインロジックの部分をその後に書くことで, 可読性を上げることが出来ます.


## ViewHolderパターン

Android特有のパターンです. ListViewで子要素を切り替えるたびに毎回View#findViewByIdをViewを実行するのはコストが高いので,
Cacheしておくパターンです.(Picasso本体ではなく, exampleフォルダのコード例になります)

https://github.com/square/picasso/blob/ceafe59cbecbc1e1a75cc6a14d028ebba3145cbe/picasso-sample/src/main/java/com/example/picasso/SampleListDetailAdapter.java#L66

```java
@Override public View getView(int position, View view, ViewGroup parent) {
  ViewHolder holder;
  if (view == null) {
    view = LayoutInflater.from(context).inflate(R.layout.sample_list_detail_item, parent, false);
    holder = new ViewHolder();
    holder.image = (ImageView) view.findViewById(R.id.photo);
    holder.text = (TextView) view.findViewById(R.id.url);
    view.setTag(holder);
  } else {
    holder = (ViewHolder) view.getTag();
  }

  ...
}

static class ViewHolder {
  ImageView image;
  TextView text;
}
```

BaseAdapter#getViewで, Viewを生成するときにそのViewが保持すべきwidgetをViewHolderに保存しておきます.
こうすることで, 次回以降の処理を短くすることが出来ます.


## Observerパターン

非同期な処理が完了, 状態が変化したことを, クライアント(主に呼び出し元のインスタンス)に通知をする時に使われるパターンです. 非常にポピュラーなパターンです.

https://github.com/square/picasso/blob/d35058278cff55874d133cfd63286dd0f1ff0d50/picasso/src/main/java/com/squareup/picasso/RequestCreator.java#L647

```java
public void into(ImageView target, Callback callback) {
  long started = System.nanoTime();
  checkMain();

  ...

  if (shouldReadFromMemoryCache(memoryPolicy)) {
    Bitmap bitmap = picasso.quickMemoryCacheCheck(requestKey);
    if (bitmap != null) {
      picasso.cancelRequest(target);
      setBitmap(target, picasso.context, bitmap, MEMORY, noFade, picasso.indicatorsEnabled);
      if (picasso.loggingEnabled) {
        log(OWNER_MAIN, VERB_COMPLETED, request.plainId(), "from " + MEMORY);
      }
      if (callback != null) {
        callback.onSuccess();
      }
      return;
    }
  }
}
```

into(ImageView, Callback)の, Callbackの部分がObserverパターンのポイントになります.
intoメソッドは非同期な処理のため, 結果が成功したかを返り値として受け取ることが出来ません.
そこで, 非同期処理が終わったら, 引数で渡したcallbackをコールするようにすることで結果を受け取ることが出来ます.
