+++
date = "Thu May 16 00:33:48 UTC 2019"
title = "Data Binding 3.5と3.6のまとめ/感想"
tags = ["android", "databinding", "jetpack", "io19"]
blogimport = true
type = "post"
draft = false
+++

Google I/O 2019でDataBindingについて少し話されていたので感想とまとめを。
動画だと[ここらへん](https://youtu.be/Qxj2eBmXLHg?t=243)になります。

## 改善系

### ビルドの高速化!

- 20%くらいビルドが早くなった
- distributed build cache対応
- Incremental annotation processing対応
    - `android.databinding.incremental=true` を設定にすると使えます

### Android Studioとの連携強化!!

- Live Class Generation
    - XMLを変更したら、コンパイルせずともクラス、フィールドにアクセスすることが出来る
- リファクタリング系
    - フィールド名の変更リファクタリングなどをしたときに、XML、コードの両方に反映される

### Errorメッセージの改善!!!

- DataBinding用のエラーセクションが出来たことで、どこでエラーが出たか特定しやすくなった

{{< figure src="https://lh3.googleusercontent.com/9sMrM9zLYG96Vg2Mlt7KON2audHhb2hPcryUDkkp9E0s2Q2oVKd94EsoxxvxZwhAfkORXabT2-V2lnxDlEF7gMoTWMmJ0AmIPjyp-o54bFdqoN7u4KQlNqwz6ufqT8xKx7JvKJoJc-cyaH-3IOujqq-m7V0h5QYeN1W2JAeon_fDv-vqPRqzHydPvW2zv8NEuboWPOCQFSqABPj61OEI0kLstUwClb6E6_1CAjtenJfmhUjG2_gBbNakPao6BnSb4-1_BHeZnpkUMPzWyip4I0bRtnAIZcCjt797Sk90o8e87H0y_JdGUu-crJ2LVL5QvJ8Sz2WKkFcs1AxaLnn4PfRYzooeh70hJAjFPXuR9tkonL5PjyT96EqgOhmQlmzmXaj6NEwYjmOc7VE8KDFXSvLUm1K5GlyaU1j8vUWhnNKvPPpfFn26XnIcTD0j2-YE_RhR4nP40LNS1XxCsFOJBpOWZCK69JwAp3flmULDQpmkJkeSp99T_suxdxlRDrPZnTb4gNAs6Bd0IMsalfJpOQWpAPQebqNkuan_BjytvzQl3sR0KPi73AafkfzcNQbPjsMXxMJNqbyMaxZFwDIIw5WLLm7GoOsU8PX8d1sadYbwTHVYpx9M_nqTcBLns-henpEBj89wDXy3tXg-xF2DyGNScK2blikn=w2160-h1620-no" width="600" >}}

## 新規系

### View Binding

簡易版Data Bindingのような立ち位置で、findViewByIdを省略 + コンパイルセーフ + コンパイルを高速にすることを目的に作られました。3.6で入るみたいです。

{{< figure src="https://lh3.googleusercontent.com/WfZyK356KdBHf5gHL1PExSKTXXo5_lEKhsvoDhrlRgSWH40Qa4Y6hhw_08w3Wm46KAoFAf26_cAhtDIzDja3OKxf7cVTju9UwNp-JAvaTDJrlI2gWHSmTkePpGB8Vl8pxzdmAcjuuJ1EzUW_h67sFEULzQE-a1ro96x03jAC5FCjWSArhx2f5jz9cqSPdhieW_zk1glrdLqHUogvoeJCoLbtaDy2KFnwQkwJDWPkWDfBs3Q2CxrkI6-Fom1fmpHIFMZ4NjUVU4TKfbDqYWCALnvB5G17HXy0YvFjYnSDLtWn4nUMH2rZWnh0R9JJU5fIVBDtncuZkqhp-AZUJBrInCsvV-8vIaUeBr6ZoUPTd9Ja4qz0ooODr7VOJDGBFPP4qHpNIzQwxWzCgYlsRU4l8i9dxMCKq0bLfTxZcIGBYYm-Bed8AGiUyPnsS_7HTMkuWu_NPGn-cTOTIFJ6n9vSdm2uR0IIJWXW8fS6s_q0S9eSZJYW6zeJBXOxdP4ot2FSaZDL8pmwbxivfaWTA_R3XraQizWG788EvVkpFvF2Xree28fR6qKP1zlKS2mSrooQCsQ-xNkoLWnTkNkLC8j9USiSRFrhDl0Mu9LmJckzPSq52FmaX8ClbXt8_H2vML3tmwqY8FQ92ZTmP9WNHDFr12jP96uX_3jKsraXa8btthUlzCHe58dIWeSlKU7GOqh1hr8k4GGcxZYvJVzjydsGIY-mqQ=w2160-h1620-no" width="600" >}}

Data Bindingと比較したときの、メリット、デメリット以下になります。（ただし、まだalphaも出ていない段階なのでI/O動画から見る限りの感想です）

#### メリット

- コンパイルが早くなる
    - Data Bindingより機能が少なくなるのでそれはそう
- `<layout>`で囲う必要がなくなった
    - これ個人的には好きでなかったので嬉しい。ネストが減る

#### デメリット

- `<data>`セクションがなくなる
    - モデルの値とViewのマッピングはコード側ですることになりそう
- 多分BindingAdapterは使えない
    - これもコード側ですることになりそう
- 双方向バインディングとか使えない

おそらくなんですけど、Data Binding、View Bindingは1つにプロジェクトに混在させることが出来るので、基本View Bindingで、双方向使いたいときはData Bindingみたいな使い方も出来るはずです。

## まとめ/感想

- 今、3.5-beta01を使っているんですが、肌感、かなり良くなっています😃
    - Live Class Generation便利すぎワロリン
- View Bindingはとても良さそう
    - Kotlin syntheticの代わりに使ってもよさそう
