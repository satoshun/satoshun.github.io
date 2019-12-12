+++
date = "Mon Jan 14 01:20:35 UTC 2019"
title = "マルチモジュールの遷移について考える Part1.5"
tags = ["android", "multimodule", "gradle", "dynamicmodule"]
blogimport = true
type = "post"
draft = true
+++

[Part1](https://satoshun.github.io/2018/12/multi-module_transition_part1/)の続きになります。
Part1.5としているのは、Part1にdynamic feature moduleを補足した内容になっているためです。

# dynamic feature moduleとは?

利点は置いといて、実装、設計に与える影響としては依存関係が逆転する点です。
今ままでのアプリだと次のような依存関係になっていました。

---

{{< figure src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLukc_oixbB7pUkUzoqw77pzCVDgvxic_jqxOodipTpSINdvnRavwNcbIX49nOKF6vUzBXfn-FcfO-RzpnkNXsha5Yi01H6LlMuQUlZvcc6s5GMboOPOYermg7K25NfPh3hC5KcvnUbSd5n0LsXe9kINvwdQmUn1qt0Y8iB90mN0ci3YQEAS3cavgK0mmO0" width="300" >}}

---

```
@startuml

title 従来のアプリ依存図

component [appモジュール] as app
component [サブ1モジュール] as sub1
component [サブ2モジュール] as sub2
component [コアモジュール] as core


app -down-> sub1
app -down-> sub2

sub1 -down-> core
sub2 -down-> core

@enduml
```

仮にサブ1、サブ2をdynamic-featureとして切り出すと次のようになります。

---

{{< figure src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLukc_oixbB7pUkUzoqw77pzCVDgvxic_jqxOodipTpSINdvnRavwNcbIX49nOKF6vUzBXfn-FcfO-RzpnkNXsha5Yi01H6LlMuQUlZvcc6s5GMboOPOYermg7K25NfPh3hC5KcvnUbSd5nWSnMq4t9By_JjGCx2MG2YW0Na80BG7IXQ08BeUY2A798pKi1XXO0" width="300" >}}

---

```
@startuml

title 従来のアプリ依存図

component [appモジュール] as app
component [サブ1モジュール] as sub1
component [サブ2モジュール] as sub2
component [コアモジュール] as core

sub1 -down-> app
sub2 -down-> app

app -down-> core

sub1 -down-> core
sub2 -down-> core

@enduml
```