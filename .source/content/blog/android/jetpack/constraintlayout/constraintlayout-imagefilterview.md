+++
date = "Mon Apr 15 00:37:20 UTC 2019"
title = "ConstraintLayoutのImageFilterViewって単体でも使えるんやなって"
tags = ["android", "constraintlayout"]
blogimport = true
type = "post"
draft = true
+++

ConstraintLayout 2.0-alphaから`ImageFilterView`クラスが追加されました。今まで、MotionLayoutと一緒に使うものだから、使い所限られそうだなぁ〜と思っていたのですが、単体でも使えそうだったので、その報告記事になります。

##　角丸にする

`round`属性を指定すると角丸に出来ます。

```xml
<androidx.constraintlayout.utils.widget.ImageFilterView
  android:id="@+id/image"
  android:layout_width="100dp"
  android:layout_height="100dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintEnd_toEndOf="parent"
  app:layout_constraintStart_toStartOf="parent"
  app:layout_constraintTop_toTopOf="parent"
  app:round="0.5dp" />
```

<img src="https://lh3.googleusercontent.com/w0ztp5sD9OXMBfDSR_hocSPM08c9eHvIQTHCSpZGbyQF7ci0lcS265HwEAPHRnvY4Q0ABX5soYO8nIB_3wSpdiDek1N2F1X1m6fkDOevgydQ5IXXyVDVMqriB_Odu-szOHdfZIJ0YCLYUEOkXNEwDc8kRgooA3eeHJMqxl3urpG4siDCaJqsnhkP4MfKH3gtqJlsw6ol7Hd27L1eyexPod5mnOz2edWIS12Ogf9yIaWkVU2or1yyoHqUlsqr8xPyChg1gNYQ1cTzwBD9u1_xewQzJyGCn4ae3jqg4CnJ-L6VmJ08KrRVA9xWGYQ14u_r6uu-dD48sLdb28XuQ0egxLC-KhjxkIKHR3cCnlz-orcZ7AeKFMHwuG74fPAXXZ4Pyry2UmYDFSTXCL0StUi7h3wbhETX8u_-e9vIePPSn-1ZTLGGaLfbJTn4Vp8gZqrU7mWjoUVx0tzNlljfc47SJCN7RfFddWlLQFrtjQc7mGCmj9GLFZcJcRrKxLzp7yAOQo4DxWAY6PMcN-LNup3v4uH8PtUW9r9I2KqbmtW8SaBb9EJOMHlGsbcFtughox_G1d7bBUpZeqTnteU3bUldjiUywQTA6lJEbH68mjpoLMWkGmGjVJN9cmszjKCvd1LYcaJq89HCmskyfCwlMtiNItgP6zFCY7M7uQcmD5d62VGnlXHDXKvsYfhHjC5WtwUmoWzIbBzwD8F3YEdfDlxL6Tq0uA=w1067-h530-no" width=400 />


## まとめ
