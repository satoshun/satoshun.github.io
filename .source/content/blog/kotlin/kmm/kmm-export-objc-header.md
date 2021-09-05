+++
date = "Sun Sep  5 03:00:44 UTC 2021"
title = "KMMでアクセス修飾子をつけるとobjcヘッダーファイルが小さくなって嬉しい話"
tags = ["kmm", "kotlin"]
blogimport = true
type = "post"
draft = false
+++

KMMで作ったライブラリはobjcヘッダーを生成し、それを介してSwift(Xcode)側から読み込むことが出来ます。

Kotlin側で、internalとか、private等のアクセス修飾子をつけることで、objcヘッダーを小さくすることが出来ます。

## つけない場合

ktorのHttpClientを例にして説明します。

コンストラクタにアクセス修飾子をつけない、次のようなクラスがあるとします。

```kotlin
import io.ktor.client.HttpClient

class Test1(
    private val httpClient: HttpClient
)
```

これは、次のようなヘッダーファイルを生成します。(一部省略)

```text
...

public class Test1 : KotlinBase {
    public init(httpClient: Ktor_client_coreHttpClient)
}


public protocol Ktor_ioCloseable {


    func close()
}

public class Ktor_client_coreHttpClient : KotlinBase, Kotlinx_coroutines_coreCoroutineScope, Ktor_ioCloseable {
    ...
}

...
```

HttpClientに関連したクラスをobjcヘッダーに公開しています。
ktorは大きいライブラリなので、objcヘッダーがとても大きくなります。

最終的に、2143行のobjcヘッダーになっていました。

## つける場合

次に、アクセス修飾子をつけてみます。

```kotlin
import io.ktor.client.HttpClient

class Test2 internal constructor(
    private val httpClient: HttpClient
)
```

これは次のようなヘッダーファイルを生成します。

```text
...

public class Test2 : KotlinBase {
}
```

internalをつけたので、HttpClientに関連したクラスをobjcヘッダーに公開せずに済みます。

最終的に、112行のobjcヘッダーになっていました。つけない場合に比べると差は歴然です。

## まとめ

internalとか、privateとかをつけて、Swift側から見れないようにすると、objcヘッダーファイルが小さくなって少し幸せになります。
