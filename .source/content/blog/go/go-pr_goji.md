+++
date = "2015-02-22T04:11:23Z"
title = "Go: GojiにPRした話"
tags = ["go", "github"]
blogimport = true
type = "post"
+++

Goにハマりつつあります. 最近家ではGo or Androidって感じです.

最近, GoのWEBフレームのソースコードをちょいちょい読んでいて, GojiにPRをしたのでその話.


## Goji?

`Goji is a minimalistic web framework that values composability and simplicity` です. SinatraのようなMicro Frameworkになっています.


## どこにPRしたか?

バグではなくて, こっちのほうがGoライクにだよ!と思ったのでPRをしました.
計2回PRしたので, それぞれ紹介します.


## switch type assertion

1つ目は, switch type assertionについてです.
switch文の冒頭にある, type assertionの結果を使うようにしました.


```
--- a/web/pattern.go
+++ b/web/pattern.go
@@ -32,13 +32,13 @@ type Pattern interface {
 }

 func parsePattern(p interface{}) Pattern {
-       switch p.(type) {
+       switch v := p.(type) {
        case Pattern:
-               return p.(Pattern)
+               return v
        case *regexp.Regexp:
-               return parseRegexpPattern(p.(*regexp.Regexp))
+               return parseRegexpPattern(v)
        case string:
-               return parseStringPattern(p.(string))
+               return parseStringPattern(v)
        default:
                log.Fatalf("Unknown pattern type %v. Expected a web.Pattern, "+
                        "regexp.Regexp, or a string.", p)
```

こちらのほうが, caseの中で個別にtype assertionをする必要がないので, 良いと思います.


## range syntax

次は, range syntaxです.
s.patsはsliceになっており, 最初はlenで実装していましたが, rangeを使ったほうがスッキリするじゃないかと思いPRしました.


```
-       for i := 0; i < len(s.pats); i++ {
+       for i, pat := range s.pats {
                sli := s.literals[i]
                if !strings.HasPrefix(path, sli) {
                        return false
@@ -55,7 +55,7 @@ func (s stringPattern) match(r *http.Request, c *C, dryrun bool) bool {
                        return false
                }
                if !dryrun {
-                       matches[s.pats[i]] = path[:m]
+                       matches[pat] = path[:m]
                }
                path = path[m:]
        }
```

これは, 正直好みの問題かなとも思います. しかしrangeを使うほうが, 全ての`s.pats`でループするということを強調できるので, readableかなと思います.


## まとめ

GojiへのPRがプライベートでの初PRになりました. 今回くらい簡単な内容でしたら問題無いですが, もっと英語を勉強せねばとも思います.
