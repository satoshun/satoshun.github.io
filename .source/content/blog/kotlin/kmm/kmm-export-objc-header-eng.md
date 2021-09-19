+++
date = "Sun Sep  5 03:00:44 UTC 2021"
title = "Make the objc header smaller in KMM"
tags = ["kmm", "kotlin"]
blogimport = true
type = "post"
draft = true
+++

KMM make the objc header, it can be loaded from the Swift (XCode).

On the Kotlin side, we can reduce the size of the objc header by adding an access modifier such as internal or private.

## no access modifier

For Example, we use a Ktor HttpClient class.

We have a class that does not have an access modifier in its constructor.

```kotlin
import io.ktor.client.HttpClient

class Test1(
    private val httpClient: HttpClient
)
```


This will generate a following objc header.

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

It's related a ktor HttpClient and it expose a objc header.

This objc header file have 2143 lines.

## use access modifier

Next, We use a access modifier in constructor.

```kotlin
import io.ktor.client.HttpClient

class Test2 internal constructor(
    private val httpClient: HttpClient
)
```

This will generate a following objc header.


```text
...

public class Test2 : KotlinBase {
}
```

It's not related a ktor HttpClient!

This objc header file have 112 lines.

## summarize

We were able to reduce the number of objc headers by adding an access modifier.
