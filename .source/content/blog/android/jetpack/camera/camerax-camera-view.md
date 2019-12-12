+++
date = "Mon May  6 05:47:14 UTC 2019"
title = "CameraX: CameraView触ってみた"
tags = ["android", "jetpack", "camerax"]
blogimport = true
type = "post"
draft = false
+++

CameraXのコードが公開されていたので、その中にあったCameraViewを触ってみました。まだ、alphaであることからAPIは大きく変わる可能性があります。

内部の実装であったり、細かい部分はpsideさんの「[CameraXのコードがきたので気合い入れて読んでみた](https://p-side.net/posts/2019-05-05-camerax/)」が詳しいです。

## 環境構築

CameraViewはまだ公開されていないため、ソースコードからビルドする必要があります。また、設定でpublishフラグがfalseになっているので、trueにしてビルドします。

```
 androidx {
     name = "Jetpack Camera View Library"
-    publish = false
+    publish = true
     mavenVersion = LibraryVersions.CAMERA
     mavenGroup = LibraryGroups.CAMERA
     inceptionYear = "2019"
 }
```

## 使い方

CameraViewは普通のViewのように使うことができます。

```xml
<androidx.constraintlayout.widget.ConstraintLayout
  ...>

  <androidx.camera.view.CameraView
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

次に初期化をします。

```kotlin
cameraView.bindToLifecycle(this) // thisはLifecycleOwner
```

LifecycleOwnerとCameraViewを結びつけることでLifecycleに合わせて自動でリソースを調整してくれます。
CameraViewはLifecycle-Aware Componentとなっています。非常に扱いやすそうです。

これだけでカメラ機能が使えるようになります！とても簡単！!

以下、CameraViewで現状使えるAPIについて紹介します。

## CameraViewで使えるAPI達

### モードの変更

CameraViewでは次の3つのモードがあります。

- Image: 写真を取る
- Video: ビデオを取る
- Mixed: 両方。ただし、動かない端末もあるらしい

次のように使います。

```kotlin
// Image
cameraView.captureMode = CameraView.CaptureMode.IMAGE
cameraView.takePicture(...)

// Video
cameraView.captureMode = CameraView.CaptureMode.VIDEO
cameraView.startRecording(...)
cameraView.stopRecording()
```

### ScaleTypeの設定

Scaleを設定することができます。

- CENTER_CROP
- CENTER_INSIDE

次のように使います。

```kotlin
cameraView.scaleType = CameraView.ScaleType.CENTER_INSIDE
```

### LensFacingの切り替え

```kotlin
cameraView.toggleCamera()

// or

cameraView.setCameraByLensFacing(CameraX.LensFacing.FRONT)
cameraView.setCameraByLensFacing(CameraX.LensFacing.BACK)
```

### Flashモードの設定

撮影時にフラッシュを有効にすることができます。

```kotlin
cameraView.flash = FlashMode.ON
cameraView.flash = FlashMode.OFF
```

### torchの有効/無効

背面ライトを有効にすることができます。

```kotlin
cameraView.enableTorch(true)
cameraView.enableTorch(false)
```

### zoomの設定

コードから変えることができます。

```kotlin
cameraView.zoomLevel = 3f
cameraView.zoomLevel = 10f
```

また、CameraViewはpinch-in-outでのzoomの変更にも対応しています。

### focus

指定した部分にfocusを合わせることができます。

```kotlin
cameraView.focus(...)
```

## まとめ

- CameraViewは柔軟性がない代わりに、簡単にカメラの機能を使うことが出来る
  - とはいえ、基本的な機能は揃っていそう

最後に、CameraViewを実装した結果になります。

{{< figure src="https://lh3.googleusercontent.com/2_s_ijtuGJSuwaJlLbsBXZ1QMTrJDjaqCM4I4ZmJQexItcrj9TgHIsh7g_VgR0nhTf-kYZ00kRDILJEsfjkd57CDK_f-d835KBsdaYdJQ0w55gsA1iCze5mC5Sm_HxDmtgHT7Asm8RUjPmYgxuI22TmFuEhP0gzxqR4ZPAroobBb0itTuZZX2Gi3X7JMfCm31wNMYSUleaBBwm9X7V3edWVCERxyLXUSoTs4ewZ-J05OMozhe0r6Wx1TsJx_5-wc4k4yrex08x4osdIiZVGJHI4W_hrlcIBL131nwHzm29djjrYLiaeb-UnKnP1kdum0NPEDIMQyvCgs3R3BR_BjoeqhLV6CadhPT2PH-AFkH4jXDKXbCC-BNRmJFbeubUtg6ATOCSXNV_Zc0i8qjzqzQFQ-kHlknUgn592eQECQYSNxS4m340CXqy_xP0DnU-o6WjE5ay4gJuYH8kR1eGflOG8sgyvoNlxaFmIXZpreBQJaXfqImK9tncglYIwVQnxEL8uMxZ_31Rc2--SVsKNCDCl625teKj28EaedQ0lzjvXlFAge4OKz-dMmMRiz12TTIfq6TFGB3TPLmjtpV1Su6VbQ0peovn6AvKgAOW2XM7JxVUmZU20GI4jtKyBMGppf4hsTY624sFnaSwoBr_7RshaOZ6f4JFykCBE4P2AvC549faCde3pnGhsQ2_ZFwA7ESpMN1hsgI7gUjFzaeSc0iLYVIw=w600-h1067-no" width="500" >}}
