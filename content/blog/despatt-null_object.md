+++
date = "2015-02-22T02:22:23Z"
title = "Design Pattern: Null Object"
tags = ["design_pattern", "code", "python"]
blogimport = true
type = "post"
+++

Null Objectパターンは, オブジェクト自身にNullかどうかの判定をしてもらうパターンです.

このパターンのメリットは, `if (obj == null) `のような面倒臭い記述を除去出来るところです.
また, ポリモーフィズムにより, nullの時の処理をObjectに委譲することが出来ます.
(nullの時の処理を, Objectに持たせることが出来るパターン)

## 例

例があった方が分かり易いので, 簡単なサンプルプログラムです.

まずは, Null Objectを使わない場合になります.

```python
class Student(object):
    def __init__(self, id):
        self.id = id

    @staticmethod
    def get_student(id):
        if id <= 10:
            return Student(id)
        return NullStudent(id)

    def show(self):
        print('id:{}'.format(self.id))


student = Student.get_student(1)
if student is not None:
    student.show()
else:
    print('not student')

```

次にNull objectパターンを使い, `if student is not None:` を除去します.

```python
class Student(object):
    def __init__(self, id):
        self.id = id

    @staticmethod
    def get_student(id):
        if id <= 10:
            return Student(id)
        return NullStudent(id)

    def show(self):
        print('id:{}'.format(self.id))

    @property
    def is_null(self):
        return False


class NullStudent(Student):
    @property
    def is_null(self):
        return True


student = Student.get_student(1)
if student.is_null:
    print('not student')
else:
    student.show()
```

NullStudentクラスを定義し, `is_null`メソッドでTrueを返すようにすることで, Nullの処理を委譲しまう.

しかし, これだけだと旨味が少ないので, ポリモーフィズムを使ってさらに書き換えてみます.


```python
class Student(object):
    def __init__(self, id):
        self.id = id

    @staticmethod
    def get_student(id):
        if id <= 10:
            return Student(id)
        return None

    def show(self):
        print('id:{}'.format(self.id))

    @property
    def is_null(self):
        return False


class NullStudent(Student):
    @property
    def is_null(self):
        return True

    def show(self):
        print('null student')


student = Student.get_student(100)
student.show()
```

これでオブジェクト指向的なコードになりました.


### まとめ

nullの時のboilerplate的な処理をオブジェクトに閉じ込めることが出来るパターンになります.
`if hoge != null`が良く出てくるようなら, このパターンを使うと良いと思います.
