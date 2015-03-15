+++
date = "2015-03-14T00:00:00Z"
title = "本: Web製作者のためのCSS設計の教科書"
tags = ["book", "web", "api"]
blogimport = true
type = "post"
draft = true
+++

設計の基本
- 修正がしやすい, シンプルな構成にする
- line-height: 1.5とすれば, それぞれの子要素のfont-sizeで1.5だけline-heightする. 1.5emとすると, 親のline-heightが継承される.


詳細度
- 詳細度が一緒なら, class="hoge fuga" の順番の後ろのものが優先
- important, インライン, ID, class, 要素, ユニバーサルの順番
- 擬似要素は, 詳細度は要素セレクタと一緒. 5番目

セレクタのリファクタリング
- h2.hoge のように要素を指定しない. 無駄に詳細度をあげない
- セレクタを短くして, 移植性を高める. ul li a だとしたら, liはいらないんじゃないか? ul aでも十分かもしれない
- IDセレクタに依存すると, その場所に依存してしまうし, ID名を変えるときもある. その時にCSSを変更する必要が出てくる. classなら, コンポーネントのような形で様々な場所で登場させられる


## コンポーネント設計

見た目と構造を分離
- padding, margin, widthなどの構造は.alertセレクタ, colorなどを, .error, .warningに宣言することで, 分離できる
