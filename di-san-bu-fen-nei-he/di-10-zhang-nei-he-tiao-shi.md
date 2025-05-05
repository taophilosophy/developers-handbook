# 第 10 章 内核调试

## 10.1. 获取内核崩溃转储

当运行开发版内核（例如，FreeBSD-CURRENT）时，或者在极端条件下运行内核（例如，极高的负载平均值、数万个连接、异常高的并发用户数、数百个 [jail(8)](https://man.freebsd.org/cgi/man.cgi?query=jail&sektion=8&format=html) 等），或者在使用 FreeBSD-STABLE 上的新特性或设备驱动（例如 PAE）时，内核有时会发生 panic。在发生 panic 时，本章节将展示如何从崩溃中提取有用的信息。

一旦内核发生 panic，系统重启是不可避免的。一旦系统重启，系统物理内存（RAM）的内容将丢失，以及在崩溃前交换设备上的任何数据。为了保存物理内存中的数据，内核使用交换设备作为在崩溃后重启时存储 RAM 中数据的临时位置。这样，在 FreeBSD 崩溃后重新启动时，可以提取内核映像并进行调试。

>**注意**
>
> 已配置为转储设备的交换设备仍然充当交换设备。当前不支持将转储写入非交换设备（例如磁带或 CDRW）。 “交换设备”等同于“交换分区”。

有几种类型的内核崩溃转储可供选择：

* 完整内存转储：包含物理内存的完整内容。
* 小型转储：仅包含内核使用的内存页（FreeBSD 6.2 及更高版本）。
* 文本转储：包含捕获的脚本或交互式调试器输出（FreeBSD 7.1 及更高版本）。

自 FreeBSD 7.0 起，小型转储是默认的转储类型，在大多数情况下，它会捕获完整内存转储中所有必要的信息，因为大多数问题只需要使用内核状态来进行隔离。

### 10.1.1. 配置转储设备

在内核将其物理内存内容转储到转储设备之前，必须先配置转储设备。可以使用 [dumpon(8)](https://man.freebsd.org/cgi/man.cgi?query=dumpon&sektion=8&format=html) 命令告诉内核将内核崩溃转储保存到哪里。必须在使用 [swapon(8)](https://man.freebsd.org/cgi/man.cgi?query=swapon&sektion=8&format=html) 配置交换分区后调用 [dumpon(8)](https://man.freebsd.org/cgi/man.cgi?query=dumpon&sektion=8&format=html) 程序。这通常通过在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中设置 `dumpdev` 变量来处理，`dumpdev` 变量指定交换设备的路径（这是提取内核转储的推荐方式），或者设置为 `AUTO`，以使用第一个配置的交换设备。`dumpdev` 的默认值在 HEAD 中为 `AUTO`，在 RELENG\_\* 分支中更改为 `NO`（除了 RELENG\_7，仍然设置为 `AUTO`）。在 FreeBSD 9.0-RELEASE 及更高版本中，安装过程中，bsdinstall 会询问是否启用目标系统的崩溃转储。

>**技巧**
>
>  检查 **/etc/fstab** 或 [swapinfo(8)](https://man.freebsd.org/cgi/man.cgi?query=swapinfo&sektion=8&format=html)，以查看交换设备的列表。


>**重要**
>
>在内核崩溃之前，确保在 rc.conf(5) 中指定的 dumpdir 已经存在！
>
>```sh
># mkdir /var/crash
># chmod 700 /var/crash
>```
>
>另外，请记住，/var/crash 目录的内容是敏感的，极有可能包含诸如密码等机密信息。


### 10.1.2. 提取内核转储

若内核转储已写入转储设备，必须在挂载交换设备之前提取该转储。要从转储设备提取转储，请使用 [savecore(8)](https://man.freebsd.org/cgi/man.cgi?query=savecore&sektion=8&format=html) 程序。如果在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中设置了 `dumpdev`，则在崩溃后的第一次多用户启动时，[savecore(8)](https://man.freebsd.org/cgi/man.cgi?query=savecore&sektion=8&format=html) 将自动调用，并且在交换设备挂载之前会被调用。提取的核心文件位置由 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中的 `dumpdir` 值指定，默认位置是 **/var/crash**，并且文件名为 **vmcore.0**。

如果 **/var/crash** 中已经存在一个名为 **vmcore.0** 的文件（或 `dumpdir` 设置的任何其他目录中），内核将在每次崩溃后递增尾部的数字，以避免覆盖现有的 **vmcore** 文件（例如：**vmcore.1**）。在转储保存之后，[savecore(8)](https://man.freebsd.org/cgi/man.cgi?query=savecore&sektion=8&format=html) 将始终在 **/var/crash** 中创建一个名为 **vmcore.last** 的符号链接。此符号链接可用于查找最新转储的文件名。

[crashinfo(8)](https://man.freebsd.org/cgi/man.cgi?query=crashinfo&sektion=8&format=html) 工具生成一个包含完整内存转储或小型转储摘要的文本文件。如果在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中设置了 `dumpdev`，则在 [savecore(8)](https://man.freebsd.org/cgi/man.cgi?query=savecore&sektion=8&format=html) 执行后，[crashinfo(8)](https://man.freebsd.org/cgi/man.cgi?query=crashinfo&sektion=8&format=html) 将自动调用。输出将保存到 `dumpdir` 中，文件名为 **core.txt.N**。


>**技巧**
>
>如果你正在测试一个新的内核，但需要启动不同的内核才能使系统恢复正常，使用引导提示符中的 -s 标志将其仅引导到单用户模式，然后执行以下步骤：
>
>```
># fsck -p
># mount -a -t ufs       # 确保 /var/crash 可写
># savecore /var/crash /dev/ad0s1b
># exit                  # 退出到多用户模式
>```

### 10.1.3. 测试内核转储配置

内核包含一个 [sysctl(8)](https://man.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&format=html) 节点，用于请求内核 panic。此功能可用于验证系统是否已正确配置以保存内核崩溃转储。你可能希望在触发崩溃之前，在单用户模式下将现有文件系统重新挂载为只读，以避免数据丢失。

```
# shutdown now
...
Enter full pathname of shell or RETURN for /bin/sh:
# mount -a -u -r
# sysctl debug.kdb.panic=1
debug.kdb.panic: panic: kdb_sysctl_panic
...
```

重新启动后，系统应将转储保存在 **/var/crash** 目录，并附带一个来自 [crashinfo(8)](https://man.freebsd.org/cgi/man.cgi?query=crashinfo&sektion=8&format=html) 的匹配摘要。

## 10.2. 使用 `kgdb` 调试内核崩溃转储

|   | 本节介绍 [kgdb(1)](https://man.freebsd.org/cgi/man.cgi?query=kgdb&sektion=1&format=html)。最新版本包含在 [devel/gdb](https://cgit.freebsd.org/ports/tree/devel/gdb/) 中。FreeBSD 11 及更早版本中也包含一个旧版本。 |
| - | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

要进入调试器并开始从转储中获取信息，启动 kgdb：

```
# kgdb -n N
```

其中 *N* 是要检查的 **vmcore.N** 后缀。要打开最近的转储，可以使用：

```
# kgdb -n last
```

通常，[kgdb(1)](https://man.freebsd.org/cgi/man.cgi?query=kgdb&sektion=1&format=html) 应该能够找到生成转储时运行的内核。如果无法找到正确的内核，可以将内核和转储的路径作为两个参数传递给 kgdb：

```
# kgdb /boot/kernel/kernel /var/crash/vmcore.0
```

你可以像调试其他程序一样，使用内核源代码调试崩溃转储。

此转储来自 5.2-BETA 内核，崩溃发生在内核的深处。下面的输出已修改，左侧包含行号。此第一个跟踪检查指令指针并获得回溯。第 41 行用于 `list` 命令的地址是指令指针，可以在第 17 行找到。如果你无法自己调试问题，大多数开发人员会要求至少将这些信息发送给他们。如果你能够解决问题，确保通过问题报告、邮件列表或提交代码的方式将你的补丁合并到源代码树中！

```sh
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

>**技巧**
>
>如果你的系统经常崩溃并且磁盘空间不足，删除 **/var/crash** 中的旧 **vmcore** 文件可以节省相当多的磁盘空间！


## 10.3. 使用 DDB 进行在线内核调试

虽然 `kgdb` 作为离线调试器提供了非常高级的用户界面，但它有一些无法完成的任务。最重要的两个是设置断点和单步执行内核代码。

如果你需要对内核进行低级调试，可以使用一个在线调试器 DDB。它允许设置断点、单步执行内核函数、检查和修改内核变量等。然而，它无法访问内核源文件，且只能访问全局和静态符号，而不像 `kgdb` 那样拥有完整的调试信息。

要配置内核以包含 DDB，请在配置文件中添加以下选项：

```
options KDB
```

```
options DDB
```

然后重新构建内核。（有关如何配置 FreeBSD 内核的详细信息，请参阅 [FreeBSD 手册](https://docs.freebsd.org/en/books/handbook/)）。

一旦 DDB 内核启动运行，你可以通过多种方式进入 DDB。首先，最早的方式是使用启动标志 `-d`。这样，内核将在调试模式下启动，并在任何设备探测之前进入 DDB。因此，你甚至可以调试设备探测/附加函数。要使用此方法，请退出加载器的启动菜单并在加载器提示符下输入 `boot -d`。

第二种方式是在系统启动后进入调试器。有两种简单的方法可以实现这一点。如果你希望从命令提示符进入调试器，只需键入以下命令：

```
# sysctl debug.kdb.enter=1
```

或者，如果你在系统控制台上，可以使用键盘上的热键。默认的断点调试器快捷键是 **Ctrl**+**Alt**+**ESC**。对于 syscons，可以重新映射此快捷键，且一些分发的映射已经这样做，所以确保你知道正确的快捷键序列。如果你使用串行控制台，则可以通过串行线路上的 BREAK 信号进入 DDB（在内核配置文件中使用 `options BREAK_TO_DEBUGGER`）。这不是默认设置，因为很多串行适配器会无故生成 BREAK 信号，例如拔掉电缆时。

第三种方式是任何 panic 条件都会跳转到 DDB，如果内核已配置为使用它。因此，为无人值守的机器配置带有 DDB 的内核并不明智。

为了获得无人值守功能，可以在内核配置文件中添加：

```
options KDB_UNATTENDED
```

然后重新构建/重新安装内核。

DDB 的命令大致类似于一些 `gdb` 命令。你可能首先需要做的是设置断点：

```
break function-name address
```

数字默认为十六进制，但为了与符号名称区分开来，十六进制数字中以字母 `a-f` 开头的需要前面加 `0x`（其他数字则不需要）。简单的表达式也可以，例如：`function-name + 0x103`。

要退出调试器并继续执行，输入：

```
continue
```

要获取当前线程的堆栈跟踪，使用：

```
trace
```

要获取任意线程的堆栈跟踪，可以将进程 ID 或线程 ID 作为第二个参数传递给 `trace`。

如果要删除一个断点，使用：

```
del
 del address-expression
```

第一个形式会在断点命中后立即接受并删除当前断点。第二个形式可以删除任何断点，但需要指定确切的地址；可以通过以下命令获取该地址：

```
show b
```

或：

```
show break
```

要单步执行内核，可以尝试：

```
s
```

这会进入函数，但你可以让 DDB 跟踪这些函数，直到匹配的返回语句被达到，使用：

```
n
```

>**注意**
>
>这与 `gdb` 的 `next` 语句不同；它类似于 `gdb` 的 `finish`。多次按下 n 将导致继续执行。

要检查内存中的数据，可以使用（例如）：

```
x/wx 0xf0133fe0,40
 x/hd db_symtab_space
 x/bc termbuf,10
 x/s stringbuf
```

用于字/半字/字节访问，以及十六进制/十进制/字符/字符串显示。逗号后的数字是对象的数量。要显示接下来的 0x10 项，只需使用：

```
x ,10
```

类似地，使用

```
x/ia foofunc,10
```

来反汇编 `foofunc` 的前 0x10 条指令，并显示它们及其相对于 `foofunc` 开始位置的偏移。

要修改内存，使用写入命令：

```
w/b termbuf 0xa 0xb 0
 w/w 0xf0010030 0 0
```

命令修饰符（`b`/`h`/`w`）指定要写入的数据大小，第一个跟随的表达式是写入地址，剩下的被解释为写入到连续内存位置的数据。

如果需要查看当前的寄存器，可以使用：

```
show reg
```

或者，你可以通过例如以下命令显示单个寄存器的值：

```
p $eax
```

并通过以下命令修改它：

```
set $eax new-value
```

如果需要从 DDB 调用一些内核函数，只需输入：

```
call func(arg1, arg2, ...)
```

返回值将被打印出来。

要查看所有正在运行的进程的 [ps(1)](https://man.freebsd.org/cgi/man.cgi?query=ps&sektion=1&format=html) 风格总结，使用：

```
ps
```

现在，你已经检查了内核崩溃的原因，想要重启系统。记住，根据先前故障的严重程度，并非所有内核部分仍然按预期工作。执行以下操作之一来关闭并重启系统：

```
panic
```

这将导致内核转储并重启，因此你可以稍后使用 [kgdb(1)](https://man.freebsd.org/cgi/man.cgi?query=kgdb&sektion=1&format=html) 在更高层次分析转储。

```
call boot(0)
```

可能是一个干净地关闭系统、`sync()` 所有磁盘并在某些情况下重启的好方法。只要内核的磁盘和文件系统接口没有损坏，这可能是一个几乎干净的关闭方式。

```
reset
```

这是灾难的最后逃生方式，几乎等同于按下大红按钮。

如果需要简短的命令总结，只需输入：

```
help
```

强烈建议在调试会话中准备一份 [ddb(4)](https://man.freebsd.org/cgi/man.cgi?query=ddb&sektion=4&format=html) 手册的打印版。记住，在单步调试内核时，很难阅读在线手册。



这将指示 [savecore(8)](https://man.freebsd.org/cgi/man.cgi?query=savecore&sektion=8&format=html) 从 /dev/ad0s1b 提取内核转储并将内容放入 /var/crash。不要忘记确保目标目录 /var/crash 有足够的空间来存储转储。同时，也不要忘记指定正确的交换设备路径，因为它可能不同于 /dev/ad0s1b！


## 10.4. 在线内核调试使用远程 GDB

FreeBSD 内核提供了第二种 KDB 后端用于在线调试： [gdb(4)](https://man.freebsd.org/cgi/man.cgi?query=gdb&sektion=4&format=html)。这一功能自 FreeBSD 2.2 以来一直被支持，并且实际上是一个非常棒的功能。

GDB 长期以来一直支持 *远程调试*。这通过一个非常简单的协议沿着串行线进行。与上述其他调试方法不同，使用远程 GDB 需要两台机器。一台是提供调试环境的主机，包括所有源代码和带有所有符号的内核二进制文件。另一台是运行相同内核副本的目标机器（可以选择剥离调试信息）。

为了使用远程 GDB，请确保在内核配置中包含以下选项：

```
makeoptions     DEBUG=-g
options         KDB
options         GDB
```

请注意，`GDB` 选项在 -STABLE 和 -RELEASE 分支的 `GENERIC` 内核中默认关闭，但在 -CURRENT 中已启用。

构建完成后，将内核复制到目标机器并启动它。将目标机器的串行线（其 uart 设备上设置了 "flags 080"）连接到调试主机的任何串行端口。有关如何在 uart 设备上设置标志的信息，请参见 [uart(4)](https://man.freebsd.org/cgi/man.cgi?query=uart&sektion=4&format=html)。

目标机器必须进入 GDB 后端，可以是由于 panic 或者通过故意触发进入调试器。在执行此操作之前，选择 GDB 调试后端：

```
# sysctl debug.kdb.current=gdb
debug.kdb.current: ddb -> gdb
```

>**注意**
>
> 可以通过 `debug.kdb.available` sysctl 列出支持的后端。如果内核配置中包含 `options DDB`，则默认选择 [ddb(4)](https://man.freebsd.org/cgi/man.cgi?query=ddb&sektion=4&format=html)。如果 `gdb` 没有出现在可用后端列表中，则可能未正确配置调试串口。

然后，强制进入调试器：

```
# sysctl debug.kdb.enter=1
debug.kdb.enter: 0KDB: enter: sysctl debug.kdb.enter
```

目标机器现在等待来自远程 GDB 客户端的连接。在调试机器上，进入目标内核的编译目录，并启动 `gdb`：

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

通过以下命令初始化远程调试会话（假设使用的是第一个串口）：

```
(kgdb) target remote /dev/cuau0
```

现在，主机 GDB 将控制目标内核：

```
Remote debugging using /dev/cuau0
kdb_enter (why=<optimized out>, msg=<optimized out>) at /usr/src/sys/kern/subr_kdb.c:506
506                     kdb_why = KDB_WHY_UNSET;
(kgdb)
```

>**技巧**
>
>根据所使用的编译器，一些局部变量可能会显示为空，从而无法被 `gdb` 直接检查。如果在调试过程中出现此问题，可以通过以较低的优化级别构建内核来改善某些变量的可见性。这可以通过将 `COPTFLAGS=-O1` 传递给 [make(1)](https://man.freebsd.org/cgi/man.cgi?query=make&sektion=1&format=html) 来完成。然而，某些类型的内核错误在更改优化级别时可能会表现不同（或根本不再出现）。 

您可以像使用任何其他 GDB 会话一样使用此会话，包括完全访问源代码，在 Emacs 窗口中以 gud-mode 运行它（这会在另一个 Emacs 窗口中自动显示源代码）等。

## 10.5. 调试控制台驱动程序

由于需要一个控制台驱动程序才能运行 DDB，如果控制台驱动程序本身出现故障，事情会变得更加复杂。您可能还记得使用串行控制台（通过修改启动块，或在 `Boot:` 提示符下指定 `-h`），并将标准终端连接到您的第一个串行端口。DDB 在任何配置了的控制台驱动程序上都能工作，包括串行控制台。

## 10.6. 调试死锁

您可能会遇到所谓的死锁，即系统停止执行有用的工作。在这种情况下，为了提供有帮助的错误报告，请按照前一节中所述使用 [ddb(4)](https://man.freebsd.org/cgi/man.cgi?query=ddb&sektion=4&format=html)。在报告中包括 `ps` 和 `trace` 的输出，针对怀疑的进程。

如果可能，考虑进行进一步调查。如果您怀疑死锁发生在 VFS 层，下面的步骤尤其有用。请将这些选项添加到内核配置文件中。

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

当死锁发生时，除了 `ps` 命令的输出外，还应提供来自 `show pcpu`、`show allpcpu`、`show locks`、`show alllocks`、`show lockedvnods` 和 `alltrace` 的信息。

为了获得线程进程的有意义的回溯，可以使用 `thread thread-id` 切换到线程栈，然后使用 `where` 进行回溯。

## 10.7. 使用 Dcons 进行内核调试

[dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 是一个非常简单的控制台驱动程序，未直接与任何物理设备连接。它仅从内核或加载程序的缓冲区读取和写入字符。由于其简单性，它在内核调试中非常有用，特别是在使用 FireWire® 设备时。目前，FreeBSD 提供了两种方式与缓冲区进行交互，使用 [dconschat(8)](https://man.freebsd.org/cgi/man.cgi?query=dconschat&sektion=8&format=html)。

### 10.7.1. 通过 FireWire® 使用 Dcons

大多数 FireWire®（IEEE1394）主机控制器基于 OHCI 规范，支持物理访问主机内存。这意味着一旦主机控制器初始化完成，我们可以在不依赖软件（内核）的情况下访问主机内存。我们可以利用这一功能与 [dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 进行交互。[dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 提供类似于串行控制台的功能。它模拟两个串行端口，一个用于控制台和 DDB，另一个用于 GDB。由于远程内存访问完全由硬件处理，即使系统崩溃，[dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 缓冲区仍然可以访问。

FireWire® 设备不限于集成在主板中的设备。桌面电脑可以使用 PCI 卡，笔记本电脑可以购买卡总线接口。

#### 10.7.1.1. 在目标机器上启用 FireWire® 和 Dcons 支持

要在目标机器的内核中启用 FireWire® 和 Dcons 支持：

* 确保您的内核支持 `dcons`、`dcons_crom` 和 `firewire`。`Dcons` 应该与内核静态链接。对于 `dcons_crom` 和 `firewire`，模块应该是可以的。
* 确保启用了物理 DMA。您可能需要在 **/boot/loader.conf** 中添加 `hw.firewire.phydma_enable=1`。
* 添加调试选项。
* 如果使用 GDB 通过 FireWire® 调试，请在 **/boot/loader.conf** 中添加 `dcons_gdb=1`。
* 在 **/etc/ttys** 中启用 `dcons`。
* 可选地，要强制将 `dcons` 设置为高级控制台，请在 **loader.conf** 中添加 `hw.firewire.dcons_crom.force_console=1`。

要在 i386 或 amd64 上启用 FireWire® 和 Dcons 支持，首先在 [loader(8)](https://man.freebsd.org/cgi/man.cgi?query=loader&sektion=8&format=html) 中启用：

在 **/etc/make.conf** 中添加 `LOADER_FIREWIRE_SUPPORT=YES`，然后重新构建 [loader(8)](https://man.freebsd.org/cgi/man.cgi?query=loader&sektion=8&format=html)：

```
# cd /sys/boot/i386 && make clean && make && make install
```

要启用 [dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 作为活动的低级控制台，请在 **/boot/loader.conf** 中添加 `boot_multicons="YES"`。

以下是一些配置示例。一个示例内核配置文件应该包含：

```
device dcons
device dcons_crom
options KDB
options DDB
options GDB
options ALT_BREAK_TO_DEBUGGER
```

一个示例 **/boot/loader.conf** 文件应包含：

```
dcons_crom_load="YES"
dcons_gdb=1
boot_multicons="YES"
hw.firewire.phydma_enable=1
hw.firewire.dcons_crom.force_console=1
```

#### 10.7.1.2. 在主机机器上启用 FireWire® 和 Dcons 支持

要在主机机器的内核中启用 FireWire® 支持：

```
# kldload firewire
```

查找 FireWire® 主机控制器的 EUI64（唯一的 64 位标识符），并使用 [fwcontrol(8)](https://man.freebsd.org/cgi/man.cgi?query=fwcontrol&sektion=8&format=html) 或 `dmesg` 查找目标机器的 EUI64。

运行 [dconschat(8)](https://man.freebsd.org/cgi/man.cgi?query=dconschat&sektion=8&format=html)，使用以下命令：

```
# dconschat -e \# -br -G 12345 -t 00-11-22-33-44-55-66-77
```

以下是运行 [dconschat(8)](https://man.freebsd.org/cgi/man.cgi?query=dconschat&sektion=8&format=html) 后可以使用的键盘组合：

| \~**+**. | 断开连接         |
| -------- | ------------ |
| \~       | ALT BREAK    |
| \~       | 重置目标         |
| \~       | 暂停 dconschat |

通过启动 [kgdb(1)](https://man.freebsd.org/cgi/man.cgi?query=kgdb&sektion=1&format=html) 并进行远程调试会话来附加远程 GDB：

```
kgdb -r :12345 kernel
```

#### 10.7.1.3. 一些常规提示

以下是一些常规提示：

为了充分利用 FireWire® 的速度，禁用其他较慢的控制台驱动程序：

```
# conscontrol delete ttyd0	     # 串口控制台
# conscontrol delete consolectl	# 视频/键盘
```

存在一个适用于 [emacs(1)](https://man.freebsd.org/cgi/man.cgi?query=emacs&sektion=1&format=html) 的 GDB 模式；您需要在 **.emacs** 中添加以下内容：

```
(setq gud-gdba-command-name "kgdb -a -a -a -r :12345")
(setq gdb-many-windows t)
(xterm-mouse-mode 1)
M-x gdba
```

对于 DDD (**devel/ddd**)，您可以使用以下命令：

```
# 远程串行协议
LANG=C ddd --debugger kgdb -r :12345 kernel
# 实时核心调试
LANG=C ddd --debugger kgdb kernel /dev/fwmem0.2
```

### 10.7.2. 与 KVM 一起使用 Dcons

我们可以通过 **/dev/mem** 直接读取 [dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 缓冲区，适用于运行中的系统，也适用于崩溃后的核心转储。这些输出类似于 `dmesg -a`，但 [dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 缓冲区包含更多信息。

#### 10.7.2.1. 使用 Dcons 与 KVM

要与 KVM 一起使用 [dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html)：

转储运行中系统的 [dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 缓冲区：

```
# dconschat -1
```

转储崩溃转储的 [dcons(4)](https://man.freebsd.org/cgi/man.cgi?query=dcons&sektion=4&format=html) 缓冲区：

```
# dconschat -1 -M vmcore.XX
```

可以通过以下方式进行实时核心调试：

```
# fwcontrol -m target_eui64
# kgdb kernel /dev/fwmem0.2
```

## 10.8. 内核调试选项词汇表

本节提供了用于调试的编译时内核选项的简要词汇表：

* `options KDB`：编译内核调试器框架。`options DDB` 和 `options GDB` 需要此选项。几乎没有性能开销。默认情况下，当系统发生 panic 时，调试器会被触发，而不是自动重启。
* `options KDB_UNATTENDED`：将 `debug.debugger_on_panic` sysctl 的默认值更改为 0，该 sysctl 控制系统在 panic 时是否进入调试器。如果内核中未编译 `options KDB`，则默认行为是在 panic 时自动重启；如果编译了 `options KDB`，默认行为是在没有编译 `options KDB_UNATTENDED` 的情况下进入调试器。如果希望将内核调试器保留在内核中，但希望系统在不进行诊断时能够恢复，除非您能使用调试器，使用此选项。
* `options KDB_TRACE`：将 `debug.trace_on_panic` sysctl 的默认值更改为 1，该 sysctl 控制是否在 panic 时自动打印堆栈跟踪。尤其是在运行 `options KDB_UNATTENDED` 时，这对于在串行或 FireWire 控制台上收集基本调试信息非常有帮助，同时仍然能进行重启恢复。
* `options DDB`：编译支持控制台调试器 DDB。此交互式调试器可以在系统的任何活动低级控制台上运行，包括视频控制台、串行控制台或 FireWire 控制台。它提供基本的集成调试功能，如堆栈跟踪、进程和线程列表、锁状态转储、虚拟内存状态、文件系统状态和内核内存管理。DDB 不需要在第二台机器上运行软件，也不需要生成核心转储或完整的调试内核符号，提供实时的内核诊断。许多错误可以仅通过 DDB 输出完全诊断。此选项依赖于 `options KDB`。
* `options GDB`：编译支持远程调试器 GDB，可以通过串行电缆或 FireWire 进行操作。当调试器被触发时，可以附加 GDB 来检查结构内容、生成堆栈跟踪等。某些内核状态比在 DDB 中更难访问，因为 DDB 可以自动生成有用的内核状态摘要，如自动遍历锁调试或内核内存管理结构，而 GDB 需要在第二台机器上运行。另一方面，GDB 结合了内核源代码和完整的调试符号，并且能够了解完整的数据结构定义、局部变量，且可以编写脚本。此选项不要求在内核核心转储上运行 GDB。此选项依赖于 `options KDB`。
* `options BREAK_TO_DEBUGGER`，`options ALT_BREAK_TO_DEBUGGER`：允许在控制台上使用中断信号或替代信号进入调试器。如果系统在没有 panic 的情况下挂起，这是进入调试器的一种有用方法。由于当前内核锁定的原因，通过串行控制台生成的中断信号在进入调试器时更为可靠，因此通常推荐使用这种方式。此选项对性能的影响很小或没有影响。
* `options INVARIANTS`：将大量运行时断言检查和测试编译到内核中，这些检查和测试不断验证内核数据结构的完整性和内核算法的不变性。由于这些测试可能会比较耗费资源，因此默认情况下不编译，但它们有助于提供有用的“故障停止”行为，在内核数据损坏发生之前，某些类别的非预期行为会先进入调试器，使其更容易调试。这些测试包括内存擦洗和使用后释放的测试，这是影响性能的一个重要因素。此选项依赖于 `options INVARIANT_SUPPORT`。
* `options INVARIANT_SUPPORT`：`options INVARIANTS` 中的许多测试需要修改的数据结构或需要定义额外的内核符号。
* `options WITNESS`：此选项启用运行时锁定顺序跟踪和验证，是诊断死锁的宝贵工具。WITNESS 维护一个按锁类型获取的锁定顺序图，并在每次获取时检查图中是否存在循环（隐式或显式）。如果检测到循环，会生成警告并打印堆栈跟踪到控制台，表明可能发生了死锁。使用 `show locks`、`show witness` 和 `show alllocks` DDB 命令需要启用 WITNESS。此调试选项具有显著的性能开销，但可以通过使用 `options WITNESS_SKIPSPIN` 来缓解。详细文档请参见 [witness(4)](https://man.freebsd.org/cgi/man.cgi?query=witness&sektion=4&format=html)。
* `options WITNESS_SKIPSPIN`：禁用 WITNESS 的自旋锁顺序检查。由于调度器中最常频繁地获取自旋锁，并且调度事件频繁发生，此选项可以显著加快运行 WITNESS 的系统。此选项依赖于 `options WITNESS`。
* `options WITNESS_KDB`：将 `debug.witness.kdb` sysctl 的默认值更改为 1，这会使 WITNESS 在检测到锁定顺序违规时进入调试器，而不仅仅是打印警告。此选项依赖于 `options WITNESS`。
* `options SOCKBUF_DEBUG`：对套接字缓冲区执行广泛的运行时一致性检查，这对于调试套接字错误和协议以及与套接字交互的设备驱动中的竞争条件非常有用。此选项对网络性能有显著影响，可能会改变设备驱动中的时序。
* `options DEBUG_VFS_LOCKS`：跟踪 lockmgr/vnode 锁的锁定获取点，扩展 DDB 中 `show lockedvnods` 显示的内容。此选项对性能有可测量的影响。
* `options DEBUG_MEMGUARD`：这是一个替代 [malloc(9)](https://man.freebsd.org/cgi/man.cgi?query=malloc&sektion=9&format=html) 的内核内存分配器，使用 VM 系统检测从分配的内存中读取或写入已释放内存。详细信息请参见 [memguard(9)](https://man.freebsd.org/cgi/man.cgi?query=memguard&sektion=9&format=html)。此选项对性能有显著影响，但在调试内核内存损坏错误时非常有用。
* `options DIAGNOSTIC`：启用附加的、较为昂贵的诊断测试，类似于 `options INVARIANTS`。
* `options KASAN`：启用内核地址消毒器（Kernel Address Sanitizer）。这启用编译器插桩，用于检测内核中的无效内存访问，如使用后释放和缓冲区溢出。此选项在很大程度上取代了 `options DEBUG_MEMGUARD`。有关详细信息，请参见 [kasan(9)](https://man.freebsd.org/cgi/man.cgi?query=kasan&sektion=9&format=html)，以及当前支持的平台。
* `options KMSAN`：启用内核内存消毒器（Kernel Memory Sanitizer）。这启用编译器插桩，用于检测未初始化内存的使用。有关详细信息，请参见 [kmsan(9)](https://man.freebsd.org/cgi/man.cgi?query=kmsan&sektion=9&format=html)，以及当前支持的平台。

