+++
date = "Mon Apr 15 13:17:53 UTC 2019"
title = "ConstraintLayoutのImageFilterViewって単体でも使えるんやなって"
tags = ["android", "constraintlayout"]
blogimport = true
type = "post"
draft = false
+++

ConstraintLayout 2.0.0-alphaから`ImageFilterView`クラスが追加されました。今まで、MotionLayoutと一緒に使うものだから、使い所限られそうだなぁ〜と思っていたのですが、単体でも使えそうだったので、その報告記事になります。

この記事ではConstraintLayout 2.0.0-alpha4を使っています。

## 角丸にする

`round`属性から指定する事ができます。

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

## 彩度

`saturation`属性から設定することが出来ます。

```xml
<androidx.constraintlayout.utils.widget.ImageFilterView
  android:id="@+id/image"
  android:layout_width="100dp"
  android:layout_height="100dp"
  app:layout_constraintStart_toStartOf="parent"
  app:layout_constraintTop_toTopOf="parent"
  app:round="0.5dp"
  app:saturation="0.1" />
```

<img src="https://lh3.googleusercontent.com/wCPl0zHaivuj-PtkvE-HrvkIgFWqBwlz5Gr614mGfxvQB4U7IsvIbMuQVI8sK1RRtgRJwctepaxU8rnOuY1MATEXZq3S3NoNqNdcU4ULpRFRYGRaFcWpWtbd70YBYB7vOkH9Qx1hEt6qkXIKn-olrNDjjro9LlxEs0SjkvtxsyjQOgf3V0hlKH_-0ip6Zy0zk_zeWMiworGc61jHd3E4NfjgdoiCR8Xjg3qtjDWLobHAPT7HGTNWBC6gv0-vkz_xHxM5FZyviTAvLkgDkkHeJh3m-YSbRBDFIZ1C4-LIqt67gABi7sgfT7w6A5r_RSNTUqkTZvRd7ZUx1LtM6SPcpDSTCs-rZFSpYKkU4suecLswHOKDLMUh1EG_K_h_mSbCZgdFEQPhvr5hP9p-EN8WMMuMdJZjpPtxi1Hw6gY0fw6iuHMHJaVb4Kbjyfi5NeJVFZnf6-WBdoieQX438Spo7weidl7HD1mVGtIbGdb30h3FDtjHUoFl6kHM3myZxWq9DN83xHlsvPOQY-7TOPWbqBOWxLy8hhbxA3Dz7qhz-x0O1O8fBrYEpDQKJprdLNhwsy1s_sH37YBP0wadBUchOGA2f0LS_bLTKLrkE2oJx2XS38yCAbfGi3QW411W_Hg6S6DDYuYsrPRTxC2bw0BmCjadyxV3AOmX=w1080-h502-no" width=400 />

## コントラスト

`contrast`属性から設定することが出来ます。

```xml
<androidx.constraintlayout.utils.widget.ImageFilterView
  android:id="@+id/image"
  android:layout_width="100dp"
  android:layout_height="100dp"
  app:contrast="0.5"
  app:layout_constraintStart_toStartOf="parent"
  app:layout_constraintTop_toTopOf="parent"
  app:round="0.5dp" />
```

<img src="https://lh3.googleusercontent.com/wnE8okXNQjzyLwlW0wHmGZreCJI1-BjgPT_ofryDbrDC1Zfz1qD3gTpGKPhCSKYpkPXRkkbpbBuGHccTxqgJG7ywcdpuWXfgIY71nquRVKmGT5YdoojjC18cQj3fRdiEQyXw2t-pXD-Pe-wBqsTrFJ6ZAk25RakjtTF8vrPMH6PvC8InmBGx1HaUWKkYO1x3umMz9ORGMJB0T9qZ8KFspUhMW05QlmiVQXrbqDD50OCXrP_HTG8ACwW1CL5XuMYAmgP6sH24zm48cYY3ABbxVQFyLOkPKx-dyFzQ1pqjzwOncnZZqvTHp6ykd3kzJwd6PYoyX2-3HMPtDtTjW2GJNUTo23MLZmEqTwDLEZP7ibAmcoivYw6IpSoa05AcT_aLD-fjEDtpZIGCiQDrVjxb32fBgaA6h6nqjbRClMN0ZpChFxsJOAP9zgC_SkAHBeGpAczvNfyPxOzR29-eFGf9q4p59axmdyFIls_y7h3W3vPEvZK_JYFe38avYSvru4C1HtU9CRslj2jvahdNN__ZblA7wQznj-MUnIdX5E1KfIP0vsiy1kFm7tCHLWyh5FZmf0sxxTCyQToL366hbXvgPsu-e4LTwuP3S1saYOZe2WVClG6cTiv98nqC8WxWO_FtosHZaY6Davi43S_VeO0kTsUn4NPDx4H5=w1080-h516-no" width=400 />

## 暖色、寒色

`warmth`属性から、暖色、寒色？を設定することできます。

```xml
<androidx.constraintlayout.utils.widget.ImageFilterView
  android:id="@+id/image"
  android:layout_width="100dp"
  android:layout_height="100dp"
  app:layout_constraintStart_toStartOf="parent"
  app:layout_constraintTop_toTopOf="parent"
  app:round="0.5dp"
  app:warmth="2" />
```

<img src="https://lh3.googleusercontent.com/a0aHhdG12uZoHpvFU8nCsxDwlnJt-ICgmdl2QhQi8GgeEifaDtLzM-xmwI8DmyB-Vqnii-ZQ4QEAvnUxqwef5P--45HBrSIFbRTT98XcfntFmaZYeduDAdNoWCMDTVXdz97uUfYewOF4p0fnKAvUbHoqjXFTdKU0lVLyTHUyyuNma6vx1qw9LE_Kri7xqevLstb6f5gvbCWsVmNuo_J-FlW133j7PK89zd-JbwYHf4B9L1tvo80Lo2eMKLjLvNyU26xFA0f88BBJeeT-Wh6IWm1_gTnEqBQlE9xrJvGTI93VjVE8SBYXSnmUN7Q9Qm5ZpY-dZLRr9H1Bvqju8O7nNcWPWdtLBLxyirDJ1fZlOJWqqXvzw1l4vncpPBNM-WfyCaY7TIxxTOD2Czvl8G9fNFs9ZQCehmvildHx088tpkB9IgMFanB9j1IZjzoP_c0ipaw_ayBR69dNNSUWVmWp5xlPCEjNXA7xi_JfruaHxhvMmocOkNKhQVxJ39BQv3KVWFIYHmZdDxnf7ZKj6iZ56V8eyb9E3R25V0_rljkAHLEC7cEEFjs9H7h7GDFAh71zbybA7bqauWB0VjRBuHDQ2eSpwKRSJh3wqLpArzKbjOkjiP9RNwGeI-CWcyQl7HCileL11hSbLem3GINlUkTLPiVoeSnMl0yG=w1080-h510-no" width=400 />

---

```xml
<androidx.constraintlayout.utils.widget.ImageFilterView
  android:id="@+id/image"
  android:layout_width="100dp"
  android:layout_height="100dp"
  app:layout_constraintStart_toStartOf="parent"
  app:layout_constraintTop_toTopOf="parent"
  app:round="0.5dp"
  app:warmth="0.5" />
```

<img src="https://lh3.googleusercontent.com/Gac2-_7RbmYctobF_eztsBTlTGVNMNHor3tJYk-awVNB0Bz9rAiTfb_jkwLMd3JuKPJZd1B8tz4UDfITyw_2rUBAM2ji0bt5FYPvm9kebLZ0I1Wujn4-Ovv51XPdQ_61BNNfnPbdIztP408lWkPpkTJlbKe7JZcrjG-Ia2RZgnUo0z77LbPO56VXFlQSR9JvN8KEzzs4n2VaE0xmA_LBQ2elyirTzYWDlWiUs__MuD0e5JoqjXxfn6HM-ORGPb--PZeCFaUKUyizAxI7Vd_C27vGwTBEFwE3ITF-bT5e1HQW2J1QwrwygWnDMUnZBaoaHGjMDmKAs-nPRmqFGCrNSKYWJs6yKEQN4CEkj_Kk4E5ldCioCZiQIK4RtsjtEuQT1_-WOE_AGSh0eSUzbIFkDG5rofk-9c_e06-S8Nba-t8r8U7h0dVNOGMmmqmU5P3gcomoZUDJVZ6HJi_Q2mVUVsMQlZA6DKSwMRlivlvTHI1KbpyGxIBPPQrfnFp7TzLdh3f3MmPWXT3e3a47UQlnOxLPH6h1ZcMPoHXf8LTQ8y2duu9xzQ1gyxvYS8_o8Rit512VGxqCdcyt1J5y5i3Suq4fq7eezfg2tdjwR64GUjZbWEbWn57KNSiZIPmrQ2GC9p1m_GYAYg38KAjRHUDNvhLrJS4lT7sA=w1080-h513-no" width=400 />

## クロスフェード

`crossfade`属性から設定することが出来ます。ただ、この属性はMotionLayoutと一緒に使うもので、単体では使わないと思います。

## まとめ

- `ImageFilterView`、単体でも結構使いどころあるかも😃
