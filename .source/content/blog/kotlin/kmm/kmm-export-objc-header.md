+++
date = "Sun Sep  5 03:00:44 UTC 2021"
title = "KMMでアクセス修飾子をつけるとobjcヘッダーファイルが小さくなって嬉しい話"
tags = ["kmm", "kotlin"]
blogimport = true
type = "post"
draft = true
+++

KMMで作ったライブラリはobjcヘッダーを生成し、それを介してSwift(Xcode)側から読み込むことが出来ます。

Kotlin側では、極力internalとか、private等のアクセス修飾子をつけることで、objcヘッダーを小さくすることが出来ます。

## つけない場合

ktorのHttpClientを例にして説明します。

アクセス修飾子をつけない場合は、次のようなヘッダーファイルを生成します。

```kotlin
import io.ktor.client.HttpClient

class Test1(
    private val httpClient: HttpClient
)
```

```text
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

HttpClientに関連したクラスをobjcヘッダーに公開するので、objcヘッダーがとても大きくなります。
2143行のobjcヘッダーになっていました。

## つける場合

次に、アクセス修飾子をつけてみます。次のようなヘッダーファイルを生成します。


```kotlin
import io.ktor.client.HttpClient

class Test2 internal constructor(
    private val httpClient: HttpClient
)
```

```text
public class Test2 : KotlinBase {
}
```

internalをつけたので、HttpClientに関連したクラスをobjcヘッダーに公開せずに済みます。
112行のobjcヘッダーになっていました。

## まとめ

internalとか、privateとかをつけて、Swift側から見れないようにすると、objcヘッダーファイルが小さくなって少し幸せになります。
