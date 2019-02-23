+++
date = "Sat Feb 23 02:47:59 UTC 2019"
title = "Robolectric + JetpackでActivityのonActivityResultメソッドをテストする"
tags = ["android", "test", "jetpack", "robolectric", "espresso"]
blogimport = true
type = "post"
+++

Robolectric4.xからユニットテスト環境で、android testと（ほぼ?）同じテストコードを動かすことが可能になりました。
まだ、完全に互換性があるとはいえませんが、Espressoライブラリが動く、`AndroidJUnit4`ランナーが使えるなど、かなりの部分が共通化出来ます。

この記事では、ユニットテストで`Activity.onActivityResult`のテストをどこまでandroid testのように書けるかを検証します。

## テスト対象コード

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

これはMainActivityで`startActivityForResult`がコールされ、Sub2Activityで`setResult`で値をセットし、MainActivityの`onActivityResult`で結果を受け取るサンプルコードになります。

では、テストを書いていきます。

## テストコード

以下が、今回書いたテストコードになります。

```kotlin
@RunWith(AndroidJUnit4::class)
internal class MainActivityTest {
  @get:Rule val intentsTestRule = IntentsTestRule(MainActivity::class.java)

  @Test
  fun onActivityResultTest() {
    val expectCode = 10

    // assertion setResult
    val scenario = ActivityScenario.launch(Sub2Activity::class.java)
    scenario.onActivity {
      it.findViewById<View>(R.id.button).performClick()
    }

    // assertion resultCode
    val result = scenario.result
    assertThat(result.resultCode).isEqualTo(Activity.RESULT_OK)

    // assertion intent params
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

      // assertion intent for startActivity(ForResult)
      val name = ComponentName(
        ApplicationProvider.getApplicationContext<Application>(),
        Sub2Activity::class.java
      )
      Intents.intended(IntentMatchers.hasComponent(name))
      Intents.intended(IntentMatchers.hasExtra("fuga", "hoge"))

      // assertion onActivityResult behaves
      Espresso
        .onView(ViewMatchers.withId(R.id.button))
        .check(ViewAssertions.matches(ViewMatchers.withText(expectCode.toString())))
    }
  }
}
```

上から順番に重要な部分を説明していきます。

```kotlin
@get:Rule val intentsTestRule = IntentsTestRule(MainActivity::class.java)
```

これは、Espresso-Intentsを使うときに必要なルールです。`Intens.intended`、`intending`を使うために必要なルールになります。

```kotlin
val scenario = ActivityScenario.launch(Sub2Activity::class.java)
scenario.onActivity {
    it.findViewById<View>(R.id.button).performClick()
}
```

ActivityScenarioはActivityを起動するためのクラスです。これはSub2Activityを起動して、ボタンをクリックするという意味になります。
ボタンがクリックされると、Sub2Activityで`setResult`が発火するようになっています。

```kotlin
val result = scenario.result
assertThat(result.resultCode).isEqualTo(Activity.RESULT_OK)

val bundleSubject = IntentSubject.assertThat(result.resultData).extras()
bundleSubject.integer("test").isEqualTo(expectCode)
```

ActivityScenarioでは、ActivityResultクラスから結果を取得することが出来ます。このクラスにはresultCodeと、resultDataがセットされており、それらの値をTruthを使いチェックします。この場合、`setResult`で、resultcodeに`Activity.RESULT_OK`が、resultdataにはキー名`test`、値10がセットされていることを確認してします。

ここまでで、Sub2ActivityのsetResultで正しい値をセットしていることがテスト出来ます。

では次に、MainActivityで上記の値を受け取れることをテストしていきます。

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
MainActivityのonActivityResultに、先ほどのSub2Activityの結果を渡すという意味になります。

```kotlin
val main = ActivityScenario.launch(MainActivity::class.java)
main.onActivity {
  it.findViewById<View>(R.id.button).performClick()

  val name = ComponentName(
    ApplicationProvider.getApplicationContext<Application>(),
    Sub2Activity::class.java
  )
  Intents.intended(IntentMatchers.hasComponent(name))
  Intents.intended(IntentMatchers.hasExtra("fuga", "hoge"))

  Espresso
    .onView(ViewMatchers.withId(R.id.button))
    .check(ViewAssertions.matches(ViewMatchers.withText(expectCode.toString())))
}
```

まずは、クリックイベントを発火し、`startActivityForResult`をコールします。渡したIntentを`Intents.intended`で正しいことを確認します。最後に、Espressoを使って、`onActivityResult`の結果を正しく反映されているかを確認します。

これで、テスト完了です😃
2つのActivityに関連するonActivityResultのテストが無事に出来ました！！

## 補足

Espressoにはご存知、clickをするためのAPIがあるのですが、うまく動きませんでした。

```kotlin
// not working!!
Espresso
  .onView(ViewMatchers.withId(R.id.button))
  .perform(ViewActions.click())
```

調べたんですが、原因がわかりませんでした😂分かり次第追記します。

## まとめ

- 上記のテストくらいなら、ユニットテストで書ける。すごい😃
  - onActivityResultみたいな、クラス間のつながりが弱い部分は意図せず壊れやすいので、テストを書いておくと安心かも😋

今回の検証に用いたサンプルコードは[satoshun-android-example/Tests](https://github.com/satoshun-android-example/Tests/blob/master/app/src/test/java/com/github/satoshun/example/tests/lifecycle/MainActivityTest.kt)にあります。

もっと良い書き方を知っているよと言う人は教えて頂けるととても嬉しいです😃😃😃
