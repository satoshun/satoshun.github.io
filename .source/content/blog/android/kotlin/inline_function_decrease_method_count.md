+++
date = "2018-08-02T00:00:00Z"
title = "Kotlin: Inline Functionだけではメソッドカウントは減らない"
tags = ["kotlin", "android"]
blogimport = true
type = "post"
+++

## 結論

- R8/Proguardをちゃんと使う

## 背景

Inline Functionを使うと、関数がインライン化されるので、自動的にメソッド数が1減ると思っていました。けど、そんなことはなかったという話です。

実際に動かして確認してみます。下記の関数を追加し、Build APK → Analyze APKでclasses.dexのメソッドカウントを調べてみます。

まずは、R8無しで調べてみます。

```kotlin
fun hoge() {
    println("hogehoge")
}
```

当然メソッドカウントが1増えています。

次に、inlineを付けてみます。

```kotlin
inline fun hoge() {
    println("hogehoge")
}
```

こちらもメソッドカウントが1増えました。
show Kotlin Bytecode → Decompileを使い、decompileされたJavaファイルを見ると、inline hogeメソッドに対応したメソッドが生成されていることが分かりました。

```kotlin
@Metadata(
    mv = {1, 1, 10},
    bv = {1, 0, 2},
    k = 2,
    d1 = {"\u0000\b\n\u0000\n\u0002\u0010\u0002\n\u0000\u001a\t\u0010\u0000\u001a\u00020\u0001H\u0086\b¨\u0006\u0002"},
    d2 = {"hoge", "", "production sources for module app"}
)
public final class MainActivityKt {
    public static final void hoge() {
        String var1 = "hogehoge";
        System.out.println(var1);
    }
}
```

ただし、ここで重要なのは、inlineをつけたメソッドはインライン化されるため、上記のメソッドは生成されるけど、実際にはコールされない点です。
コールされない、参照がないのでR8で削除することができます。

実際にR8/Proguardを有効にしてBuild APK → Analyze APKで確認したところ、inlineをつけた場合はメソッドカウントが増えていないことが確認できました。

## まとめ

- InlineメソッドはR8/Proguardと組み合わせればメソッドカウントを減らす効果がある
  - とはいえ、5.0(ART)からはruntimeからmultidexをサポートしているので、昔ほどメソッドカウントを意識する必要はないと思う
