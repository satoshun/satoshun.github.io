+++
date = "2016-07-17T00:00:00Z"
title = "Linux perf: CPU編"
tags = ["linux"]
blogimport = true
type = "post"
draft = false
+++

# Linux perf: CPU編

CPU利用率, 高負荷なプロセスの特定の仕方について説明します.


## 0. lscpu

システムのコア数,マシンのスペックを最初に調べます.

```shell
root@vagrant:~# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 69
Model name:            Intel(R) Core(TM) i7-4650U CPU @ 1.70GHz
Stepping:              1
CPU MHz:               2300.000
BogoMIPS:              4600.00
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              4096K
NUMA node0 CPU(s):     0-3
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm
```

コア数は4, Core i7ということが分かります.


## 1. uptime

次に, 何はともあれ`uptime`です. ロードアベレージを出力します.

ロードアベレージの値で, ざっくりとシステム全体の負荷を調べることが出来ます.

```shell
root@vagrant:~# uptime
 10:28:54 up 25 min,  2 users,  load average: 0.41, 0.87, 0.59
```

一般的には, CPUのコア数前後が基準となります.
この値が大きい場合は, CPU or メモリ or etcに, 高負荷なポイントがあります.


## 2. mpstat

システム全体のCPU利用率を`mpstat`コマンドで調べます.

個別のコアごとの利用率を出力する `-P ALL` オプションをつけて実行します.

```shell
root@vagrant:~# mpstat -P ALL
Linux 4.4.0-21-generic (vagrant)  07/17/2016  _x86_64_  (4 CPU)

10:27:44 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
10:27:45 AM  all    6.50    0.00   18.25    0.00    0.00    0.00    0.00    0.00    0.00   75.25
10:27:45 AM    0    0.00    0.00    0.98    0.00    0.00    0.00    0.00    0.00    0.00   99.02
10:27:45 AM    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
10:27:45 AM    2    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
10:27:45 AM    3   25.25    0.00   74.75    0.00    0.00    0.00    0.00    0.00    0.00    0.00
```

この出力から, 1つのコアに処理が集中していることが分かります.
また, iowaitが発生していないことから, CPUドリブンなプロセスということも分かります.


## 3. top

次に, どのプロセスがCPUを専有しているのかを`top`コマンドで調べます.

```shell
top - 10:46:54 up 43 min,  2 users,  load average: 0.99, 0.67, 0.48
Tasks: 148 total,   2 running, 146 sleeping,   0 stopped,   0 zombie
%Cpu(s):  7.4 us, 17.7 sy,  0.0 ni, 74.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  4046448 total,  3670180 free,    98452 used,   277816 buff/cache
KiB Swap:  1048572 total,  1048572 free,        0 used.  3895064 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 2682 root      20   0    7620    692    616 R  99.7  0.0   4:38.47 dd
 2195 root      20   0  101016  32260   9380 S   2.3  0.8   0:43.51 glances
 1019 root      20   0  409164  35092  21860 S   0.3  0.9   0:01.99 docker
 1123 www-data  20   0  125416   3148   1552 S   0.3  0.1   0:00.20 nginx
 1367 vagrant   20   0   95464   5128   4180 S   0.3  0.1   0:01.71 sshd
    1 root      20   0   37968   6040   4008 S   0.0  0.1   0:01.96 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kthreadd
...
```

pid 2682の`dd`コマンドがCPUを専有していることが分かります.
これで, 負荷が高いプロセスを特定できました.


## 4. strace

次に, 特定したプロセスがどういったシステムコールを使っているのかを`strace`コマンドで調べます.

```shell
root@vagrant:~# strace -p 2682
strace: Process 1376 attached
write(1, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
read(0, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
write(1, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
read(0, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
write(1, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
read(0, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
write(1, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
read(0, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
write(1, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 512) = 512
...
```

ddコマンドは, read, writeシステムコールを繰り返していることが分かります.

余談ですが, nginxのworkerプロセスをstraceすると, 以下の出力をします.

```shell
root@vagrant:~# strace -p 1118
strace: Process 1118 attached
epoll_wait(9,
[{EPOLLIN, {u32=1012871416, u64=140064191361272}}], 512, -1) = 1
accept4(7, {sa_family=AF_INET6, sin6_port=htons(43248), inet_pton(AF_INET6, "::1", &sin6_addr), sin6_flowinfo=0, sin6_scope_id=0}, [28], SOCK_NONBLOCK) = 13
epoll_ctl(9, EPOLL_CTL_ADD, 13, {EPOLLIN|EPOLLRDHUP|EPOLLET, {u32=1012871880, u64=140064191361736}}) = 0
epoll_wait(9, [{EPOLLIN, {u32=1012871880, u64=140064191361736}}], 512, 60000) = 1
recvfrom(13, "GET / HTTP/1.1\r\nHost: localhost\r"..., 1024, 0, NULL, NULL) = 73
stat("/var/www/html/", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
stat("/var/www/html/", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
stat("/var/www/html/index.html", 0x7fffe02ba480) = -1 ENOENT (No such file or directory)
stat("/var/www/html", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
stat("/var/www/html/index.htm", 0x7fffe02ba480) = -1 ENOENT (No such file or directory)
stat("/var/www/html/index.nginx-debian.html", {st_mode=S_IFREG|0644, st_size=612, ...}) = 0
stat("/var/www/html/index.nginx-debian.html", {st_mode=S_IFREG|0644, st_size=612, ...}) = 0
open("/var/www/html/index.nginx-debian.html", O_RDONLY|O_NONBLOCK) = 14
fstat(14, {st_mode=S_IFREG|0644, st_size=612, ...}) = 0
setsockopt(13, SOL_TCP, TCP_CORK, [1], 4) = 0
writev(13, [{"HTTP/1.1 200 OK\r\nServer: nginx/1"..., 247}], 1) = 247
sendfile(13, 14, [0] => [612], 612)     = 612
write(4, "::1 - - [17/Jul/2016:11:27:21 +0"..., 80) = 80
close(14)                               = 0
setsockopt(13, SOL_TCP, TCP_CORK, [0], 4) = 0
recvfrom(13, 0x55583aa2a770, 1024, 0, NULL, NULL) = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(9, [{EPOLLIN|EPOLLRDHUP, {u32=1012871880, u64=140064191361736}}], 512, 65000) = 1
recvfrom(13, "", 1024, 0, NULL, NULL)   = 0
close(13)                               = 0
epoll_wait(9,
```

ざっくりと,

1. accept4でリクエストをacceptする
2. recvfromで, リクエストの中身を取得する
3. /var/www/html/index.nginx-debian.htmlを開く
4. writev, sendfile, writeでレスポンスを作成する
5. closeで後処理をする

という感じでしょうか?
`strace`コマンドを使うことで, 大体の処理内容が分かります.


## 5. perf

最後に, `perf`コマンドを使って, プロセスのCPU利用率のグラフを見ます.

まず, pidを指定して, プロファイルを作成します.

```shell
perf record -F 99 -p 2682 -a -g -- sleep 10
```

そして, プロファイリングの結果を出力します.

```shell
root@vagrant:~# perf report --stdio
no symbols found in /bin/dd, maybe install a debug package?
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 985  of event 'cpu-clock'
# Event count (approx.): 9949494850
#
# Children      Self  Command  Shared Object      Symbol
# ........  ........  .......  .................  .................................
#
    59.80%     0.30%  dd       [kernel.kallsyms]  [k] entry_SYSCALL_64_fastpath
                 |
                 |--59.49%-- entry_SYSCALL_64_fastpath
                 |          |
                 |          |--39.59%-- sys_read
                 |          |          |
                 |          |          |--34.72%-- vfs_read
                 |          |          |          |
                 |          |          |          |--19.29%-- __vfs_read
                 |          |          |          |          |
                 |          |          |          |          |--17.87%-- new_sync_read
                 |          |          |          |          |          |
                 |          |          |          |          |          |--14.72%-- read_iter_zero
                 |          |          |          |          |          |          |
                 |          |          |          |          |          |          |--12.49%-- iov_iter_zero
                 |          |          |          |          |          |          |          |
                 |          |          |          |          |          |          |           --10.05%-- __clear_user
                 |          |          |          |          |          |          |
                 |          |          |          |          |          |          |--0.91%-- __clear_user
                 |          |          |          |          |          |          |
                 |          |          |          |          |          |           --0.10%-- apic_timer_interrupt
                 |          |          |          |          |          |                     smp_apic_timer_interrupt
                 |          |          |          |          |          |                     irq_exit
                 |          |          |          |          |          |                     __do_softirq
                 |          |          |          |          |          |
                 |          |          |          |          |          |--0.30%-- iov_iter_zero
                 |          |          |          |          |          |
                 |          |          |          |          |          |--0.30%-- iov_iter_init
                 |          |          |          |          |          |
                 |          |          |          |          |           --0.20%-- _cond_resched
                 |          |          |          |          |
                 |          |          |          |          |--0.91%-- read_iter_zero
                 |          |          |          |          |
                 |          |          |          |           --0.20%-- iov_iter_init
                 |          |          |          |
                 |          |          |          |--10.56%-- rw_verify_area
...
```

`perf`コマンドを使うことで, より詳細なプロセスの内容が分かります.


## その他

- htop: topのいい感じ版
- glances: topのいい感じ版
- sar: CPU + メモリの使用率を調べる
