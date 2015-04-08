+++
date = "2015-04-07T00:00:00Z"
title = "[ERROR] InnoDB: Cannot allocate memory for the buffer pool の対処法"
tags = ["database", "mysql"]
blogimport = true
type = "post"
+++

上記のエラーが出たらメモリ不足なので, スワップ領域を作るのが良いと思います.

```
## swap領域の確認
$ swapon -s
Filename                Type        Size    Used    Priority

## swap領域の確保
$ sudo dd if=/dev/zero of=/swapfile bs=1024 count=1024k

## swap領域の作成
$ sudo mkswap /swapfile

## swap領域の割り当て
$ sudo swapon /swapfile

$ swapon -s
Filename                Type        Size    Used    Priority
/swapfile                               file        1048572 4320    -1

$ free
             total       used       free     shared    buffers     cached
Mem:        760132     689248      70884       4672       7740      48944
-/+ buffers/cache:     632564     127568
Swap:      1048572      88556     960016

## 再起動時にもswapを割り当てるように設定
$ sudo vi /etc/fstab
```

これでおｋ.


## まとめ

パフォーマンス的にはスワップ領域を確保するのではなく, メモリを増設したほうが良いと思います.
