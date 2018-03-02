+++
date = "2018-03-02T00:28:00Z"
title = "感想: Androidアプリ設計パターン入門"
tags = ["book", "android", "architecture"]
blogimport = true
type = "post"
+++

Androidアプリ設計パターン入門を読んだのでざっくりと感想。

https://peaks.cc/books/architecture_patterns

## 感想

- MVP
    - PresenterがViewとModelへの仲介役なので、Presenterはどうしてもfatになりそう
        - PresenterでView、Modelが何を出来るかを知らなければならない
    - Contractみたいなインタフェースを切るのは好き
        - それを見ただけで何をそのページでやっているのかが掴めるので
    - PresenterはContextを知らなく良い、Pure Javaなのでテスタブルだし綺麗になりそう
- MVVM
    - 個人的にはMVPより好き
        - ViewModelがViewの参照を持たなくて良いので少しスッキリする
            - ただViewへの参照がないだけで、LiveDataなりObservableFieldに値を書き出すので実質的にはViewがどんなことをしたいか知っているから同等といえば同等
    - Viewへの参照を持たないので、AACのViewModelへの適合性は高いと思う
        - DataBindingのObservableFieldとかを使わない前提。使うとViewへの参照を持ってメモリリークしちゃう
- Flux
    - 単一方向データフローは凄い良いと思う
        - 単一方向はFluxだけに限らないけど、Fluxを象徴する1つの特徴
    - ViewModelとかPresenterって処理が集中する傾向にあると感じていて、FluxだとStore、ActionCreatorって形で切り出せるから良い
    - FluxもAACとの相性は良いと思う
- 負債の話
    - 負債に対する解決策の1つとしてReact Nativeを出すのは発想として凄いと思ったし、そういう解決方法もあるのかと思った
- メモ
    - データ層の抽象化に関してはRepository的なものを作るで良さそう
    - データ側はRx、UI側に反映する時はLiveDataが良さそう
