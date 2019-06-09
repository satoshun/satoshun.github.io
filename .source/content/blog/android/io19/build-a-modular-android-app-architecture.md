+++
date = "Sun Jun  9 05:02:05 UTC 2019"
title = "IO19: Build a Modular Android App Architectureのまとめ/感想"
tags = ["android", "arch", "io19"]
blogimport = true
type = "post"
draft = true
+++

[Build a Modular Android App Architecture (Google I/O'19)](https://www.youtube.com/watch?v=PZBg5DIzNww)

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
    - Plaidの例

<img src="https://lh3.googleusercontent.com/eKPc5MsRrRbOjCXfIJnVppLjpevSWsz4e13QVgu8wAmd1AI2An-aVSerQ1Wz0DuALe6RCP3JPHkkJh1mBZx-9N2lwJj2i4aBoCrIqztFu2CLyZXu6QZBEC5AuO5FzXD90hqMMV8J12g0Sn9WjVMUQHA4RxREdVaSmNwXRt0OBfIVG_4UYdnp8SDJJ8OOKPsDqBX3cOhopNBWEK2B6Of-VwkN25G3IRrV952EnleuGaFbsxrGXkGTv64oAbnB6aw71alRN4JO3HUbXlJY54XZOXtJV3wMHu1NWOqUF9sM5GsEtcMQn_fdKsfkynN3UIZ-QTRNI2PKefKKV7X-aPZp6Jg_OAm9WoGA-VktrPj9dQRpx1EPvT6av3KfChS_x0_oCdIeJpXwLBObooQMmRpiJzKJCb112Pnq4s3XhDOcK9n08kOQZgOHOOFx_jC-CjhUEMRmK3j-XQBNmmJjqLdzBksgpajJqpbl8QS7B6snoEXPRa7AoVIfhPNRQurFi5880dhTx0JrFYEm1yeHzHYTHqW7bXpjjtAYWW5MQJ6HDwSOpphlZiR1z6AnJK3DTiM5K7Q1-p27e802TcjF8O3yhFq4XDLl6IqZ_99TFoxcNcocxFCd2FlYcBcD1lOsXcIwpXOMRDapFTv7Tx8UBhzeeijqGc512lTI=w720-h402-no" width=500 />

    - メリット
        - 分離/Isolation
        - 例えば、Retrofit、Roomへの依存を特定のモジュール内に閉じ込めることが出来る

<img src="https://lh3.googleusercontent.com/_81GbLd7eZJQvhnKKjlzv08wTjRKtzqt_IZPQNf1BlYcUmCvBv1B4kD6Pjw5b0MfQfPMke62kIzaXW7AjRON695kLYHQKHkk_-03L7IynQHKNggkWh9L5BZWafVz2nEz4f8Bn_mFDjuo2FT8iK6Ho4HI8L3d67qQh7N-Ljj-5G7glOklHSEQMGLOPzLRr7bH_b_jogHxLZS48_hWkeeFLqsBFscsiPXMjuyz5mjfCv9z-ZgvKa45LL8vkqe6jMWaBH2bGhkMJChHCD8fMbPaVDN-eNaSq5hvU6XcmAexyYMqM72exG2fuzxj2aGDPFCtZ3v9_8g5RMHVTb4wG4yG87epXC2XDRG9TJ3P6hAUl4IEgX0bJScukdHZMJDB7zqVJXgP_WSrHzkcecQEaj5cFLnZlCaS4TDXF-4g9gD4dOgA7RyE2nusWN0fWIgd7BC5JzdjIOU8Z9M8IoMB_wFjw8y0y8lshQCQJSgmgS0sg-GUihaICY0SuV_CRNuLSYwcJhUgA7-a1fXHnfg5dUOJt7nDyAfAsocbLxv1p35c9bgFkyDQMMqDB0dcljZgyl8mAjcMIXv_XbmhZohbYKdFVHqT_IZ6m3q94GGZk3tpFB-FFNf023I_6HS3vqeWgEXA4ukB86FZdiCmeHVBbWuqyWvYLKZXITx-=w979-h593-no" width=500 />

    - Web Servicesの知識はUIはいらないので、implementationを指定する
    - Entitiesの知識はUIが必要なので、apiを指定する
        - ただし、この場合、DAOsの知識をUIが知ってしまうので微妙
            - Common Value Objectsの導入
<img src="https://lh3.googleusercontent.com/nfbY2Dd1jEsPoFt75QM0ikteiUp7vVYGhMWJvx0nCaRgEw_WKMR3c0MABc_UOr3pLDAZnq6s6rBgNQpwF1jD7C7WR_o2ipG8_IYodL_94C8rLYaY-gP6uIgEajxxC9RC7d4vRWn6cRhT4s-ZFcwJPvnHwH3WFYMmXRNu_OnHQMSWzY6oBnjQe-LzH_km1C38s18qLcLgvdMWbJvqJLfOjOqUQrv485RI4M-R5VWfEbTAgrZ7H7XFBvZ1WVwMizg4IpWEIDWfJvO788FiavN8ipD0LS6FCYIfcyfNcmng92uFgp7wtifaSKVLl1_DnLS3rkU-QB2nJr9l_if46OjLzCMeZ33JOLRbBYxGmch0NGD_ruyQSpmK3bceTHkZ-tzaNX-6ZxY-QJsjT3OdfcI5uJnnO_4vJWRwUQ07p9FJJ8-LqAuI2rjFpS3gh3aw9F4xhGb3LPB_RynE2wQFdw7ntOj_yJR6oakRxKk2JrE-0DiTH-5BrItd0QCngLvf-7oyDqR08LidmheYrMm4WIFTiMWW5QKceMotv6Q1Sxj5jpvW_vsIiNZ7gWwtx5Xk1MljhKYdb6q900RAZozhiAIYuYtgGlvt8yBE4cA7xdjlRD4my4F3CBNneEWt-3ZtYk7TQ3OeCgEtgySEP6zQQvIqIEhSQlCEdWy8=w2160-h1210-no" width=500 />

    - こうすることで、UIがDAOの知識を知らずに済む
        - より細かく制御が出来る

- Fakes for tests

<img src="https://lh3.googleusercontent.com/hkjj2f7NohLfThmJNQienXwvchycNGrieetXpGITrD1Lyupjz_0HHbZBdRNDpO2rR2f8RRhBB2hfqX4FkHnv7-UpRehP6Kry0vZE-upK7MjZcINGpEzCCR9mB-H_LjQ2jgQGTpn3rsvMJEoLfKp2B1GNoULlj9aLucM0_gs9nHiN1MIMHRL4AhwQHVdk6ai8yyYtu7T8LBf-vBvFNX2e8nCbuvMNclYZvSukJwLpyGX8LX5bmtTnSuuUzJvPtvCR041-tFqZd2ju44eHkeE5YrU8ZYGn3y62ojO-4A91rj8-g8tIJ3okmISj4U4qYMKRUs6CjnKYBX4bQz42ZF1jdrBMKvP3wCqfwJ4OmAkCA6lpVWWKBFZijStGF2_S2UnmXeQn5PcjyeeAqiExz644SMNifNidRglSEEVq19OARawWvp_U40MjG2Jyj3M0jxixF4qVm_cJdehDBQ_mJpUnPngzeqFfYZ-pIeC_bc4D8bzfeqv7IhVvXd_cLDsGCNlt6VpuvkMbOAVShoxg-mejMLtSA_bSOAuTalT3clfTQWJJyzPNvHW7te5sczXVdIthtqvSnsStJCItNbdcuPpDDScCKLzqHVBIWj2NT1McfivRzECx2P_AYEWkCkPgDLESKYFUz-ORoJJeLhseWuqdz2VWU-NtXyus=w2160-h1202-no" width=500 />

- モックよりも優れた方法
    - fakeはRepository（対象のクラス）がどのように振る舞うかを理解した上で作られる

- Featureモジュール
    - カプセル化
    - オンデマンド配信
- 層
    - サードパーティライブラリの依存を独立
    - 構造をもたらす

- Dynamic Deliveryの課題
    - Navigation
        - appから、Dynamic Featureモジュールへの依存を持てない
            - 文字列、リフレクションでstartActivityするしかない
            - Fragmentも同様
            - Navigationライブラリで自然に書けるようになる

## database

### one database

- pros
    - 管理が楽
    - テーブルのシェアが簡単
- cons
    - モジュール間に依存が生まれる

### モジュールごとにone database

- pros
    - モジュール間が疎になる
- cons
    - databaseのコネクション管理がめんどい

### hybridアプローチ

特にルールを決めずに、databaseモジュールを作っていく

- pros
    - Flexible!!
- cons
    - Fiexible!!
        - どこに置けば良いのかの判断を常に迫られる

<img src="https://lh3.googleusercontent.com/gTSq0TPmgz2B2iyb_qidLaq-G8VSSnmSOI_VDez7Mu0UBASB5h7PIRDKWMJPrH40XGxFCw3QDenkSgiD6f4D2U5EhBAK_OEayUdIVtH5hX53vvUQM7Q-4AiJHx0NDt-4iRjCjaTeIlJs16YyeZpEt6iIvgp9HrAD_cr7vE5myFyRzeJnaHSU_vzZULGICVIWBmgxKFAcHUKDdiHRHUMuncyewUxZGc_O5t3xlQvXUanCQxVh6NXvBQZXxhFpqYRMwPiSTTNmIGz8BFFnX5JqHZAdKULHDZDV8FjeT6KtQXFjPQMMrEm6c_Sdka9aHgH5c4U76VbhF9lJd1xnn0i_jeVhbBXZ4Ylhwuem8trNGg0L2-JSphGwz2dlud0oS39CwN3spcRFEXkyQ3F6AjaW0omnmWYh1S3r2p_W9Wd3Yxyy3H7egkYyHdYmdqU76m5s0y9pCL3VWaiGvLZFr61Z0NVABPVW9O_dDS5jroA8OV4yJn_q6kMEQ_QgYz5uCatq_a86gXcWFRBipdZfhqMCZCMvJ8apGzKzrIyDuiM-3GMk_IZLR4ybtgdpFoxIEdXhCPHpUnWiCPQ6xsH3Od2P485MLbKLdZWDp76NB36DZRhlPAs33Z5h-ItZfm1mKok05rXrzibepyocRNr2rUKnOoyaHtsaxKek=w2160-h1202-no" width=500 />

マルチモジュール対応Room絶賛開発中😃

## Android free modules

multi platform
- Kotlin nativeみたいなやつ
    - ちゃんとユースケースを持っているならええんちゃう？

faster tests
- testのためだけにAndroid freeにするのは冗長
    - Robolectricで十分やで

## 最後に

- 最小限の依存関係を
- 関心事の分離
    - ビジネスロジックがUIに依存しないとかね
- 抽象化
    - 複雑になりがちなので、抽象化により得れる利点を常に考える
- これらのやりかたは、あくまでオプション
    - ユーザはマルチモジュールなコードだからって5つ星はくれない
    - あくまでユーザが一番大事
