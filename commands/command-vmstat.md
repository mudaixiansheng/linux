<div class="BlogAnchor">
   <p>
   <b id="AnchorContentToggle" title="收起" style="cursor:pointer;">目录[+]</b>
   </p>
   
  <div class="AnchorContent" id="AnchorContent"> </div>
</div>


# linux命令之vmstat

## 1、vmstat命令简介

vmstat命令是最常见的Linux/Unix监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。相比top，我可以看到整个机器的CPU,内存,IO的使用情况，而不是单单看到各个进程的CPU使用率和内存使用率(使用场景不一样)。

### 1.1 语法

	vmstat(选项)(参数)

- -a：显示活动内页；
- -f：显示启动后创建的进程总数；
- -m：显示slab信息；
- -n：头信息仅显示一次；
- -s：以表格方式显示事件计数器和内存状态；
- -d：报告磁盘状态；
- -p：显示指定的硬盘分区状态；
- -S：输出信息的单位。


一般vmstat工具的使用是通过两个数字参数来完成的，第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数。

	[root@localhost sysshell]# vmstat 2 3
	procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
	 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
	 1  0 1694780 213608      0 448844    6    8    11    39    1    5  4  1 93  3  0
	 0  0 1694776 213576      0 448872    0    0     0    14  297  403  4  0 96  0  0
	 0  0 1694776 211236      0 448816    0    0     0     0  874 1113  1  2 97  0  0

这表示vmstat每2秒采集数据，采集3次结束，当然也可以不指定次数最后手动结束程序。

### 1.2 参数讲解

**1、Process（进程）**

- r：运行队列，指多少个进程真正分配到了CPU，这个数值如果超过了CPU数目，就会出现CPU瓶颈，一般负载超过3比较高，超过5就高，10之上就不正常。运行队列中进程数量，这个值也可以判断是否需要增加CPU。（长期大于1）
- b：表示阻塞的进程数量。

**2、Memory（内存）**

- swap：虚拟内存已经使用的大小，如果大于0，说明物理机的内存不足，如果不是程序内存泄漏，说明需要内存升级或者将内存消耗任务转移。
- free：空闲的物理内存大小。
- buff：系统用来存储目录内容、权限等缓存。
- cache：cache用于记忆我们打开的文件，给文件做缓存。将空闲的物理内存一部分拿来做文件和目录的缓存，是为了提高系统的性能，当程序使用内存时，buff/cache会很快的被使用。

**3、Swap**

- si：每秒从磁盘读入虚拟内存的大小，值大于0，说明内存不够用。
- so：每秒虚拟内存写入磁盘的大小。

注意：内存够用的时候，这2个值都是0，如果这2个值长期大于0时，系统性能会受到影响，磁盘IO和CPU资源都会被消耗。有些朋友看到空闲内存（free）很少的或接近于0时，就认为内存不够用了，不能光看这一点，还要结合si和so，如果free很少，但是si和so也很少（大多时候是0），那么不用担心，系统性能这时不会受到影响的。

**4、IO（现在的Linux版本块的大小为1kb）**

- bi：块设备每秒接收的块数量，这里的块设备指系统上所有的磁盘和其它的块设备，默认块大小为1024byte。
- bo：块设备每秒发送的块数量。bi、bo一般要接近0，不然就是io频繁。

注意：随机磁盘读写的时候，这2个值越大（如超出1024k)，能看到CPU在IO等待的值也会越大。

**5、system（系统）**

- in：每秒cpu中断的次数，包括时间中断。
- cs：每秒切换上下文的次数。例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目,例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的。

**6、CPU（以百分比表示）**

- us：用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。
- sy：系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。
- id：空闲 CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。
- wt：等待IO CPU时间。