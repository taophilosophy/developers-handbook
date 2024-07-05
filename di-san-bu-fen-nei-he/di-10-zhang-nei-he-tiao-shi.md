# 第 10 章 内核调试

## 10.1. 获取内核崩溃转储

运行开发内核（例如 FreeBSD-CURRENT）时，例如在极端条件下的内核（例如非常高的负载平均值，成千上万的连接，极高数量的并发用户，数百个jail（8）等），或者在 FreeBSD-STABLE 上使用新功能或设备驱动程序（例如 PAE），有时内核会发生崩溃。如果发生崩溃，本章将演示如何从崩溃中提取有用信息。

一旦内核发生崩溃，系统重启是不可避免的。一旦系统重新启动，系统的物理内存（RAM）的内容将丢失，以及在崩溃之前位于交换设备上的任何位。为了保留物理内存中的位，内核利用交换设备作为在崩溃后跨重启存储在 RAM 中的位的临时位置。通过这种方式，当 FreeBSD 在崩溃后启动时，现在可以提取内核映像，并进行调试。

|  | 配置为转储设备的交换设备仍然作为交换设备。目前不支持转储到非交换设备（例如磁带或 CDRW）。"交换设备" 与 "交换分区" 是同义词。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------ |

可用几种类型的内核崩溃转储：

完整内存转储保存物理内存的完整内容。

迷你转储仅保存内核正在使用的内存页面（FreeBSD 6.2 及更高版本）。

TextdumpsHold 捕获、脚本化或交互式调试器输出（FreeBSD 7.1 及更高版本）。

Minidumps 是 FreeBSD 7.0 的默认转储类型，在大多数情况下将捕获完整内存转储中存在的所有必要信息，因为大多数问题只能使用内核状态来隔离。

### 10.1.1. 配置转储设备

内核将转储其物理内存的内容到转储设备之前，必须配置转储设备。 使用 dumpon(8)命令指定转储设备，告诉内核在哪里保存内核崩溃转储。 在交换分区使用 swapon(8)配置后，必须调用 dumpon(8)程序。 通常通过在 rc.conf(5)中设置 dumpdev 变量为交换设备路径（提取内核转储的推荐方式）或 AUTO 以使用第一个配置的交换设备来处理。 在 HEAD 中，默认为 dumpdev ，在 RELENG_*分支上更改为 NO （除了 RELENG_7，它保持设置为 AUTO ）。 在 FreeBSD 9.0-RELEASE 和更高版本中，bsdinstall 将在安装过程中询问是否应在目标系统上启用崩溃转储。

|  | 检查/etc/fstab 或 swapinfo(8)以获取交换设备列表。 |
| -- | --------------------------------------------------- |

```
# mkdir /var/crash
# chmod 700 /var/crash
```

此外，请记住/var/crash 的内容是敏感的，很可能包含诸如密码之类的机密信息。

### 10.1.2. 提取内核转储

一旦将转储写入转储设备，必须在挂载交换设备之前提取转储。要从转储设备中提取转储，请使用 savecore(8)程序。如果在 rc.conf(5)中设置了 dumpdev ，则在崩溃后的第一次多用户引导时，将自动调用 savecore(8)，在交换设备挂载之前。提取的核心位置放置在 rc.conf(5)值 dumpdir 中，默认为/var/crash，并将命名为 vmcore.0。

如果/var/crash 中已经存在名为 vmcore.0 的文件（或者设置为 dumpdir 的任何内容），内核将为每次崩溃递增尾随数字，以避免覆盖现有的 vmcore（例如，vmcore.1）。在保存转储后，savecore(8)将始终在/var/crash 中创建一个名为 vmcore.last 的符号链接。此符号链接可用于查找最近转储的名称。

crashinfo(8) 实用程序生成一个文本文件，其中包含来自完整内存转储或迷你转储的信息摘要。如果在 rc.conf(5) 中设置了 dumpdev ，则在 savecore(8) 之后会自动调用 crashinfo(8)。输出保存在名为 core.txt.N 的文件中。

```
# fsck -p
# mount -a -t ufs       # make sure /var/crash is writable
# savecore /var/crash /dev/ad0s1b
# exit                  # exit to multi-user
```

这会指示 savecore(8) 从 /dev/ad0s1b 中提取一个内核转储，并将内容放在 /var/crash 中。不要忘记确保目标目录 /var/crash 有足够的空间来存储转储。还要记得指定正确的交换设备路径，因为它很可能与 /dev/ad0s1b 不同！

### 10.1.3. 测试内核转储配置

内核包含一个 sysctl(8) 节点，用于请求内核崩溃。这可用于验证系统是否正确配置以保存内核崩溃转储。在触发崩溃之前，您可能希望在单用户模式下将现有文件系统重新挂载为只读，以避免数据丢失。

```
# shutdown now
...
Enter full pathname of shell or RETURN for /bin/sh:
# mount -a -u -r
# sysctl debug.kdb.panic=1
debug.kdb.panic:panic: kdb_sysctl_panic
...
```

重新启动后，您的系统应在 /var/crash 中保存一个转储文件，并附有来自 crashinfo(8) 的匹配摘要。

## 10.2. 使用 kgdb 调试内核崩溃转储

|  | 本节介绍了 kgdb(1)。最新版本包含在 devel/gdb 中。在 FreeBSD 11 及更早版本中也有旧版本。 |
| -- | ----------------------------------------------------------------------------------------- |

要进入调试器并开始从转储中获取信息，请启动 kgdb：

```
# kgdb -n N
```

其中 N 是要检查的 vmcore.N 的后缀。要打开最近的转储，请使用：

```
# kgdb -n last
```

通常情况下，kgdb(1) 应该能够定位在生成转储时运行的内核。如果无法定位正确的内核，请将内核的路径名和转储作为两个参数传递给 kgdb：

```
# kgdb /boot/kernel/kernel /var/crash/vmcore.0
```

您可以像调试任何其他程序一样使用内核源代码来调试崩溃转储。

这个转储文件来自于一个 5.2-BETA 内核，崩溃是在内核深处发生的。下面的输出已经经过修改，在左侧包含了行号。这个第一次跟踪检查指令指针并获得一个回溯。在第 41 行使用的地址已经在第 17 行找到，用于 list 命令的指令指针。如果您无法自行调试出问题，大多数开发人员会要求至少要将这些信息发送给他们。但是，如果您解决了问题，请确保您的补丁通过问题报告、邮件列表或能够提交它的方式进入源码树！

```
 1:# cd /usr/obj/usr/src/sys/KERNCONF
 2:# kgdb kernel.debug /var/crash/vmcore.0
 3:GNU gdb 5.2.1 (FreeBSD)
 4:Copyright 2002 Free Software Foundation, Inc.
 5:GDB is free software, covered by the GNU General Public License, and you are
 6:welcome to change it and/or distribute copies of it under certain conditions.
 7:Type "show copying" to see the conditions.
 8:There is absolutely no warranty for GDB.  Type "show warranty" for details.
 9:This GDB was configured as "i386-undermydesk-freebsd"...
10:panic: page fault
11:panic messages:
12:---
13:Fatal trap 12: page fault while in kernel mode
14:cpuid = 0; apic id = 00
15:fault virtual address   = 0x300
16:fault code:             = supervisor read, page not present
17:instruction pointer     = 0x8:0xc0713860
18:stack pointer           = 0x10:0xdc1d0b70
19:frame pointer           = 0x10:0xdc1d0b7c
20:code segment            = base 0x0, limit 0xfffff, type 0x1b
21:                        = DPL 0, pres 1, def32 1, gran 1
22:processor eflags        = resume, IOPL = 0
23:current process         = 14394 (uname)
24:trap number             = 12
25:panic: page fault
26      cpuid = 0;
27:Stack backtrace:
28
29:syncing disks, buffers remaining... 2199 2199 panic: mi_switch: switch in a critical section
30:cpuid = 0;
31:Uptime: 2h43m19s
32:Dumping 255 MB
33: 16 32 48 64 80 96 112 128 144 160 176 192 208 224 240
34:---
35:Reading symbols from /boot/kernel/snd_maestro3.ko...done.
36:Loaded symbols for /boot/kernel/snd_maestro3.ko
37:Reading symbols from /boot/kernel/snd_pcm.ko...done.
38:Loaded symbols for /boot/kernel/snd_pcm.ko
39:#0  doadump () at /usr/src/sys/kern/kern_shutdown.c:240
40:240             dumping++;
41:(kgdb) list *0xc0713860
42:0xc0713860 is in lapic_ipi_wait (/usr/src/sys/i386/i386/local_apic.c:663).
43:658                     incr = 0;
44:659                     delay = 1;
45:660             } else
46:661                     incr = 1;
47:662             for (x = 0; x < delay; x += incr) {
48:663                     if ((lapic->icr_lo & APIC_DELSTAT_MASK) == APIC_DELSTAT_IDLE)
49:664                             return (1);
50:665                     ia32_pause();
51:666             }
52:667             return (0);
53:(kgdb) backtrace
54:#0  doadump () at /usr/src/sys/kern/kern_shutdown.c:240
55:#1  0xc055fd9b in boot (howto=260) at /usr/src/sys/kern/kern_shutdown.c:372
56:#2  0xc056019d in panic () at /usr/src/sys/kern/kern_shutdown.c:550
57:#3  0xc0567ef5 in mi_switch () at /usr/src/sys/kern/kern_synch.c:470
58:#4  0xc055fa87 in boot (howto=256) at /usr/src/sys/kern/kern_shutdown.c:312
59:#5  0xc056019d in panic () at /usr/src/sys/kern/kern_shutdown.c:550
60:#6  0xc0720c66 in trap_fatal (frame=0xdc1d0b30, eva=0)
61:    at /usr/src/sys/i386/i386/trap.c:821
62:#7  0xc07202b3 in trap (frame=
63:      {tf_fs = -1065484264, tf_es = -1065484272, tf_ds = -1065484272, tf_edi = 1, tf_esi = 0, tf_ebp = -602076292, tf_isp = -602076324, tf_ebx = 0, tf_edx = 0, tf_ecx = 1000000, tf_eax = 243, tf_trapno = 12, tf_err = 0, tf_eip = -1066321824, tf_cs = 8, tf_eflags = 65671, tf_esp = 243, tf_ss = 0})
64:    at /usr/src/sys/i386/i386/trap.c:250
65:#8  0xc070c9f8 in calltrap () at {standard input}:94
66:#9  0xc07139f3 in lapic_ipi_vectored (vector=0, dest=0)
67:    at /usr/src/sys/i386/i386/local_apic.c:733
68:#10 0xc0718b23 in ipi_selected (cpus=1, ipi=1)
69:    at /usr/src/sys/i386/i386/mp_machdep.c:1115
70:#11 0xc057473e in kseq_notify (ke=0xcc05e360, cpu=0)
71:    at /usr/src/sys/kern/sched_ule.c:520
72:#12 0xc0575cad in sched_add (td=0xcbcf5c80)
73:    at /usr/src/sys/kern/sched_ule.c:1366
74:#13 0xc05666c6 in setrunqueue (td=0xcc05e360)
75:    at /usr/src/sys/kern/kern_switch.c:422
76:#14 0xc05752f4 in sched_wakeup (td=0xcbcf5c80)
77:    at /usr/src/sys/kern/sched_ule.c:999
78:#15 0xc056816c in setrunnable (td=0xcbcf5c80)
79:    at /usr/src/sys/kern/kern_synch.c:570
80:#16 0xc0567d53 in wakeup (ident=0xcbcf5c80)
81:    at /usr/src/sys/kern/kern_synch.c:411
82:#17 0xc05490a8 in exit1 (td=0xcbcf5b40, rv=0)
83:    at /usr/src/sys/kern/kern_exit.c:509
84:#18 0xc0548011 in sys_exit () at /usr/src/sys/kern/kern_exit.c:102
85:#19 0xc0720fd0 in syscall (frame=
86:      {tf_fs = 47, tf_es = 47, tf_ds = 47, tf_edi = 0, tf_esi = -1, tf_ebp = -1077940712, tf_isp = -602075788, tf_ebx = 672411944, tf_edx = 10, tf_ecx = 672411600, tf_eax = 1, tf_trapno = 12, tf_err = 2, tf_eip = 671899563, tf_cs = 31, tf_eflags = 642, tf_esp = -1077940740, tf_ss = 47})
87:    at /usr/src/sys/i386/i386/trap.c:1010
88:#20 0xc070ca4d in Xint0x80_syscall () at {standard input}:136
89:---Can't read userspace from dump, or kernel process---
90:(kgdb) quit
```

|  | 如果您的系统经常崩溃并且磁盘空间不足，删除/var/crash 中的旧 vmcore 文件可以节省大量磁盘空间！ |
| -- | ----------------------------------------------------------------------------------------------- |

## 10.3. 使用 DDB 进行在线内核调试

虽然 kgdb 作为一种脱机调试器提供了非常高级的用户界面，但它无法做一些事情。最重要的是断点设置和单步执行内核代码。

如果您需要在内核上进行低级调试，可以使用名为 DDB 的在线调试器。它允许设置断点，逐步执行内核函数，检查和更改内核变量等。但是，它无法访问内核源文件，只能访问全局和静态符号，而不能像 kgdb 那样访问完整的调试信息。

要配置内核以包含 DDB，请添加选项

```
options KDB
```

```
options DDB
```

到您的配置文件，并重新构建。（有关配置 FreeBSD 内核的详细信息，请参阅 FreeBSD 手册）。

一旦您的 DDB 内核运行起来，有几种方法可以进入 DDB。第一种，也是最早的方法是使用引导标志 -d 。内核将以调试模式启动，并在任何设备探测之前进入 DDB。因此，您甚至可以调试设备的探测/附加功能。要使用此功能，请退出加载程序的引导菜单，并在加载程序提示符处输入 boot -d 。

第二种情况是在系统引导后立即转到调试器。有两种简单的方法可以实现这一点。如果您想从命令提示符中断到调试器，请简单地键入以下命令：

```
# sysctl debug.kdb.enter=1
```

或者，如果您在系统控制台上，可以使用键盘上的热键。默认的中断到调试器序列是 Ctrl+Alt+ESC。对于 syscons，此序列可以重新映射，一些分发的映射已经这样做了，因此请确保您知道要使用的正确序列。对于允许在控制台线上使用串行线 BREAK 进入 DDB 的串行控制台，有一个选项（ options BREAK_TO_DEBUGGER 在内核配置文件中）。这不是默认选项，因为周围有很多不必要生成 BREAK 条件的串行适配器，例如在拔下电缆时。

第三种方式是，如果内核配置为使用它，任何恐慌条件都将分支到 DDB。出于这个原因，配置一个带有 DDB 的内核不明智，用于运行无人看管的机器。

要获得无人看管功能，请添加：

```
options	KDB_UNATTENDED
```

到内核配置文件中并重新构建/重新安装。

DDB 命令大致类似于某些 gdb 命令。您可能需要做的第一件事情是设置断点:

```
 break function-name address
```

默认情况下，数字被视为十六进制，但为了使它们与符号名称区分开；以字母 a-f 开头的十六进制数需要在前面加上 0x （对于其他数字而言，这是可选的）。可以使用简单的表达式，例如: function-name + 0x103 .

要退出调试器并继续执行，请键入:

```
 continue
```

要获取当前线程的堆栈跟踪，请使用：

```
 trace
```

要获取任意线程的堆栈跟踪，请将进程 ID 或线程 ID 指定为第二个参数 trace 。

如果要删除断点，请使用

```
 del
 del address-expression
```

第一种形式在断点命中后立即被接受，并删除当前断点。第二种形式可以移除任何断点，但您需要指定确切的地址；这可以从中获得：

```
 show b
```

 或：

```
 show break
```

要单步执行内核，请尝试：

```
 s
```

这将进入函数，但你可以让 DDB 追踪它们，直到达到匹配的返回语句：

```
 n
```

|  | 这不同于 gdb 的 next 语句；它像 gdb 的 finish 。多次按 n 将导致继续。 |
| -- | ----------------------------------------------------------------------- |

要从内存中检查数据，请使用（例如）：

```
 x/wx 0xf0133fe0,40
 x/hd db_symtab_space
 x/bc termbuf,10
 x/s stringbuf
```

用于字/半字/字节访问，以及十六进制/十进制/字符/字符串显示。逗号后面的数字是对象计数。要显示接下来的 0x10 个项目，只需使用：

```
 x ,10
```

 同样，使用

```
 x/ia foofunc,10
```

来反汇编 foofunc 的前 0x10 条指令，并显示它们以及它们距离 foofunc 起始处的偏移量。

用 write 命令修改内存：

```
 w/b termbuf 0xa 0xb 0
 w/w 0xf0010030 0 0
```

命令修饰符 ( b / h / w ) 指定要写入的数据的大小，随后的第一个表达式是要写入的地址，其余被解释为要写入连续内存位置的数据。

如果你需要了解当前寄存器，请使用：

```
 show reg
```

或者，您可以通过例如显示单个寄存器值

```
 p $eax
```

并通过修改它

```
 set $eax new-value
```

如果您需要从 DDB 调用一些内核函数，只需说

```
 call func(arg1, arg2, ...)
```

返回值将被打印。

要获取所有运行进程的 ps(1) 风格摘要，请使用：

```
 ps
```

现在您已经检查了内核失败的原因，并希望重新启动。请记住，根据先前故障的严重程度，内核的某些部分可能仍无法正常工作。执行以下操作之一以关闭并重新启动系统：

```
 panic
```

这将导致您的内核转储核心并重新启动，因此您可以稍后使用 kgdb(1)在更高级别上分析核心。

```
 call boot(0)
```

这可能是一个干净关闭运行系统的好方法， sync() 所有磁盘，并最终，在某些情况下，重新启动。只要内核的磁盘和文件系统接口没有损坏，这可能是一个几乎干净的关机方式。

```
 reset
```

这是灾难中的最后一招，几乎与按下大红按钮相同。

如果您需要简短的命令摘要，请输入：

```
 help
```

强烈建议在调试会话中准备打印版本的 ddb(4) 手册页面。请记住，在单步执行内核时很难阅读在线手册。

## 10.4. 使用远程 GDB 进行在线内核调试

FreeBSD 内核为在线调试提供了第二个 KDB 后端：gdb（4）。自 FreeBSD 2.2 以来就支持此功能，实际上非常方便。

GDB 长期以来一直支持远程调试。这是通过串行线路沿着一个非常简单的协议完成的。与上述其他调试方法不同，您需要两台机器来执行此操作。一台是提供调试环境的主机，其中包括所有源代码以及一个包含所有符号的内核二进制副本。另一台是运行完全相同内核副本（可选地删除了调试信息）的目标机器。

要使用远程 GDB，请确保您的内核配置中存在以下选项：

```
makeoptions     DEBUG=-g
options         KDB
options         GDB
```

请注意，在-STABLE 和-RELEASE 分支的内核中， GENERIC 默认情况下关闭了 GDB 选项，但在-CURRENT 上启用。

构建完成后，将内核复制到目标机器，并引导它。将目标机器的串行线连接到调试主机的任何串行线，其中该串行线上的 uart 设备设置为"flags 080"。有关如何设置 uart 设备上标志的信息，请参阅 uart(4)。

目标机器必须使其进入 GDB 后端，要么由于发生紧急情况，要么通过有意诱发陷阱进入调试器。在执行此操作之前，选择 GDB 调试器后端：

```
# sysctl debug.kdb.current=gdb
debug.kdb.current: ddb -> gdb
```

|  | 可以通过 debug.kdb.available sysctl 列出支持的后端。如果内核配置包括 options DDB ，那么默认将选择 ddb(4)。如果 gdb 不出现在可用后端列表中，则调试串行port可能未正确配置。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

然后，强制进入调试器：

```
# sysctl debug.kdb.enter=1
debug.kdb.enter: 0KDB: enter: sysctl debug.kdb.enter
```

目标机现在等待远程 GDB 客户端的连接。在调试机上，前往目标内核的编译目录，并启动 gdb ：

```
# cd /usr/obj/usr/src/amd64.amd64/sys/GENERIC/
# kgdb kernel
GNU gdb (GDB) 10.2 [GDB v10.2 for FreeBSD]
Copyright (C) 2021 Free Software Foundation, Inc.
...
Reading symbols from kernel...
Reading symbols from /usr/obj/usr/src/amd64.amd64/sys/GENERIC/kernel.debug...
(kgdb)
```

通过以下方式启动远程调试会话（假设正在使用第一个串行port）：

```
(kgdb) target remote /dev/cuau0
```

您的主机 GDB 现在将控制目标内核：

```
Remote debugging using /dev/cuau0
kdb_enter (why=<optimized out>, msg=<optimized out>) at /usr/src/sys/kern/subr_kdb.c:506
506                     kdb_why = KDB_WHY_UNSET;
(kgdb)
```

|  | 根据使用的编译器，一些局部变量可能显示为 <optimized out> ，直接通过 gdb 检查可能会有问题。如果这在调试时造成问题，则可以通过向 make(1)传递 COPTFLAGS=-O1 来以降低的优化级别构建内核。然而，当优化级别更改时，某些内核错误类别可能会以不同的方式（或根本不会）显现。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

您几乎可以像使用其他 GDB 会话一样使用此会话，包括完全访问源代码，在 Emacs 窗口中以 gud 模式运行它（这将在另一个 Emacs 窗口中自动显示源代码等）。

## 10.5. 调试控制台驱动程序

由于您需要一个控制台驱动程序来运行 DDB，如果控制台驱动程序本身出现故障，情况就会变得更加复杂。您可能会记得使用串行控制台（要么使用修改后的引导块，要么在提示符处指定 -h ），并将标准终端连接到您的第一个串行port。DDB 可在任何配置的控制台驱动程序上运行，包括串行控制台。

## 10.6. 调试死锁

您可能会遇到所谓的死锁，这是一个系统停止执行有用工作的情况。在这种情况下，为了提供有用的 bug 报告，请使用上一节中描述的 ddb(4)。在报告中包含 ps 和 trace 的疑似进程的输出。

如果可能的话，考虑进行进一步调查。如果您怀疑死锁发生在 VFS 层，请使用下面的方法。将这些选项添加到内核配置文件中。

```
makeoptions 	DEBUG=-g
options 	INVARIANTS
options 	INVARIANT_SUPPORT
options 	WITNESS
options 	WITNESS_SKIPSPIN
options 	DEBUG_LOCKS
options 	DEBUG_VFS_LOCKS
options 	DIAGNOSTIC
```

当发生死锁时，除了执行 ps 命令的输出外，还提供来自 show pcpu ， show allpcpu ， show locks ， show alllocks ， show lockedvnods 和 alltrace 的信息。

要为线程化进程获取有意义的回溯信息，请使用 thread thread-id 切换到线程堆栈，并使用 where 进行回溯。

## 10.7. 使用 Dcons 进行内核调试

dcons(4)是一个非常简单的控制台驱动程序，不直接与任何物理设备连接。它只是从内核或加载程序中的缓冲区读取和写入字符。由于其简单的特性，它在内核调试中非常有用，特别是与 FireWire®设备一起使用。目前，FreeBSD 提供了两种与缓冲区进行外部交互的方式，使用 dconschat(8)。

### 10.7.1. FireWire®上的 Dcons

大多数 FireWire®（IEEE1394）主机控制器都基于支持对主机内存进行物理访问的 OHCI 规范。这意味着一旦主机控制器初始化，我们可以在没有软件（内核）帮助的情况下访问主机内存。我们可以利用这个功能与 dcons(4)进行交互。dcons(4)提供类似于串行控制台的功能。它模拟两个串行端口，一个用于控制台和 DDB，另一个用于 GDB。由于远程内存访问完全由硬件处理，即使系统崩溃，dcons(4)缓冲区也是可访问的。

FireWire® 设备不仅限于集成到主板中。桌面上有 PCI 卡可用，笔记本电脑可以购买 CardBus 接口。

#### 10.7.1.1. 在目标机器上启用 FireWire®和 Dcons 支持

要在目标机器的内核中启用 FireWire®和 Dcons 支持：

* 确保您的内核支持 dcons ， dcons_crom 和 firewire 。 Dcons 应该与内核静态链接。对于 dcons_crom 和 firewire ，模块应该没问题。
* 确保启用物理 DMA。您可能需要将 hw.firewire.phydma_enable=1 添加到 /boot/loader.conf 中。
* 添加用于调试的选项。
* 如果您使用 GDB 在 FireWire®上，请在/boot/loader.conf 中添加 dcons_gdb=1 。
* 在/etc/ttys 中启用 dcons 。
* 可选地，要强制 dcons 成为高级控制台，请将 hw.firewire.dcons_crom.force_console=1 添加到 loader.conf 中。

在 i386 或 amd64 上启用 FireWire® 和 Dcons 支持，请在 loader(8) 中添加以下内容：

在 /etc/make.conf 中添加 LOADER_FIREWIRE_SUPPORT=YES 并重新构建 loader(8)：

```
# cd /sys/boot/i386 && make clean && make && make install
```

要将 dcons(4) 作为主动低级控制台启用，请将 boot_multicons="YES" 添加到 /boot/loader.conf。

这里有一些配置示例。一个样本内核配置文件将包含：

```
device dcons
device dcons_crom
options KDB
options DDB
options GDB
options ALT_BREAK_TO_DEBUGGER
```

一个样本 /boot/loader.conf 将包含：

```
dcons_crom_load="YES"
dcons_gdb=1
boot_multicons="YES"
hw.firewire.phydma_enable=1
hw.firewire.dcons_crom.force_console=1
```

#### 10.7.1.2. 在主机上启用 FireWire® 和 Dcons 支持

在主机上的内核中启用 FireWire® 支持：

```
# kldload firewire
```

找出 FireWire® 主机控制器的唯一 64 位标识符（EUI64），并使用 fwcontrol(8) 或 dmesg 找到目标机器的 EUI64。

 运行 dconschat(8)，使用：

```
# dconschat -e \# -br -G 12345 -t 00-11-22-33-44-55-66-77
```

一旦 dconschat(8) 运行，可以使用以下组合键：

| \~+. | 断开连接       |
| --------- | ---------------- |
| \~   | ALT 断开       |
| \~   | 重置目标       |
| \~   | 挂起 dconschat |

使用远程调试会话启动 kgdb(1)以附加远程 GDB：

```
 kgdb -r :12345 kernel
```

#### 10.7.1.3. 一些一般提示

这里有一些一般提示：

利用 FireWire® 的速度优势，禁用其他慢速控制台驱动程序：

```
# conscontrol delete ttyd0	     # serial console
# conscontrol delete consolectl	# video/keyboard
```

emacs(1) 存在一个 GDB 模式；这是您需要添加到您的 .emacs 的内容：

```
(setq gud-gdba-command-name "kgdb -a -a -a -r :12345")
(setq gdb-many-windows t)
(xterm-mouse-mode 1)
M-x gdba
```

以及 DDD（devel/ddd）：

```
# remote serial protocol
LANG=C ddd --debugger kgdb -r :12345 kernel
# live core debug
LANG=C ddd --debugger kgdb kernel /dev/fwmem0.2
```

### 10.7.2. 使用 KVM 的 Dcons

我们可以直接通过 /dev/mem 读取 dcons(4) 缓冲区以获取活动系统的信息，并在崩溃系统的核心转储中获取。这些为您提供类似于 dmesg -a 的输出，但 dcons(4) 缓冲区包含更多信息。

#### 10.7.2.1. 使用 KVM 的 Dcons

使用 dcons(4) 与 KVM：

转储活动系统的 dcons(4) 缓冲区：

```
# dconschat -1
```

转储崩溃转储的 dcons(4) 缓冲区：

```
# dconschat -1 -M vmcore.XX
```

可以通过实时核心调试进行调试：

```
# fwcontrol -m target_eui64
# kgdb kernel /dev/fwmem0.2
```

## 10.8. 内核调试选项术语表

本节简要介绍了用于调试的编译时内核选项的术语表

* options KDB ：在内核调试器框架中编译。对 options DDB 和 options GDB 必需。几乎没有性能开销。默认情况下，在发生紧急情况时会进入调试器，而不是自动重新启动。
* options KDB_UNATTENDED ：将 debug.debugger_on_panic sysctl 的默认值更改为 0，该值控制发生紧急情况时是否进入调试器。当 options KDB 未编译到内核中时，行为是在发生紧急情况时自动重新启动；当它编译到内核中时，默认行为是进入调试器，除非 options KDB_UNATTENDED 也编译进去。如果您想将内核调试器编译到内核中，但希望系统在您未使用调试器进行诊断时重新启动，请使用此选项。
* options KDB_TRACE ：将 debug.trace_on_panic sysctl 的默认值更改为 1，该值控制发生紧急情况时调试器是否自动打印堆栈跟踪。特别是在运行时，这对于在串行或 firewire 控制台上收集基本调试信息并仍然重新启动以恢复可能很有帮助。
* options DDB ：在系统的活动低级控制台上运行的交互式调试器，DDB 的支持编译。这包括视频控制台、串行控制台或 FireWire 控制台。它提供基本的集成调试功能，如堆栈跟踪、进程和线程列表、锁状态转储、VM 状态、文件系统状态和内核内存管理。DDB 不需要在第二台机器上运行软件或能够生成核心转储或完整的调试内核符号，并提供运行时内核的详细诊断。许多错误可以仅使用 DDB 输出进行完全诊断。此选项取决于 options KDB 。
* options GDB ：编译支持远程调试器 GDB，它可以通过串行电缆或 FireWire 操作。当进入调试器时，GDB 可以附加以检查结构内容、生成堆栈跟踪等。有些内核状态比在 DDB 中更难访问，DDB 能够自动生成内核状态的有用摘要，如自动遍历锁调试或内核内存管理结构，需要第二台运行调试器的机器。另一方面，GDB 结合了内核源代码和完整的调试符号的信息，并且了解完整的数据结构定义、本地变量，并且可编写脚本。此选项不需要在内核核心转储上运行 GDB。此选项取决于 options KDB 。
* options BREAK_TO_DEBUGGER ， options ALT_BREAK_TO_DEBUGGER ：允许在控制台上发送中断信号或替代信号进入调试器。如果系统在没有恐慌的情况下挂起，这是一个有用的进入调试器的方法。由于当前内核锁定，在串行控制台上生成的中断信号更可靠地进入调试器，并且通常建议使用。此选项几乎不会对性能产生影响。
* options INVARIANTS ：将大量运行时断言检查和测试编译到内核中，不断测试内核数据结构的完整性和内核算法的不变性。这些测试可能很昂贵，因此默认情况下不会编译进去，但有助于提供有用的“故障停止”行为，在内核数据损坏发生之前，某些类别的不良行为会进入调试器，使它们更容易调试。测试包括内存擦除和使用后释放测试，这是开销较大的重要开销来源之一。此选项取决于 options INVARIANT_SUPPORT 。
* options INVARIANT_SUPPORT ： options INVARIANTS 中的许多测试需要修改的数据结构或额外的内核符号来定义。
* options WITNESS ：此选项启用运行时锁定顺序跟踪和验证，是死锁诊断的宝贵工具。WITNESS 通过锁类型维护已获取的锁定顺序图，并在每次获取时检查图表中的循环（隐式或显式）。如果检测到循环，将向控制台生成警告和堆栈跟踪，指示可能发生死锁。为了使用 show locks 、 show witness 和 show alllocks DDB 命令，需要 WITNESS。此调试选项具有显着的性能开销，可以通过使用 options WITNESS_SKIPSPIN 来在一定程度上减轻。详细文档可在 witness(4)中找到。
* options WITNESS_SKIPSPIN ：禁用 WITNESS 的自旋锁顺序的运行时检查。由于调度器中最频繁地获取自旋锁，并且调度器事件经常发生，此选项可以显著加快运行 WITNESS 的系统速度。此选项取决于 options WITNESS 。
* options WITNESS_KDB ：将 debug.witness.kdb sysctl 的默认值更改为 1，这会导致 WITNESS 在检测到锁顺序违规时进入调试器，而不仅仅是打印警告。此选项取决于 options WITNESS 。
* options SOCKBUF_DEBUG ：对套接字缓冲区执行广泛的运行时一致性检查，这对于调试套接字错误和协议以及与套接字交互的设备驱动程序中的竞争条件非常有用。此选项显著影响网络性能，并可能改变设备驱动程序竞争中的时序。
* options DEBUG_VFS_LOCKS ：跟踪锁管理器/ vnode 锁的锁获取点，扩展了 DDB 中显示的信息量。此选项会对性能产生可衡量的影响。
* options DEBUG_MEMGUARD ：malloc(9) 内核内存分配器的替代品，使用 VM 系统来检测释放后分配内存的读取或写入。详细信息可在 memguard(9) 中找到。此选项会产生显著的性能影响，但在调试内核内存损坏错误时非常有帮助。
* options DIAGNOSTIC ：启用额外的、更昂贵的诊断测试，沿着 options INVARIANTS 的方向。
* options KASAN ：启用内核地址消毒剂。这将启用编译器插桩，可用于检测内核中的无效内存访问，如释放后使用和缓冲区溢出。这在很大程度上取代了 options DEBUG_MEMGUARD 。有关详细信息，请参阅 kasan(9)，以及当前支持的平台。
* options KMSAN ：启用内核内存消毒剂。这将启用编译器插桩，可用于检测未初始化内存的使用。有关详细信息，请参阅 kmsan(9)，以及当前支持的平台。
