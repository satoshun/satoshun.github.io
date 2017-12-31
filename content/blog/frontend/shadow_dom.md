+++
date = "2015-03-14T00:00:00Z"
title = "HTML5: Shadow DOMについて"
tags = ["frontend", "html5"]
blogimport = true
type = "post"
+++

Web Componentsの機能の一つ Shadow DOMについて説明します.


## Shadow DOMとは?

外部から影響を受けない, 外部に影響を与えない隔離された要素です. 「カプセル化されたHTML要素」みたいな感じです.


## Shadow DOMの誕生背景

なぜShadow DOMが出てきたかについて説明します.

CSS, JavaScriptは, 全要素に対して影響が及ぶという性質があります.(グローバルに影響を与える) ある箇所を修正したら, 予期せぬ箇所にも影響が出てしまうことがよく起こりますが, これはバグを生み出しやすくとてもよくないです. 例をあげると,

```css
.hoge {
  font-size: 30px;
}
```

と書くと, `hoge`をclass属性に持つ要素の文字サイズが30pxになります. 偶然にも他の部分で`hoge`クラスセレクターを使っていたとしたらそのセレクターにも影響を与えてしまいます.
仮に, 他の人が作ったコンテンツが隔離されて使用できれば, 同じセレクターを使ったとしても, 問題がなくなります.

フロント側が年々複雑になったことにより, スタイルシート, JavaScriptが肥大化した結果, コンテンツを互いに隔離したいという要望が高まり, Shadow DOMが誕生しました.


## 使い方

Shadow DOMを作るには`createShadowRoot`APIを使います. 以下, Shadow DOM版Hello Worldです.

```html
  <div id="shadow"></div>

  <script>
  function insertShadow() {
    // Shadow DOMの作成
    var shadow = document.querySelector('#shadow').createShadowRoot();
    shadow.textContent = 'Hello world';
  }
  insertShadow();
  </script>
```

Shadow DOMが挿入され, Hello Worldと表示されます.

上の例ではShadow DOMの良さが全く現れていないので, もう1つ例をあげます. Shadow DOMは, templateと組み合わせることでより強力に使うことが出来ます.

```html
  <div id="hello1">Hello world1</div>
  <div id="hello2"></div>
  <template id="templateHello">
    <!-- ここのスタイルは, 外部に影響を与えない! -->
    <style>
      div {
        color: red;
      }
    </style>
    <div>Hello world2</div>
  </template>
  <script>
  function insertShadow2() {
    var target = document.querySelector('#hello2').createShadowRoot(),
        hello = document.querySelector('#templateHello').content;
    target.appendChild(hello.cloneNode(true));
  }
  insertShadow2();
  </script>
```

上記のように書くと, Hello world1は黒いままで, `color: red;`は効きません. なぜなら, Shadow DOMでtemplateの中身は外部と隔離されているためです. これがShadow DOMです.


## まとめ

Shadow DOMの概要, 簡単な使い方について書いてみました.

2014年現在は実用するには厳しいですが, 2017年くらいには実用的になっていると思います(なんとなく). 今すぐに使いたい人は https://github.com/Polymer/polymer などのpolyfillを使うのがいいかと思います.


## 参考サイト

- Shadow DOM 101: http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/
- Web Componentsで行うHTMLのコンポーネント化: http://ameblo.jp/ca-1pixel/entry-11815188808.html
