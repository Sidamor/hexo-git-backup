---
title: JVM调优 Vol.1 
date: 2017-06-26 16:09:42
categories: 
- JVM
tags:
- Java
- JVM
---

打开jdk安装目录下的bin目录，除了java，javac这些命令，我们还会看到很多。比如：jps, jstat, jinfo, jmap, jhat, jstack等等。这些都是深入探索JVM的工具。今天我们先着重探讨一下jps和jstat这两个命令。

<!--more-->

# jps
和linux下的ps指令相似，jps用来查看jvm虚拟机下的进程状态。
		jps[options][hostid] 


## [options]选项 
-q：仅输出VM标识符，不包括classname,jar name,arguments in main method 
-m：输出传递给主类main()函数的参数 
-l：输出完全的包名，应用主类名，jar的完全路径名 
-v：输出虚拟机进程启动时jvm参数 

## [hostid]：
jps可以通过RMI协议查询开启了RMI服务的远程虚拟机进程状态，hostid为RMI注册表中注册的主机名。
[protocol:][[//]hostname][:port][/servername]


## e.g.

		root@iZ23u31elifZ:~# jps -lv  
		768 XNYLOCALAppSvr.Main  
		1633 DPPLOCALAlertCenter.Main  
		739 BSTRMI.Main  
		761 XNYLOCALRMI.Main  
		749 DPPLOCALAppSvr.Main  
		2429 sun.tools.jps.Jps -Denv.class.path=.:/usr/local/jdk_64/lib/dt.jar:/usr/local/jdk_64/lib/tools.jar -Dapplication.home=/usr/local/jdk_64 -Xms8m  
		750 DPPLOCALRMI.Main  
		1631 org.apache.catalina.startup.Bootstrap -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms300m -Xmx500m -XX:MaxNewSize=128m -XX:PermSize=128M -XX:MaxPermSize=256m -XX:+UseParallelOldGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/log/tomcat/dumpfile/heap.bin -Xloggc:/var/log/tomcat/logs/gc.log -Djdk.tls.ephemeralDHKeySize=2048 -Djava.endorsed.dirs=/usr/local/tomcat/endorsed -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp  

以上，显示了进程id，完整的包名，以及启动时jvm参数

# jstat

语法结构：

		Usage: jstat -help|-options
			jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
			
参数解释：

		Options — 选项，我们一般使用 -gcutil 查看gc情况
		vmid    — VM的进程号，即当前运行的java进程号
		interval– 间隔时间，单位为秒或者毫秒
		count   — 打印次数，如果缺省则打印无数次
		
S0  — Heap上的 Survivor space 0 区已使用空间的百分比
S1  — Heap上的 Survivor space 1 区已使用空间的百分比
E   — Heap上的 Eden space 区已使用空间的百分比
O   — Heap上的 Old space 区已使用空间的百分比
P   — Perm space 区已使用空间的百分比；jdk1.8之后对于方法区的实现改为M(MetaSpace)
CCS — 压缩使用比例
YGC — 从应用程序启动到采样时发生 Young GC 的次数
YGCT– 从应用程序启动到采样时 Young GC 所用的时间(单位秒)
FGC — 从应用程序启动到采样时发生 Full GC 的次数
FGCT– 从应用程序启动到采样时 Full GC 所用的时间(单位秒)
GCT — 从应用程序启动到采样时用于垃圾回收的总时间(单位秒)

		root@iZ23u31elifZ:~# jstat -gcutil 1631 1000 10
		  S0     S1     E      O      M     CCS      YGC    YGCT    FGC    FGCT     GCT   
		  0.00   0.00   0.00  23.38  94.84  87.64    118    1.049    94    8.640    9.689
		  0.00  53.62   0.00  23.44  94.84  87.64    123    1.064    99    9.147   10.212
		  3.12   0.00   0.00  23.45  94.84  87.64    130    1.087   106    9.774   10.861
		  0.00   2.30   0.00  23.48  94.84  87.64    133    1.109   108    9.953   11.062
		  6.76   0.00   3.26  21.07  94.84  87.64    134    1.119   108   10.046   11.165
		  6.76   0.00   6.29  21.07  94.84  87.64    134    1.119   108   10.046   11.165
		  6.76   0.00   6.29  21.07  94.84  87.64    134    1.119   108   10.046   11.165
		  6.76   0.00   6.29  21.07  94.84  87.64    134    1.119   108   10.046   11.165
		  6.76   0.00   6.41  21.07  94.84  87.64    134    1.119   108   10.046   11.165
		  6.76   0.00   6.41  21.07  94.84  87.64    134    1.119   108   10.046   11.165

以上，10秒打印记录（一秒一次）。
