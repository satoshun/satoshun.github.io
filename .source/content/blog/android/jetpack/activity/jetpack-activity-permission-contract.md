+++
date = "Thu Jun 11 10:47:24 UTC 2020"
lastmod = "Fri Jun 12 13:28:24 UTC 2020"
title = "Android: ActivityResultContractを使ってRuntime Permissionsを実装する"
tags = ["android", "jetpack", "activity"]
blogimport = true
type = "post"
draft = false
+++

Activity 1.2.0-alpha02から導入された、ActivityResultContractを使うことで、Runtime Permissionsをいい感じに実装することが可能になりました。

この記事では、どのように実装すれば良いかを簡単に説明します。

## 単一のPermissonを要求する

新しく追加された `registerForActivityResult` 拡張関数に、`ActivityResultContracts.RequestPermission()`を指定することで、単一のPermissionを要求することが出来ます。

例えば、`android.permission.ACCESS_FINE_LOCATION`が欲しい時は、次のように書くことが出来ます。

```kotlin
class HogeActivity : AppCompatActivity() {

  // 定義側
  private val requestLocation = registerForActivityResult(
    ActivityResultContracts.RequestPermission(),
    ACCESS_FINE_LOCATION
  ) { isGranted ->
    // Permissionの取得に成功したかどうか、Boolean値で返ってくる
    Toast.makeText(this@AppActivity, "isGranted $isGranted", Toast.LENGTH_LONG).show()
  }

  private fun hoge() {
    // 呼び出し側
    requestLocation.launch(Unit)
  }
}
```

## 複数のPermissonを要求する

複数の場合は、`ActivityResultContracts.RequestMultiplePermissions()`を指定します。

例えば、`android.permission.ACCESS_FINE_LOCATION`、`android.permission.READ_EXTERNAL_STORAGE`が欲しい時は、次のように書くことが出来ます。

```kotlin
class HogeActivity : AppCompatActivity(R.layout.app_act) {
  // 定義側
  private val requestPermissions = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions(),
    arrayOf(ACCESS_FINE_LOCATION, READ_EXTERNAL_STORAGE
  ) { grants ->
    // Permissionの取得に成功したかどうか、Map<String, Boolean>>で返ってくる
    Toast.makeText(this@AppActivity, "grants $grants", Toast.LENGTH_LONG).show()
  }

  private fun hoge() {
    // 呼び出し側
    requestPermissions.launch(Unit)
  }
}
```

## まとめ

ActivityResultContract便利〜
