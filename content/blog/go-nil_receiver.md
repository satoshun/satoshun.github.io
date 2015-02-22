+++
date = "2015-02-22T02:08:23Z"
title = "Golang: nil pointer receiverの話"
tags = ["golang", "code"]
blogimport = true
type = "post"
+++

nil pointer receiverについてお話しようと思います. 具体例をあげたほうが分かりやすいので, コードを元に説明していきます.

gist: https://gist.github.com/satoshun/3dc1302dbc163c9a9245

にソースコードがあります.


## nil pointer receiver

nilについて. 下のコードを見てください.

```
package main

import "fmt"

type A struct {
}

func (a *A) b() {
    fmt.Println(1000)
}

func main() {
    var a *A = nil
    a.b()
}
```

どう考えても, `runtime error: nil pointer access` 的なものが出るだろうと思っていました.

```shell
sato$ go run main.go
1000
```

ちゃんと実行できました. Goではnilにも型情報があるので, nilの場合でもPointer receiverの場合は実行できるのです!
この機能を使えば, Pointer receiverの中でnilの場合に処理を変えることが可能です. 覚えておくと便利だと思います.

ちなみに, Value receiverの場合はエラーが出ます.

```
package main

import "fmt"

type A struct {
}

// b is value receiver
func (a A) b() {
    fmt.Println(1000)
}

func main() {
    var a *A = nil
    a.b()
}
```

## まとめ

nil Pointer receiverは, [Null Objectパターン](http://blog.s-shun.net/design_pattern-null_object/)と似ている気がしました. Receiverのほうでは, nilでも動くようにしておく必要がありそうです.
