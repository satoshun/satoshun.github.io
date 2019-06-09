+++
date = "Sun Jun  9 05:02:05 UTC 2019"
title = "IO19: Build a Modular Android App Architectureのまとめ/感想"
tags = ["android", "arch", "io19"]
blogimport = true
type = "post"
draft = true
+++

## なぜモジュール化をするか?

### スケール

モジュール化することで、開発者が独立して開発出来るようになる
- 人数が増えてきた時に、モジュール化は有効

### 保守性

モノリシックアプリだとレイアウトファイルを1つのディレクトリに持つことになる
- パット見で何をしているのかが分かりにくい
  - 長いレイアウトファイル名になりがち

### ビルド時間の短縮

変更があったモジュール + 依存関係にあるモジュールがビルドされるため、各モジュールが小さければビルド時間が早くなる

### CIの高速化

再ビルドが必要なモジュールのみテストをすれば良い
- 全モジュールをテストする必要がない
- [dependencyTracker](https://android.googlesource.com/platform/frameworks/support/+/androidx-master-dev/buildSrc/src/main/kotlin/androidx/build/dependencyTracker/)

### Smaller APKs

App Bundle、Dynamic Deliveryの恩恵を受けられる


## モジュール

どのようにモジュール分けをするか?

- Feature（機能）ごと
    - ライブラリモジュール
    - Dynamic Featureモジュール
        - onDemand true
            - Paidのような一部のユーザが使う機能
        - onDemand false
            - Onboardingのように、後でいらなくなる機能
    - Plaidでは以下のようにした

<img src="https://lh3.googleusercontent.com/qwNff93mQNWP8E8yIOdnxHlB7VxeyatfnF6mB5UV8OM79kqVVVy4bH1syJsrv3Y2ABqIkebCB2ASKv1-vyLt0dPas4mIbO9CkTt1CZ7wUJ78nPRp35B0guWwfdZ0B3qEtge5wTLi-tCVpT2akRMPHBHV34dGIJ1kiI-PUiRIhy_NXtz4LCyrk5ib1AeXa2K0DPkCW4GLF-IfFMbNrffNOjy7YG1_8CUBRQplKTKrk2s_6F7keIBlgfPCk8i_ZwOImb7S-6SHMKtPF5gAjZtaSKeDkDee7-otF9ca671scd8gRoZteWpBtZBdbYcarckAZB3Kr8b2ZW187r_CSwZIyC17TdKoAu5z6lgaEoOE_dR-XAn5mnrIY8MrjqqF3o6muIL2kZZ2zd3dmyJvb3PopvKSb2H8UvMg3nyIQifrVii3RWMebioPyvdbiB8sRfLtsYHaF3X_2gj8FMk3YayCGY99FgGUsyOngHJgthm8CE7lFU6GavE008tozIL6HKKIDgs1kPJ3RlpDwNCcBG__lEUNCoZqbsQjN0Wo6lv3URK7xv3ZOzj-eBpaeN6oZQCs6OK7W6SEtZ5-vlP8CYxCMpxSqIjI1cepS6NOO3-IjcCTIQA7JxYjkPswTCIOi6Ur5tU56GMSR9wXN08ePlt3ELR1_CRsKA3e=w720-h404-no" width=500 />

- Layer（層、階層）ごと
