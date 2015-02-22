+++
date = "2015-02-22T02:20:23Z"
title = "様々な言語のフィボナッチ関数"
tags = ["algorithm", "code", "java", "javascript", "golang", "python", "shell"]
blogimport = true
type = "post"
+++

Java, JavaScript, Go, Python, Bashでfibonacciを書いてみました.

極力, その言語特有の機能を使って実装するようにしました.


## Go

`type fibonacci int`で, int型にfibonacci用の関数を生やしました.
せっかくなので, goroutineも使ってみました.

```go
package main

import "fmt"

type fibonacci int

func (self fibonacci) value() chan int {
  ch := make(chan int, 1)
  a, b := 0, 1
  index := 0
  go func() {
    defer close(ch)
    for {
      if int(self) < index {
        break
      }
      a, b = b, a+b
      index++
      ch <- a
    }
  }()

  return ch
}

func main() {
  var i fibonacci
  i = 10
  for v := range i.value() {
    fmt.Printf("%d ", v)
  }
  fmt.Println()
}
```


## Python

iteratorを定義して, 実装しました.

```python
class Fibonacci(object):
    def __init__(self, i):
        self.__i = i
        self.index = 0

    def __iter__(self):
        self.a = 0
        self.b = 1
        return self

    def __next__(self):
        while self.__i >= self.index:
            self.index += 1
            self.a, self.b = self.b, self.a + self.b
            return self.a
        raise StopIteration()


if __name__ == '__main__':
    f = Fibonacci(10)
    for number in f:
        print(number, end=' ')
    print()
```


## bash

 `$[]`でexpressionを表現しています. これ実装するまで知りませんでした.

```sh
#!/bin/bash

function fib()
{
    case $1 in
        0) echo 0 ;;
        [1-2]) echo 1 ;;
        *) echo $[`fib $[$1-1]` + `fib $[$1-2]`] ;;
    esac
}

for i in {0..10}; do
    fib $i
done
```

## JavaScript

単純な再起で実装しました.

```javascript
function fib(i) {
  return function inner(i) {
    return i > 2 ? inner(i - 1) + inner(i - 2) : (i === 2 ? 1 : i);
  }(i);
}

for (var i = 0; i <= 10; i++) {
  console.log(fib(i));
}
```

## Java

Java8のstreamを使って実装しました. filter非常に便利.

```java
import java.util.*;


public class Fibonacci {
    public static void main(String[] args) {
        System.out.println(fibonacci(10));
        System.out.println(fibonacci(20));
    }

    public static int fibonacci(int n) {
        List<Integer> data = new ArrayList<Integer>() {{
            add(0); add(1); add(1); add(2); add(3);
        }};

        for (int i = 4; i < n; i++) {
            data.add(
                data
                    .stream()
                    .filter(nn -> nn >= data.get(data.size()-2))
                    .mapToInt(nn -> nn)
                    .sum()
                );
        }
        return data.get(n);
    }
}
```


## まとめ

fibonacciの計算には向かないけど, Java8のstreamが便利でした. Javaも進化しているんだなと.
