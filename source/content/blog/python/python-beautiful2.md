+++
date = "2015-04-30T00:00:00Z"
title = "Python: Pythonライクな書き方 パート2"
tags = ["python"]
blogimport = true
type = "post"
draft = true
+++

Pythonライクな書き方について紹介します. パート2です.


## formatメソッドを関数のように使う

```python
namef = 'Hello {name}'.format
workf = '{name} is a {work}'.format
namef(name='Ken')
workf(name='ken', work='software engineer')
```
