+++
date = "Sat Jan  5 02:15:50 UTC 2019"
title = "dexdumpメモ"
tags = ["android", "dalvik", "dexdump"]
blogimport = true
type = "post"
+++

Android build-toolsにはdexdumpというdexファイルを逆アセンブルするツールがあります。
逆アセンブルしたファイルを読みたかったのでそれのメモです。

内容はまとまっていません。自分用のメモレベルの記事になります。

dexファイルをコマンドから作るにはjavacとd8コマンドを使います。d8コマンドは https://r8.googlesource.com/r8 の手順通りにビルドします。

```
javac *.java // or kotlinc *.kt
java -jar $R8_HOME/build/libs/d8.jar --lib $ANDROID_HOME/platforms/android-28/android.jar --release *.class
```

d8コマンドで出来たdexファイルをdexdumpで逆アセンブルします。

```
$ANDROID_HOME/build-tools/28.0.3/dexdump -d classes.dex
```

例えば次のJavaコードを逆アセンブルしてみます。

```java
class Test {
    public void main() {
        show(1, 10.0);
        String[] array = {""};
        toggle(array);
        fade();
        drag();
        hide();
        click();
        hover();
    }

    public String show(int a, double d) {
        System.out.println(a);
        return "showed";
    }

    void toggle(String[] args) {
        System.out.println("toggle");
    }

    protected void fade() {
        System.out.println("fade");
    }

    final protected void drag() {
        System.out.println("drag");
    }

    private void hide() {
        System.out.println("hide");
    }

    public static void click() {
        System.out.println("click");
    }

    static void hover() {
        System.out.println("hover");
    }
}
```

```
Processing 'classes.dex'...
Opened 'classes.dex', DEX version '035'
Class #0            -
  Class descriptor  : 'LTest;' // クラス名Test
  Access flags      : 0x0000 () // パッケージプライベート
  Superclass        : 'Ljava/lang/Object;' // スーパークラスがObject
  Interfaces        - // インターフェースを持たない
  Static fields     - // Staticフィールドを持たない
  Instance fields   - // インスタンスフィールドを持たない
  Direct methods    -
    #0              : (in LTest;)
      name          : '<init>' // コンストラクタを示す
      type          : '()V' // 引数なし/戻り値なし
      access        : 0x10000 (CONSTRUCTOR)
      code          -
      registers     : 1
      ins           : 1
      outs          : 1
      insns size    : 4 16-bit code units
0001e4:                                        |[0001e4] Test.<init>:()V
0001f4: 7010 0b00 0000                         |0000: invoke-direct {v0}, Ljava/lang/Object;.<init>:()V // method@000b // Objectのsuper
0001fa: 0e00                                   |0003: return-void
      catches       : (none)
      positions     :
        0x0000 line=1
      locals        :
        0x0000 - 0x0004 reg=0 this LTest;

    #1              : (in LTest;)
      name          : 'click'
      type          : '()V'
      access        : 0x0009 (PUBLIC STATIC)
      code          -
      registers     : 2
      ins           : 0
      outs          : 2
      insns size    : 8 16-bit code units
0001fc:                                        |[0001fc] Test.click:()V
00020c: 6200 0000                              |0000: sget-object v0, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
000210: 1a01 0f00                              |0002: const-string v1, "click" // string@000f
000214: 6e20 0a00 1000                         |0004: invoke-virtual {v0, v1}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@000a
00021a: 0e00                                   |0007: return-void
      catches       : (none)
      positions     :
        0x0000 line=35
      locals        :

    #2              : (in LTest;)
      name          : 'hide'
      type          : '()V'
      access        : 0x0002 (PRIVATE)
      code          -
      registers     : 3
      ins           : 1
      outs          : 2
      insns size    : 8 16-bit code units
00025c:                                        |[00025c] Test.hide:()V
00026c: 6200 0000                              |0000: sget-object v0, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
000270: 1a01 1200                              |0002: const-string v1, "hide" // string@0012
000274: 6e20 0a00 1000                         |0004: invoke-virtual {v0, v1}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@000a
00027a: 0e00                                   |0007: return-void
      catches       : (none)
      positions     :
        0x0000 line=31
      locals        :
        0x0000 - 0x0008 reg=2 this LTest;

    #3              : (in LTest;)
      name          : 'hover'
      type          : '()V'
      access        : 0x0008 (STATIC)
      code          -
      registers     : 2
      ins           : 0
      outs          : 2
      insns size    : 8 16-bit code units
00027c:                                        |[00027c] Test.hover:()V
00028c: 6200 0000                              |0000: sget-object v0, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
000290: 1a01 1300                              |0002: const-string v1, "hover" // string@0013
000294: 6e20 0a00 1000                         |0004: invoke-virtual {v0, v1}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@000a
00029a: 0e00                                   |0007: return-void
      catches       : (none)
      positions     :
        0x0000 line=39
      locals        :

  Virtual methods   -
    #0              : (in LTest;)
      name          : 'drag'
      type          : '()V'
      access        : 0x0014 (PROTECTED FINAL)
      code          -
      registers     : 3
      ins           : 1
      outs          : 2
      insns size    : 8 16-bit code units
00021c:                                        |[00021c] Test.drag:()V
00022c: 6200 0000                              |0000: sget-object v0, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
000230: 1a01 1000                              |0002: const-string v1, "drag" // string@0010
000234: 6e20 0a00 1000                         |0004: invoke-virtual {v0, v1}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@000a
00023a: 0e00                                   |0007: return-void
      catches       : (none)
      positions     :
        0x0000 line=27
      locals        :
        0x0000 - 0x0008 reg=2 this LTest;

    #1              : (in LTest;)
      name          : 'fade'
      type          : '()V'
      access        : 0x0004 (PROTECTED)
      code          -
      registers     : 3
      ins           : 1
      outs          : 2
      insns size    : 8 16-bit code units
00023c:                                        |[00023c] Test.fade:()V
00024c: 6200 0000                              |0000: sget-object v0, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
000250: 1a01 1100                              |0002: const-string v1, "fade" // string@0011
000254: 6e20 0a00 1000                         |0004: invoke-virtual {v0, v1}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@000a
00025a: 0e00                                   |0007: return-void
      catches       : (none)
      positions     :
        0x0000 line=23
      locals        :
        0x0000 - 0x0008 reg=2 this LTest;

    #2              : (in LTest;)
      name          : 'main'
      type          : '()V'
      access        : 0x0001 (PUBLIC)
      code          -
      registers     : 4
      ins           : 1
      outs          : 4
      insns size    : 32 16-bit code units
00029c:                                        |[00029c] Test.main:()V
0002ac: 1210                                   |0000: const/4 v0, #int 1 // #1
0002ae: 1901 2440                              |0001: const-wide/high16 v1, #long 4621819117588971520 // #4024
0002b2: 6e40 0700 0321                         |0003: invoke-virtual {v3, v0, v1, v2}, LTest;.show:(ID)Ljava/lang/String; // method@0007
0002b8: 2300 0800                              |0006: new-array v0, v0, [Ljava/lang/String; // type@0008
0002bc: 1201                                   |0008: const/4 v1, #int 0 // #0
0002be: 1a02 0000                              |0009: const-string v2, "" // string@0000
0002c2: 4d02 0001                              |000b: aput-object v2, v0, v1
0002c6: 6e20 0800 0300                         |000d: invoke-virtual {v3, v0}, LTest;.toggle:([Ljava/lang/String;)V // method@0008
0002cc: 6e10 0300 0300                         |0010: invoke-virtual {v3}, LTest;.fade:()V // method@0003
0002d2: 6e10 0200 0300                         |0013: invoke-virtual {v3}, LTest;.drag:()V // method@0002
0002d8: 7010 0400 0300                         |0016: invoke-direct {v3}, LTest;.hide:()V // method@0004
0002de: 7100 0100 0000                         |0019: invoke-static {}, LTest;.click:()V // method@0001
0002e4: 7100 0500 0000                         |001c: invoke-static {}, LTest;.hover:()V // method@0005
0002ea: 0e00                                   |001f: return-void
      catches       : (none)
      positions     :
        0x0003 line=3
        0x0006 line=4
        0x000d line=5
        0x0010 line=6
        0x0013 line=7
        0x0016 line=8
        0x0019 line=9
        0x001c line=10
      locals        :
        0x0000 - 0x0020 reg=3 this LTest;

    #3              : (in LTest;)
      name          : 'show'
      type          : '(ID)Ljava/lang/String;'
      access        : 0x0001 (PUBLIC)
      code          -
      registers     : 4
      ins           : 4
      outs          : 2
      insns size    : 8 16-bit code units
0001c4:                                        |[0001c4] Test.show:(ID)Ljava/lang/String;
0001d4: 6202 0000                              |0000: sget-object v2, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
0001d8: 6e20 0900 1200                         |0002: invoke-virtual {v2, v1}, Ljava/io/PrintStream;.println:(I)V // method@0009
0001de: 1a01 1800                              |0005: const-string v1, "showed" // string@0018
0001e2: 1101                                   |0007: return-object v1
      catches       : (none)
      positions     :
        0x0000 line=14
      locals        :
        0x0000 - 0x0008 reg=0 this LTest;

    #4              : (in LTest;)
      name          : 'toggle'
      type          : '([Ljava/lang/String;)V'
      access        : 0x0000 ()
      code          -
      registers     : 3
      ins           : 2
      outs          : 2
      insns size    : 8 16-bit code units
0002ec:                                        |[0002ec] Test.toggle:([Ljava/lang/String;)V
0002fc: 6202 0000                              |0000: sget-object v2, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
000300: 1a00 1900                              |0002: const-string v0, "toggle" // string@0019
000304: 6e20 0a00 0200                         |0004: invoke-virtual {v2, v0}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@000a
00030a: 0e00                                   |0007: return-void
      catches       : (none)
      positions     :
        0x0000 line=19
      locals        :
        0x0000 - 0x0008 reg=1 this LTest;

  source_file_idx   : 10 (Test.java)
```

- finalメソッドのdragがvirtualなのがなぜなのかわからない
  - finalはdirectメソッドなのでは?
