+++
date = "Tue Mar 19 12:07:37 UTC 2019"
title = "Kotlin: Sealed class(interface)で実装を隠す"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = true
+++

ライブラリを作る時など、クライアントから実装を隠したいケースはままあります。

例えば、次のコードの実装を隠してみます。

```kotlin
class CameraController(...)

fun createCameraController(): CameraController {
  ...
}
```

この場合、実装であるCameraControllerが参照できます。
これを隠すために、例えば次のようにします。

```kotlin
interface CameraController { ... }

internal class CameraControllerImpl(...): CameraController

fun createCameraController(): CameraController {
  ...
}
```

これで実装を隠すことが出来ました。

さらにKotlinでは、ここから先に進めて、sealedを使うことで、外で実装を作ることを禁止することが出来ます。

```kotlin
sealed interface CameraController { ... }

internal class CameraControllerImpl(...): CameraController

fun createCameraController(): CameraController {
  ...
}
```

sealedの第一用途として、when式における網羅性(exhaustive)が挙げられますが、実装を隠したい + 実装をクライアント側で作って欲しくない場合にも使うことが出来ます。
