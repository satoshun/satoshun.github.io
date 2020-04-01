+++
date = "2016-08-01T00:00:00Z"
title = "Golang: structで宣言したメソッドを明示的に呼び出す"
tags = ["golang"]
blogimport = true
type = "post"
+++

Goでは, 宣言したtypeにメソッドを定義することが出来ます.

典型的な用途としては, structでデータ型を宣言して, それにメソッドを定義する使い方だと思います.

```go
type Hoge struct {
}

func (h Hoge) h1() {
  fmt.Println("called h1")
}

func (h *Hoge) h2() {
  fmt.Println("called h2")
}
```

通常メソッドを呼ぶ際は, インスタンスを作成し, そのインスタンスを通じてメソッドをコールします.

```go
h := Hoge{}

h.h1() // output: called h1
h.h2() // output: called h2
```

このさい, h1, h2メソッドを通じて, 暗黙的にインスタンスも渡されています.

暗黙的に渡している部分を, 明示的に渡すと下のようになります.

```go
h := Hoge{}
Hoge.h1(h)     // output: called h1  ==  h.h1()
(*Hoge).h2(&h) // output: called h2  ==  h.h2()

(*Hoge).h1(&h) // output: called h1  ==  h.h1()
// Hoge.h2(h)  // error:  invalid method expression Hoge.h2 (needs pointer receiver: (*Hoge).h2)
```

また, 下のような事もできます.

```go
h1Direct := Hoge.h1
h1Direct(h) // output: called h1  ==  h.h1()

h1 := h.h1
h1() // output: called h1  ==  h.h1()
```


## まとめ

こんな書き方が出来ることを知らなかった(小並感)
