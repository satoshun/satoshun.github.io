+++
date = "2015-04-30T00:00:00Z"
title = "Python: Pythonライクな書き方 パート1"
tags = ["python"]
blogimport = true
type = "post"
+++

Pythonライクな書き方についてまとめました. パート1です.


## for文関連


### indexを使いたいとき

Bad: range, lenを使う

```python
names = ['TDN', 'suzuki', 'tom']
for i in range(len(names)):
    print i, names[i]
```

Good: enumerateを使う

```python
for i, name in enumerate(names):
    print i, name
```

### 2つのコレクションを扱うとき

Bad: indexを使ってアクセスする

```python
names = ['TDN', 'suzuki', 'tom']
ages = [12, 19, 11]

for i in range(min(len(names), len(ages))):
    print names[i], ages[i]
```

Good: 組み込み関数zipを使う

```python
for name, age in zip(names, ages):
    print name, age
```


## dictionaryでkey, valueを使う

Bad: keyでアクセス, valueを取得する

```python
persons = {'TDN': 12, 'tanaka': 24, 'nakata': 11}
for k in persons:
    print k, persons[k]
```

Good: itemsメソッドを使う

```python
persons = {'TDN': 12, 'tanaka': 24, 'nakata': 11}
for k, v in persons.items():
    print k, v
```


## 値の入れかえ(swap values)

Bad: tmp変数を定義する

```python
a = 0
b = 10

tmp = a
a = b
b = tmp
```

Good: onelineで交換する

```python
a, b = 0, 10

a, b = b, a
```


## dictionaryの作成

indexを使う方法

```python
names = ['TDN', 'suzuki', 'tom']
d = dict(enumerate(names))

> {0: 'TDN', 1: 'suzuki', 2: 'tom'}
```

2つのリストのペアを使う方法

```python
names = ['TDN', 'suzuki', 'tom']
ages = [12, 19, 11]
d = dict(zip(names, ages))

> {'tom': 11, 'suzuki': 19, 'TDN': 12}
```


## dictionaryを使ったグルーピング

Bad: ifでいちいちチェックする

```python
names = ['TDN', 'suzuki', 'tom', 'sato', 'seko']
d = {}
for name in names:
    key = name[0]
    if key not in d:
        d[key] = []
    d[key].append(name)
```

Good: setdefaultを使う

```python
names = ['TDN', 'suzuki', 'tom', 'sato', 'seko']
d = {}
for name in names:
    key = name[0]
    d.setdefault(key, []).append(name)
```

Good: defaultdictを使う

```python
from collections import defaultdict

names = ['TDN', 'suzuki', 'tom', 'sato', 'seko']
d = defaultdict(list)
for name in names:
    key = name[0]
    d[key].append(name)
```


## 効率のよいソート

Bad: cmpを用いてソートする

```python
names = ['TDN', 'suzuki', 'tom']
def cmp_function(a, b):
    if len(a) > len(b):
        return 1
    if len(a) < len(b):
        return -1
    return 0
sorted(names, cmp=cmp_function)
```

Good: keyを用いてソートする

```python
sorted(names, key=len)
```

## ファイルを開く, 閉じる時

Bad: finallyを使う

```python
f = open('build.gradle')
try:
    data = f.read()
finally:
    f.close()
```

Good: withを使う

```python
with open('build.gradle') as f:
    data = f.read()


## リスト内包

Bad: forをわざわざ使う

```python
total = 0
for i in range(10):
    total += i ** 2
```

Good: リスト内包, ジェネレータを使う

```
total = sum(i ** 2 for i in range(10))
```


## 文字列結合

Bad: stringを + でつないでいく

```python
names = ['TDN', 'suzuki', 'tom', 'sato', 'seko']
s = ''
for name in names:
    s += name
```

Good: joinメソッドを使う

```python
names = ['TDN', 'suzuki', 'tom', 'sato', 'seko']
''.join(names)
```


## デコレータを使う

Bad: メソッド内にcacheのロジックが入りこんでいる

```python
def get_data(filepath, saved={}):
    if filepath in saved:
        return cache[filepath]

    with open(filepaht, 'r') as f:
        saved[filepath] = f.read()
    return saved[filepath]
```

Good: cacheのロジックを外部に切り出す

```python
from functools import wraps

def cache(func):
    saved = {}

    @wraps(func)
    def wrapper(filepath):
        if filepath in saved:
            return saved[filepath]
        result = func(filepath)
        saved[filepath] = result
        return result
    return wrapper


@cache
def get_data(filepath):
    with open(filepath, 'r') as f:
        return f.read()
```


## contextmanagerを使う

Before: 例外をpassしているのが正しいのか, 実装し忘れか判断が付き難い

```python
import os

try:
    os.remove('.tmp')
except OSError:
    pass
```

After: ignoredのように明示的な名前をつけて定義する

```python
import os
from contextlib import contextmanager

@contextmanager
def ignored(*exc):
    try:
        yield
    except exc:
        pass


with ignored(OSError):
    os.remove('tmp')  
```


## まとめ

パート2に続く..


## 参考

[Transforming Code into Beautiful, Idiomatic Python](https://www.youtube.com/watch?v=OSGv2VnC0go)
