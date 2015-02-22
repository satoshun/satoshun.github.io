+++
date = "2015-02-22T02:32:23Z"
title = "CSS: Clearfixについて"
tags = ["css"]
blogimport = true
type = "post"
+++


CSSには「clearfix」と呼ばれるテクニックがあります.

この記事では, なぜclearfixを使うのか, clearfixとは何なのかについて説明します.


## floatの問題点

clearfixを説明する前に, floatプロパティの問題点について説明します.

まず例をあげます.

```html
<div>
  <img class="test" src="hoge.png" style="float: right" />
  <p>test1</p>
  <p>test2</p>
</div>
```

上記HTMLは, imgが右に寄り, その左側にtest1, test2と表示されます.
floatは, 指定したエレメント以降のエレメントを反対側に回りこませることが出来ます.

回りこみを止めたいときは, `clear: both;`プロパティを指定します.

```html
<div>
  <img class="test" src="hoge.png" style="float: right" />
  <p>test1</p>
  <p>test2</p>
</div>

<!-- clear:bothがないと, このdivも左側に回りこむ -->
<div style="clear:both;">
  hogehoge
</div>
```

`clear:both;`の記述を忘れると, 後続の意図しないエレメントにも影響を及ぼしてしまうので, 忘れずに指定しなければいけません.

とは言っても, floatingの解除は忘れてしまいがちです. そこでclearfixと呼ばれるテクニックを使います.


## clearfixとは

clearfixとは, after擬似要素を使いfloatを解除するテクニックです.

具体的には以下のように記述します.

```css
.clearfix:after {
  content: "";
  clear: both;
  display: block;
}
```

`clear: both;`をしてくれる要素をafter擬似要素で挿入しているだけです.
これを, 親エレメントにつけておけば, floatを解除してくれます.

```html
<div class="clearfix">
  <img class="test" src="hoge.png" style="float: right" />
  <p>test1</p>
  <p>test2</p>
</div>
<!-- ここにclear:both; 擬似要素が挿入される -->
<div>
  hogehoge
</div>
```

後続の要素に`clear:both;`を記述しなくてよいので, 無駄なミスを防ぐことが出来ます.


## まとめ

floatはCSSのハマリポイントの1つだと思います. 要素の高さが取得できないなどの時は, floatが解除されていないかを疑いましょう.

他にも, いろいろなclearfixの書き方があるので, 調べてみて下さい.
