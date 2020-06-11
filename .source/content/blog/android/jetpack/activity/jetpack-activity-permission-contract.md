+++
date = "Thu Jun 11 10:28:26 UTC 2020"
title = "Android: ActivityResultContractを使ってRuntime Permissionsを実装する"
tags = ["android", "jetpack", "activity"]
blogimport = true
type = "post"
draft = true
+++

Activity 1.2.0-alpha02から導入された、ActivityResultContractを使うことで、Runtime Permissionsをいい感じに実装することが可能になりました。

## 書き方

`android.permission.ACCESS_FINE_LOCATION`が欲しい時は、次のように書くことが出来ます。

```kotlin
class HogeActivity : AppCompatActivity() {

  // 定義側
  private val requestLocation = registerForActivityResult(
    ActivityResultContracts.RequestPermission(),
    ACCESS_FINE_LOCATION
  ) { isGranted ->
    // Permissionの取得に成功したかどうかがBoolean値で返ってくる
    Toast.makeText(this@AppActivity, "isGranted $isGranted", Toast.LENGTH_LONG).show()
  }

  private fun hoge() {
    // 呼び出し側
    requestLocation.launch(Unit)
  }
}
```

単一のPermissionの場合、このように書くことが出来ます。

---

また、複数のPermissonが欲しい時は次のように書くことが出来ます。

```kotlin
class HogeActivity : AppCompatActivity(R.layout.app_act) {
  // 定義側
  private val requestPermissions = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions(),
    arrayOf(ACCESS_FINE_LOCATION, READ_EXTERNAL_STORAGE
  ) { grants ->
    // Permissionの取得に成功したかどうかがMap<String, Boolean>>で返ってくる
    Toast.makeText(this@AppActivity, "grants $grants", Toast.LENGTH_LONG).show()
  }

  private fun hoge() {
    // 呼び出し側
    requestPermissions.launch(Unit)
  }
}
```

## まとめ

ActivityResultContract API便利〜
