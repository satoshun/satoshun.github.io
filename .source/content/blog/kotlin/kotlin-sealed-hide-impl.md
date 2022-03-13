+++
date = "Sun Mar 13 03:33:43 UTC 2022"
title = "Kotlin: Sealed class(interface)で実装を隠す"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = false
+++

ライブラリを作る時など、クライアントから実装を隠したいケースはままあります。

例えば、次のコードの実装を隠してみます。

```kotlin
class CameraController(...)

fun createCameraController(): CameraController {
  ...
}
```

この場合、実装であるCameraControllerが参照されています。

例えば次のようにして、実装を隠してみます。

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

実装をクライアント側で作って欲しくない場合には、interfaceやabstract classの代わりに、sealed classを使えば達成できます。

ライブラリ開発などで、実装を見せたくないが、ライブラリ側でしか実装しないようなインターフェースの場合に、使うことができるテクニックです。
