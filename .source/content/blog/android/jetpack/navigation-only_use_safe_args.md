+++
date = "2018-05-20T00:00:00Z"
title = "Android: Navigationのsafe args Gradle pluginだけを使うのは良いかもしれない"
tags = ["android", "jetpack", "navigation"]
blogimport = true
type = "post"
draft = true
+++

Google I/O 2018でJetpackが登場し、新たにNavigationライブラリが導入されました。
ざっくりと説明すると、画面の遷移の実装を助けるライブラリになっています。実装的には、FragmentTransactionを直接いじることがなくなるといったメリットがあります。

このライブラリの機能に、safe argsというものがあります。これが便利で、最初はこの機能だけを使うのもいいのでは? と思っているのでその紹介です。

## safe argsを使ってみる

一般的にFragmentに適当な値を渡すときは`Bundle`を通して渡します。

```kotlin
// 呼び出し側
val bundle = Bundle()
bundle.putInt("step", 10)

// 呼び出され側
val step = arguments?.getInt("step")
```

これの問題点としては、typesafeで無いところです。例えばリファクタリングなどで、片方の文字列を`"step2"`にしてしまうと、ランタイムエラーになります。

これを解決にするためにNavigationではsafe argsという機能を提供しています。
これは、DataBindingのように、クラスを生成することでtypesafeを実現します。

```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools">
  <fragment
      android:name="com.example.android.codelabs.navigation.HogeFragment"
      android:label="Hoge">

    <argument
        android:name="step"
        app:type="integer"
        android:defaultValue="1"/>
  </fragment>
</navigation>
```

とnavigationを記述すると、

```java
public class HogeFragmentArgs {
  private int step = 1;

  private HogeFragmentArgs() {
  }

  public static HogeFragmentArgs fromBundle(Bundle bundle) {
    HogeFragmentArgs result = new HogeFragmentArgs();
    if (bundle.containsKey("step")) {
      result.step = bundle.getInt("step");
    }
    return result;
  }

  public int getStep() {
    return step;
  }

  public Bundle toBundle() {
    Bundle __outBundle = new Bundle();
    __outBundle.putInt("step", this.step);
    return __outBundle;
  }

  public static class Builder {
    private int step = 1;

    public Builder(HogeFragmentArgs original) {
      this.step = original.step;
    }

    public Builder() {
    }

    public HogeFragmentArgs build() {
      HogeFragmentArgs result = new HogeFragmentArgs();
      result.step = this.step;
      return result;
    }

    public Builder setStep(int step) {
      this.step = step;
      return this;
    }

    public int getStep() {
      return step;
    }
  }
}
```

上記のクラスが生成されます。これを使うことでtypesafeを実現出来ます。

```kotlin
// 呼び出し側
HogeFragmentArgs.Builder()
    .setStep(10)
    .build()
    .toBunble()

// 呼び出され側
val args = HogeFragmentArgs.fromBundle(arguments)
```

## まとめ

Navigationは大きいライブラリなので、とりあえずの練習でsafe argsの機能だけつまみ食いするのは、
個人的には良さそうだと思っています。

## 参考

- https://github.com/googlecodelabs/android-navigation
