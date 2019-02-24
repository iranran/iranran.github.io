---
layout: post
title: Linux上的free命令详解
cnavigation: True
categories: 
- Linux
feature_image: "https://picsum.photos/2560/600?image=872"
---

本文简要的说明Linux free命令中的buffer和cached是怎么回事。最近闲来无事，在应用服务器上执行了一下 free -m,出来下面的内容。

```
free -m

            total       used       free     shared    buffers     cached
Mem:         15951      15647        303          0        323       9708
-/+ buffers/cache:       5615      10335
Swap:         4031         40       3991
```


一看不对内存只剩303M，这是要死的节奏呀。于是乎上网查了一下此命令，才发现之前一直误解used、free这几个值的意思。

* Mem这行是操作系统认为已使用和剩余的内存值
* -/+ buffers/cache: 这个是实际使用的内存值
* A buffer is something that has yet to be "written" to disk. 
* A cache is something that has been "read" from the disk and stored for later use.
* buffer是用于存放要输出到disk（块设备）的数据的，而cache是存放从disk上读出的数据。这二者是为了提高IO性能的,弥补快速的内存和磁盘之间的速度差。
* 当内存空间不够用的时候，会从buffer、cached中再拿回来，给对应的进程使用。

参考：

[Linux上的free命令详解](http://www.cnblogs.com/coldplayerest/archive/2010/02/20/1669949.html)