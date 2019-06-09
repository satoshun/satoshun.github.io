+++
date = "Sun Jun  9 12:04:27 UTC 2019"
title = "Google I/O'19: Build a Modular Android App Architectureのまとめ/感想"
tags = ["android", "architecture", "io19", "android-architecture"]
blogimport = true
type = "post"
draft = false
+++

以下の動画のまとめです。

[Build a Modular Android App Architecture (Google I/O'19)](https://www.youtube.com/watch?v=PZBg5DIzNww)

## なぜモジュール化をするか?

### スケール

モジュール化することで、開発者が独立して開発出来るようになる

- 人数が増えてきた時、アプリが大きくなってきた時にモジュール化は有効

### 保守性

例えば、モノリシックアプリだとレイアウトファイルを1つのディレクトリに持つことになる

- 何をしているのか、何をしたいのかが理解しにくい
- 長いレイアウトファイル名になりがち

### ビルド時間の短縮

変更があったモジュール + その依存関係にあるモジュールが再ビルドされるため、ビルド時間が短くなる

### CIの高速化

再ビルドが必要なモジュールのみテストをすれば良いので、テスト時間が短くなる

- [androidx/dependencyTracker](https://android.googlesource.com/platform/frameworks/support/+/androidx-master-dev/buildSrc/src/main/kotlin/androidx/build/dependencyTracker/)を使うといい感じにテストが出来る（らしい）

### APKサイズの縮小

App Bundle、Dynamic Deliveryの恩恵を受けられる


## モジュール

どのようにモジュール分けをするか?

### 1. Feature（機能）ごとに分ける

ライブラリモジュールとDynamic Featureモジュールの2種類がある。

- ライブラリモジュール
    - `com.android.library`を指定する
- Dynamic Featureモジュール
    - onDemand trueとfalseがある
        - Paidのような一部のユーザが使う機能の場合はtrueが良い
        - Onboardingのように、後でいらなくなる機能の場合はfalseが良い

Plaidでは以下のようなモジュール構成にした。

<img src="https://lh3.googleusercontent.com/qwNff93mQNWP8E8yIOdnxHlB7VxeyatfnF6mB5UV8OM79kqVVVy4bH1syJsrv3Y2ABqIkebCB2ASKv1-vyLt0dPas4mIbO9CkTt1CZ7wUJ78nPRp35B0guWwfdZ0B3qEtge5wTLi-tCVpT2akRMPHBHV34dGIJ1kiI-PUiRIhy_NXtz4LCyrk5ib1AeXa2K0DPkCW4GLF-IfFMbNrffNOjy7YG1_8CUBRQplKTKrk2s_6F7keIBlgfPCk8i_ZwOImb7S-6SHMKtPF5gAjZtaSKeDkDee7-otF9ca671scd8gRoZteWpBtZBdbYcarckAZB3Kr8b2ZW187r_CSwZIyC17TdKoAu5z6lgaEoOE_dR-XAn5mnrIY8MrjqqF3o6muIL2kZZ2zd3dmyJvb3PopvKSb2H8UvMg3nyIQifrVii3RWMebioPyvdbiB8sRfLtsYHaF3X_2gj8FMk3YayCGY99FgGUsyOngHJgthm8CE7lFU6GavE008tozIL6HKKIDgs1kPJ3RlpDwNCcBG__lEUNCoZqbsQjN0Wo6lv3URK7xv3ZOzj-eBpaeN6oZQCs6OK7W6SEtZ5-vlP8CYxCMpxSqIjI1cepS6NOO3-IjcCTIQA7JxYjkPswTCIOi6Ur5tU56GMSR9wXN08ePlt3ELR1_CRsKA3e=w720-h404-no" width=500 />

dribbleと、designernewsがDynamic Featureモジュールになっている。

### 2. Layer（層、階層）ごとに分ける

Plaidでは以下のように分けた。

<img src="https://lh3.googleusercontent.com/eKPc5MsRrRbOjCXfIJnVppLjpevSWsz4e13QVgu8wAmd1AI2An-aVSerQ1Wz0DuALe6RCP3JPHkkJh1mBZx-9N2lwJj2i4aBoCrIqztFu2CLyZXu6QZBEC5AuO5FzXD90hqMMV8J12g0Sn9WjVMUQHA4RxREdVaSmNwXRt0OBfIVG_4UYdnp8SDJJ8OOKPsDqBX3cOhopNBWEK2B6Of-VwkN25G3IRrV952EnleuGaFbsxrGXkGTv64oAbnB6aw71alRN4JO3HUbXlJY54XZOXtJV3wMHu1NWOqUF9sM5GsEtcMQn_fdKsfkynN3UIZ-QTRNI2PKefKKV7X-aPZp6Jg_OAm9WoGA-VktrPj9dQRpx1EPvT6av3KfChS_x0_oCdIeJpXwLBObooQMmRpiJzKJCb112Pnq4s3XhDOcK9n08kOQZgOHOOFx_jC-CjhUEMRmK3j-XQBNmmJjqLdzBksgpajJqpbl8QS7B6snoEXPRa7AoVIfhPNRQurFi5880dhTx0JrFYEm1yeHzHYTHqW7bXpjjtAYWW5MQJ6HDwSOpphlZiR1z6AnJK3DTiM5K7Q1-p27e802TcjF8O3yhFq4XDLl6IqZ_99TFoxcNcocxFCd2FlYcBcD1lOsXcIwpXOMRDapFTv7Tx8UBhzeeijqGc512lTI=w720-h402-no" width=500 />

<img src="https://lh3.googleusercontent.com/_81GbLd7eZJQvhnKKjlzv08wTjRKtzqt_IZPQNf1BlYcUmCvBv1B4kD6Pjw5b0MfQfPMke62kIzaXW7AjRON695kLYHQKHkk_-03L7IynQHKNggkWh9L5BZWafVz2nEz4f8Bn_mFDjuo2FT8iK6Ho4HI8L3d67qQh7N-Ljj-5G7glOklHSEQMGLOPzLRr7bH_b_jogHxLZS48_hWkeeFLqsBFscsiPXMjuyz5mjfCv9z-ZgvKa45LL8vkqe6jMWaBH2bGhkMJChHCD8fMbPaVDN-eNaSq5hvU6XcmAexyYMqM72exG2fuzxj2aGDPFCtZ3v9_8g5RMHVTb4wG4yG87epXC2XDRG9TJ3P6hAUl4IEgX0bJScukdHZMJDB7zqVJXgP_WSrHzkcecQEaj5cFLnZlCaS4TDXF-4g9gD4dOgA7RyE2nusWN0fWIgd7BC5JzdjIOU8Z9M8IoMB_wFjw8y0y8lshQCQJSgmgS0sg-GUihaICY0SuV_CRNuLSYwcJhUgA7-a1fXHnfg5dUOJt7nDyAfAsocbLxv1p35c9bgFkyDQMMqDB0dcljZgyl8mAjcMIXv_XbmhZohbYKdFVHqT_IZ6m3q94GGZk3tpFB-FFNf023I_6HS3vqeWgEXA4ukB86FZdiCmeHVBbWuqyWvYLKZXITx-=w979-h593-no" width=500 />

- Web Servicesの知識はUIはいらないので、implementationを指定する
    - そうすることで、UIがDTOやRetrofitの知識を知らないですむ
- Entitiesの知識はUIが必要なので、apiを指定する
    - ただし、この場合、DAOsの知識までUIが知ってしまうので微妙
        - そこでCommon Value Objectsの導入

<img src="https://lh3.googleusercontent.com/nfbY2Dd1jEsPoFt75QM0ikteiUp7vVYGhMWJvx0nCaRgEw_WKMR3c0MABc_UOr3pLDAZnq6s6rBgNQpwF1jD7C7WR_o2ipG8_IYodL_94C8rLYaY-gP6uIgEajxxC9RC7d4vRWn6cRhT4s-ZFcwJPvnHwH3WFYMmXRNu_OnHQMSWzY6oBnjQe-LzH_km1C38s18qLcLgvdMWbJvqJLfOjOqUQrv485RI4M-R5VWfEbTAgrZ7H7XFBvZ1WVwMizg4IpWEIDWfJvO788FiavN8ipD0LS6FCYIfcyfNcmng92uFgp7wtifaSKVLl1_DnLS3rkU-QB2nJr9l_if46OjLzCMeZ33JOLRbBYxGmch0NGD_ruyQSpmK3bceTHkZ-tzaNX-6ZxY-QJsjT3OdfcI5uJnnO_4vJWRwUQ07p9FJJ8-LqAuI2rjFpS3gh3aw9F4xhGb3LPB_RynE2wQFdw7ntOj_yJR6oakRxKk2JrE-0DiTH-5BrItd0QCngLvf-7oyDqR08LidmheYrMm4WIFTiMWW5QKceMotv6Q1Sxj5jpvW_vsIiNZ7gWwtx5Xk1MljhKYdb6q900RAZozhiAIYuYtgGlvt8yBE4cA7xdjlRD4my4F3CBNneEWt-3ZtYk7TQ3OeCgEtgySEP6zQQvIqIEhSQlCEdWy8=w2160-h1210-no" width=500 />

こうすることで、UIがDAOの知識を知らずに済む

### Fakes for tests

<img src="https://lh3.googleusercontent.com/hkjj2f7NohLfThmJNQienXwvchycNGrieetXpGITrD1Lyupjz_0HHbZBdRNDpO2rR2f8RRhBB2hfqX4FkHnv7-UpRehP6Kry0vZE-upK7MjZcINGpEzCCR9mB-H_LjQ2jgQGTpn3rsvMJEoLfKp2B1GNoULlj9aLucM0_gs9nHiN1MIMHRL4AhwQHVdk6ai8yyYtu7T8LBf-vBvFNX2e8nCbuvMNclYZvSukJwLpyGX8LX5bmtTnSuuUzJvPtvCR041-tFqZd2ju44eHkeE5YrU8ZYGn3y62ojO-4A91rj8-g8tIJ3okmISj4U4qYMKRUs6CjnKYBX4bQz42ZF1jdrBMKvP3wCqfwJ4OmAkCA6lpVWWKBFZijStGF2_S2UnmXeQn5PcjyeeAqiExz644SMNifNidRglSEEVq19OARawWvp_U40MjG2Jyj3M0jxixF4qVm_cJdehDBQ_mJpUnPngzeqFfYZ-pIeC_bc4D8bzfeqv7IhVvXd_cLDsGCNlt6VpuvkMbOAVShoxg-mejMLtSA_bSOAuTalT3clfTQWJJyzPNvHW7te5sczXVdIthtqvSnsStJCItNbdcuPpDDScCKLzqHVBIWj2NT1McfivRzECx2P_AYEWkCkPgDLESKYFUz-ORoJJeLhseWuqdz2VWU-NtXyus=w2160-h1202-no" width=500 />

- モックよりも優れた方法
    - fakeはRepository（対象のクラス）がどのように振る舞うかを理解した上で作られる
        - （モックはその場、その場で作られがちという意味？）

### Featureモジュールの利点

- カプセル化
- Dynamic Delivery

### Layerごとに分ける利点

- サードパーティライブラリの依存を独立
- 構造をもたらす

### Dynamic Deliveryの課題

Featureモジュール間のNavigationをどうするか?

- appから、Dynamic Featureモジュールへの依存を持てない
    - （これはDynamic Deliveryの制約のため）
    - 文字列、リフレクションでstartActivityするしかない
    - Fragmentも同様

今後、Navigationライブラリで自然に書けるようになるらしい😃

## データベース

### 1つのデータベースのみ持つ

- pros
    - コネクションの管理が楽
    - テーブルのシェアが簡単
- cons
    - モジュール間に依存が生まれる

### モジュールごとに1つのデータベースを持つ

- pros
    - モジュール間の依存が疎になる
- cons
    - データベースのコネクション管理がめんどい
    - 繰り返しのコードが生まれやすい

### hybridアプローチ

特にルールを決めずに、データベースモジュールを作っていく

- pros
    - Flexible
- cons
    - Fiexible
        - どこに、どのDAO、Entityを置けば良いのかの判断を常に迫られる

<img src="https://lh3.googleusercontent.com/gTSq0TPmgz2B2iyb_qidLaq-G8VSSnmSOI_VDez7Mu0UBASB5h7PIRDKWMJPrH40XGxFCw3QDenkSgiD6f4D2U5EhBAK_OEayUdIVtH5hX53vvUQM7Q-4AiJHx0NDt-4iRjCjaTeIlJs16YyeZpEt6iIvgp9HrAD_cr7vE5myFyRzeJnaHSU_vzZULGICVIWBmgxKFAcHUKDdiHRHUMuncyewUxZGc_O5t3xlQvXUanCQxVh6NXvBQZXxhFpqYRMwPiSTTNmIGz8BFFnX5JqHZAdKULHDZDV8FjeT6KtQXFjPQMMrEm6c_Sdka9aHgH5c4U76VbhF9lJd1xnn0i_jeVhbBXZ4Ylhwuem8trNGg0L2-JSphGwz2dlud0oS39CwN3spcRFEXkyQ3F6AjaW0omnmWYh1S3r2p_W9Wd3Yxyy3H7egkYyHdYmdqU76m5s0y9pCL3VWaiGvLZFr61Z0NVABPVW9O_dDS5jroA8OV4yJn_q6kMEQ_QgYz5uCatq_a86gXcWFRBipdZfhqMCZCMvJ8apGzKzrIyDuiM-3GMk_IZLR4ybtgdpFoxIEdXhCPHpUnWiCPQ6xsH3Od2P485MLbKLdZWDp76NB36DZRhlPAs33Z5h-ItZfm1mKok05rXrzibepyocRNr2rUKnOoyaHtsaxKek=w2160-h1202-no" width=500 />

マルチモジュール対応Room絶賛開発中らしい😃

## Android free modules

multi platformにする場合

- Kotlin nativeとか
    - ちゃんとユースケースを持っているなら、multi platformは良い選択

テストの高速化

- 確かにテストは早くなるが、テストのためだけにAndroid freeにするのは冗長
    - Robolectricで十分
    - Android Frameworkの再開発は無駄e

## 最後に

- 最小限の依存関係でモジュールを構成する
- 関心事の分離
    - ビジネスロジックがUIに依存しない
- 抽象化
    - 抽象化は非常に便利だが、複雑になりがちなので、抽象化により得れる利点を常に考える
- これらのモジュール化のやりかたは、あくまでオプション
    - ユーザはマルチモジュールなコードだからって5つ星はくれない
    - あくまでユーザが一番大事
