+++
date = "Fri Feb 22 13:37:11 UTC 2019"
title = "todo"
tags = ["android", "test", "androidx", "robolectric", "espresso"]
blogimport = true
type = "post"
draft = true
+++

Robolectric4.xからユニットテスト環境で、android test（実機）と同じテストコードを動かすことが可能になりました。
まだ、完全に互換性があるとはいえませんが、Espressoライブラリが動く、`AndroidJUnit4`ランナーが使えるなど、かなりの部分が対応しています。

この記事では、ユニットテストで`Activity.onActivityResult`のテストを書いてみます。

## テスト対象のアプリコードコード

まず最初に、テスト対象コードは次のようになっています。

```kotlin
class MainActivity : AppCompatActivity() {
  ...
  override fun onCreate(savedInstanceState: Bundle?) {
    ...

    button.setOnClickListener {
      startActivityForResult(
        Intent(this, Sub2Activity::class.java).apply {
          putExtra("fuga", "hoge")
        },
        1
      )
    }
  }

  override fun onActivityResult(
    requestCode: Int,
    resultCode: Int,
    data: Intent?
  ) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == 1) {
      if (resultCode == Activity.RESULT_OK) {
        val value = data!!.getIntExtra("test", -1)
        button.text = value.toString()
      }
    }
  }
}
```

```kotlin
class Sub2Activity : AppCompatActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.sub_act)

    button.setOnClickListener {
      val intent = Intent()
      intent.putExtra("test", 10)
      setResult(Activity.RESULT_OK, intent)
      finish()
    }
  }
}
```

このサンプルコードが意味しているところは次になります。

1. MainActivityのボタンをクリックする
    2. `startActivityForResult`がコールされ、Sub2Activityが開始
2. Sub2Activityのボタンをクリックする
    3. `setResult`メソッドがコールされ、終了
3. MainActivityの`onActivityResult`メソッドに結果が返ってくる
    4. 結果がViewに反映される

となっています。では、これらを満たすテストを書いていきます。

## テストコード

最初にテストコードをのせます。

```kotlin
@RunWith(AndroidJUnit4::class)
internal class MainActivityTest {
  @get:Rule val intentsTestRule = IntentsTestRule(MainActivity::class.java)

  @Test
  fun onActivityResultTest() {
    val expectCode = 10

    // finish test
    val scenario = ActivityScenario.launch(Sub2Activity::class.java)
    scenario.onActivity {
      it.findViewById<View>(R.id.button).performClick()
    }

    // resultCode test
    val result = scenario.result
    assertThat(result.resultCode).isEqualTo(Activity.RESULT_OK)

    // intent params test
    val bundleSubject = IntentSubject.assertThat(result.resultData).extras()
    bundleSubject.integer("test").isEqualTo(expectCode)

    scenario.close()

    Intents
      .intending(
        IntentMatchers.hasComponent(
          ComponentName(
            ApplicationProvider.getApplicationContext<Application>(),
            Sub2Activity::class.java
          )
        )
      )
      .respondWith(result)

    val main = ActivityScenario.launch(MainActivity::class.java)
    main.onActivity {
      it.findViewById<View>(R.id.button).performClick()

      // check intent for startActivity(ForResult)
      val name = ComponentName(
        ApplicationProvider.getApplicationContext<Application>(),
        Sub2Activity::class.java
      )
      Intents.intended(IntentMatchers.hasComponent(name))
      Intents.intended(IntentMatchers.hasExtra("fuga", "hoge"))

      // check onActivityResult behaves
      Espresso
        .onView(ViewMatchers.withId(R.id.button))
        .check(ViewAssertions.matches(ViewMatchers.withText(expectCode.toString())))
    }
  }
}
```

ちょっと長いですが、やっていることは大したことないです。上から説明していきます。

```kotlin
@get:Rule val intentsTestRule = IntentsTestRule(MainActivity::class.java)
```

これは、Espresso-Intentsを使うときに設定するルールです。`Intents.intended`、`intending`が使えるようになります。

```kotlin
val scenario = ActivityScenario.launch(Sub2Activity::class.java)
scenario.onActivity {
    it.findViewById<View>(R.id.button).performClick()
}
```

ActivityScenarioはActivityを起動するためのクラスです。Sub2Activityを起動して、ボタンをクリックしています。

```kotlin
val result = scenario.result
assertThat(result.resultCode).isEqualTo(Activity.RESULT_OK)

val bundleSubject = IntentSubject.assertThat(result.resultData).extras()
bundleSubject.integer("test").isEqualTo(expectCode)
```

ActivityScenarioでは、ActivityResultを取得することが出来ます。このクラスにはresultCodeと、resultDataがセットされており、それらをTruthを使い値をチェックしています。

```kotlin
Intents
  .intending(
    IntentMatchers.hasComponent(
      ComponentName(
        ApplicationProvider.getApplicationContext<Application>(),
        Sub2Activity::class.java
      )
    )
  )
  .respondWith(result)
```

`Intents.intending`はマッチしたIntentが発行されたときに、onActivityResultに結果を返すAPIになります。
先ほど取得したActivityResultを返すように設定します。

```kotlin
val main = ActivityScenario.launch(MainActivity::class.java)
main.onActivity {
  it.findViewById<View>(R.id.button).performClick()

  // check intent for startActivity(ForResult)
  val name = ComponentName(
    ApplicationProvider.getApplicationContext<Application>(),
    Sub2Activity::class.java
  )
  Intents.intended(IntentMatchers.hasComponent(name))
  Intents.intended(IntentMatchers.hasExtra("fuga", "hoge"))

  // check onActivityResult behaves
  Espresso
    .onView(ViewMatchers.withId(R.id.button))
    .check(ViewAssertions.matches(ViewMatchers.withText(expectCode.toString())))
}
```

`Intents.intended`でstartActivtyForResuotで渡したIntentの中身が正しいことを確認します。最後に、Espressoを使って、`onActivityResult`の結果が正しく反映されているかを確認しています。

これで、テスト完了です😃

## 補足

Espressoにはご存知、clickをするためのAPIがあるのですが、うまく動きませんでした。

```kotlin
// not working!!
Espresso
  .onView(ViewMatchers.withId(R.id.button))
  .perform(ViewActions.click())
```

調べたんですが、わかりませんでした😂分かり次第追記します。

## まとめ

- 上記のテストくらいなら、ユニットテストで書ける。すごい😃
