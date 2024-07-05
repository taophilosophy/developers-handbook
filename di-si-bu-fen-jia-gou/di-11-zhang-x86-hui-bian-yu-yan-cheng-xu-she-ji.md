# 第 11 章 x86 汇编语言程序设计

这一章由 G. Adam Stanislav <adam@redprince.net> 撰写。

## A.1. 概要

UNIX® 下的汇编语言编程是高度未记录的。一般认为没有人会想要使用它，因为各种 UNIX® 系统运行在不同的微处理器上，所以一切都应该用 C 语言编写以实现可移植性。

实际上，C 的可移植性相当于神话。即使是 C 程序在从一个 UNIX® 移植到另一个 UNIX® 时也需要进行修改，而不管每个系统运行在哪个处理器上。通常，这样的程序充满了依赖于其编译系统的条件语句。

即使我们相信所有的 UNIX®软件都应该用 C 语言或其他高级语言编写，我们仍然需要汇编语言程序员：谁会写访问内核的 C 库部分呢？

在本章中，我将尝试向您展示如何在 FreeBSD 下使用汇编语言编写 UNIX®程序。

本章不解释汇编语言的基础知识。关于这方面有足够的资源（要了解汇编语言的完整在线课程，请参阅 Randall Hyde 的《汇编语言艺术》；或者如果您更喜欢纸质书籍，请查看 Jeff Duntemann 的《逐步学习汇编语言》（ISBN：0471375233）。然而，一旦本章完成，任何汇编语言程序员都将能够快速高效地为 FreeBSD 编写程序。

版权 © 2000-2001 G. Adam Stanislav。保留所有权利。

## A.2. 工具

### A.2.1. 汇编器

汇编语言编程最重要的工具是汇编器，这种软件将汇编语言代码转换为机器语言。

FreeBSD 提供了三种非常不同的汇编器。llvm-as(1)（包含在 devel/llvm 中）和 as(1)（包含在 devel/binutils 中）都使用传统的 UNIX®汇编语言语法。

另一方面，nasm(1)（通过 devel/nasm 安装）使用 Intel 语法。它的主要优势是可以为许多操作系统汇编代码。

本章使用 nasm 语法，因为大多数来自其他操作系统的汇编语言程序员会发现这样更容易理解。而且，坦率地说，这是我习惯于的。

### 链接器

汇编器的输出，就像任何编译器的输出一样，需要链接以形成可执行文件。

标准的 ld(1)链接器随 FreeBSD 提供。它可以与用汇编器组装的代码一起工作。

## A.3. 系统调用

### A.3.1. 默认调用约定

默认情况下，FreeBSD 内核使用 C 调用约定。此外，尽管内核是使用 int 80h 访问的，但假定程序将调用发出 int 80h 的函数，而不是直接发出 int 80h 。

这种约定非常方便，比 MS-DOS®使用的 Microsoft®约定要好得多。为什么？因为 UNIX®约定允许任何用任何语言编写的程序访问内核。

汇编语言程序也可以做到。例如，我们可以打开一个文件：

```
kernel:
	int	80h	; Call kernel
	ret

open:
	push	dword mode
	push	dword flags
	push	dword path
	mov	eax, 5
	call	kernel
	add	esp, byte 12
	ret
```

这是一种非常干净和便携的编码方式。如果您需要将代码转移到一个使用不同中断或不同参数传递方式的 UNIX®系统，您需要更改的只是内核过程。

但是汇编语言程序员喜欢节省周期。上面的示例需要一个 call/ret 组合。我们可以通过 push 一个额外的双字来消除它：

```
open:
	push	dword mode
	push	dword flags
	push	dword path
	mov	eax, 5
	push	eax		; Or any other dword
	int	80h
	add	esp, byte 16
```

我们放置在 EAX 中的 5 标识内核函数，在本例中为 open 。

### 3.2. 替代调用约定

FreeBSD 是一个非常灵活的系统。 它提供了调用内核的其他方式。 但是，为了使其工作，系统必须安装 Linux 模拟。

Linux 是一个类似 UNIX® 的系统。 但是，其内核使用与 MS-DOS® 相同的通过寄存器传递参数的系统调用约定。 与 UNIX® 约定一样，函数号放置在 EAX 中。 然而，参数不是通过堆栈传递而是通过 EBX, ECX, EDX, ESI, EDI, EBP 传递。

```
open:
	mov	eax, 5
	mov	ebx, path
	mov	ecx, flags
	mov	edx, mode
	int	80h
```

这种约定在汇编语言编程方面与 UNIX® 方式相比有一个很大的缺点：每次进行内核调用时，您必须 push 寄存器，然后稍后 pop 它们。这使得您的代码更庞大，更慢。尽管如此，FreeBSD 为您提供了选择。

如果您选择 Linux 约定，您必须让系统知道这一点。在程序汇编和链接完成后，您需要给可执行文件打上标记：

```
% brandelf -t Linux filename
```

### A.3.3. 您应该使用哪种约定？

如果你专门为 FreeBSD 编码，你应该始终使用 UNIX® 约定：它更快，你可以在寄存器中存储全局变量，你不必标记可执行文件，并且你不会强加在目标系统上安装 Linux 模拟软件包。

如果你想创建可以在 Linux 上运行的可移植代码，你可能仍然希望为 FreeBSD 用户提供尽可能高效的代码。在我解释基础知识之后，我会告诉你如何做到这一点。

### A.3.4. 调用号码

要告诉内核您正在调用哪个系统服务，请将其编号放在 EAX 中。 当然，您需要知道编号是多少。

#### A.3.4.1. 系统调用文件

编号列在系统调用中。 locate syscalls 可以找到此文件的几种不同格式，这些格式都是从 syscalls.master 自动生成的。

您可以在 /usr/src/sys/kern/syscalls.master 找到默认 UNIX®调用约定的主文件。如果您需要使用在 Linux 仿真模式中实现的其他约定，请阅读 /usr/src/sys/i386/linux/syscalls.master。

|  | FreeBSD 和 Linux 不仅使用不同的调用约定，有时还对相同功能使用不同的编号。 |
| -- | --------------------------------------------------------------------------- |

syscalls.master 描述了调用的执行方式：

```
0	STD	NOHIDE	{ int nosys(void); } syscall nosys_args int
1	STD	NOHIDE	{ void exit(int rval); } exit rexit_args void
2	STD	POSIX	{ int fork(void); }
3	STD	POSIX	{ ssize_t read(int fd, void *buf, size_t nbyte); }
4	STD	POSIX	{ ssize_t write(int fd, const void *buf, size_t nbyte); }
5	STD	POSIX	{ int open(char *path, int flags, int mode); }
6	STD	POSIX	{ int close(int fd); }
etc...
```

它是最左边的列，告诉我们要放在 EAX 中的数字。

最右边的列告诉我们要 push 什么参数。它们从右到左 push 。

例如，要 open 文件，我们需要先 push {{$2}}，然后 flags ，然后存储 path 的地址。

## 返回值

大多数情况下，如果系统调用不返回某种值，它将毫无用处：打开文件的文件描述符、读取到缓冲区的字节数、系统时间等。

另外，系统需要告知我们是否发生错误：文件不存在、系统资源耗尽、我们传递了无效参数等。

### A.4.1. Man Pages

传统查找 UNIX® 系统下各种系统调用信息的地方是手册页。FreeBSD 在第 2 节中描述其系统调用，有时在第 3 节中。

例如，open(2) 说：

如果成功， open() 返回一个非负整数，称为文件描述符。如果失败，它返回 -1 并设置 errno 以指示错误。

初次接触 UNIX® 和 FreeBSD 的汇编语言程序员会立即问出令人困惑的问题： errno 在哪里，我该如何到达那里呢？

|  | 手册页中呈现的信息适用于 C 程序。汇编语言程序员需要额外的信息。 |
| -- | ----------------------------------------------------------------- |

### A.4.2. 返回值在哪里？

不幸的是，这取决于...对于大多数系统调用，它在 EAX 中，但不是全部。一个好的经验法则是，在第一次使用系统调用时，要在 EAX 中查找返回值。如果那里没有，你需要进一步研究。

|  | 我知道有一个系统调用会在 EDX 中返回值: SYS_fork 。我使用过的所有其他系统调用都使用 EAX 。但我还没有使用过它们全部。 |
| -- | --------------------------------------------------------------------------------------------------------------------- |

|  | 如果您在这里或其他任何地方找不到答案，请阅读 libc 源代码，看看它如何与内核进行交互。 |
| -- | -------------------------------------------------------------------------------------- |

### A.4.3. errno 在哪？

 实际上，不存在…

errno 是 C 语言的一部分，而不是 UNIX®内核。当直接访问内核服务时，错误代码以 EAX 返回，通常正确返回值也会出现在同一个寄存器中。

这是完全合理的。如果没有错误，就没有错误代码。如果有错误，就没有返回值。一个寄存器可以包含其中一个。

### A.4.4. 确定发生了错误

当使用标准 FreeBSD 调用约定时， carry flag 在成功时被清除，在失败时被设置。

当使用 Linux 仿真模式时， EAX 中的有符号值在成功时为非负，并包含返回值。在发生错误时，该值为负，即 -errno 。

## A.5. 创建可移植代码

可移植性通常不是汇编语言的优势之一。然而，使用 nasm 编写可以在不同平台上汇编的汇编语言程序是可能的，特别是在 Windows®和 FreeBSD 等不同操作系统上。

当您希望您的代码在两个基于类似架构但不同的平台上运行时，这种可能性就更大了。

例如，FreeBSD 是 UNIX®，Linux 是类 UNIX®。我只提到了它们之间的三个差异（从汇编语言程序员的角度看）：调用约定、函数编号和返回值的方式。

### 处理函数编号

在许多情况下，功能编号是相同的。然而，即使它们不同，问题也很容易处理：在代码中不要使用数字，而是使用根据目标架构不同声明的常量：

```
%ifdef	LINUX
%define	SYS_execve	11
%else
%define	SYS_execve	59
%endif
```

### 处理约定

通过宏可以解决调用约定和返回值（ errno 问题）：

```
%ifdef	LINUX

%macro	system	0
	call	kernel
%endmacro

align 4
kernel:
	push	ebx
	push	ecx
	push	edx
	push	esi
	push	edi
	push	ebp

	mov	ebx, [esp+32]
	mov	ecx, [esp+36]
	mov	edx, [esp+40]
	mov	esi, [esp+44]
	mov	ebp, [esp+48]
	int	80h

	pop	ebp
	pop	edi
	pop	esi
	pop	edx
	pop	ecx
	pop	ebx

	or	eax, eax
	js	.errno
	clc
	ret

.errno:
	neg	eax
	stc
	ret

%else

%macro	system	0
	int	80h
%endmacro

%endif
```

### A.5.3. 处理其他可移植性问题

上述解决方案可以处理大多数在 FreeBSD 和 Linux 之间编写可移植代码的情况。然而，对于一些内核服务，差异更深。

在这种情况下，您需要为这些特定的系统调用编写两个不同的处理程序，并使用条件汇编。幸运的是，您的大部分代码除了调用内核之外，通常只需要在代码中使用几个这样的条件部分。

### 使用库

您可以通过编写系统调用库来完全避免主代码中的可移植性问题。为 FreeBSD 编写一个单独的库，为 Linux 编写一个不同的库，以及为更多操作系统编写其他库。

在你的库中，为每个系统调用编写一个单独的函数（或者如果您更喜欢传统的汇编语言术语，则为过程）。使用传递参数的 C 调用约定。但仍然使用 EAX 来传递调用号码。在这种情况下，你的 FreeBSD 库可以非常简单，因为许多看似不同的函数实际上只是指向相同代码的标签：

```
sys.open:
sys.close:
[etc...]
	int	80h
	ret
```

你的 Linux 库将需要更多不同的函数。但是即使在这里，你也可以使用相同数量的参数来分组系统调用：

```
sys.exit:
sys.close:
[etc... one-parameter functions]
	push	ebx
	mov	ebx, [esp+12]
	int	80h
	pop	ebx
	jmp	sys.return

...

sys.return:
	or	eax, eax
	js	sys.err
	clc
	ret

sys.err:
	neg	eax
	stc
	ret
```

起初，库方法可能看起来有些不方便，因为它要求你生成一个代码依赖的单独文件。但它有许多优点：首先，你只需要编写一次，并且可以在所有程序中使用。你甚至可以让其他汇编语言程序员使用它，或者使用别人写的库。但是，库最大的优点也许是你的代码可以被简单地移植到其他系统，甚至是其他程序员，只需编写一个新库，而无需对代码进行任何更改。

如果你不喜欢拥有一个库的想法，你至少可以把所有的系统调用放在一个单独的汇编语言文件中，并将其与你的主程序链接起来。在这里，所有的移植者只需要创建一个新的目标文件，以便与你的主程序链接起来。

### 使用包含文件

如果您将软件发布为(或与之一起发布的)源代码，您可以使用宏，并将其放在一个单独的文件中，然后在您的代码中包含它们。

您的软件的搬运工将简单地编写一个新的包含文件。不需要库或外部目标文件，但您的代码是可移植的，无需编辑代码。

|  | 这是我们将在本章中采用的方法。我们将命名我们的包含文件为 system.inc，并在处理新系统调用时添加内容。 |
| -- | ----------------------------------------------------------------------------------------------------- |

我们可以通过声明标准文件描述符来开始我们的 system.inc：

```
%define	stdin	0
%define	stdout	1
%define	stderr	2
```

接下来，我们为每个系统调用创建一个符号名称：

```
%define	SYS_nosys	0
%define	SYS_exit	1
%define	SYS_fork	2
%define	SYS_read	3
%define	SYS_write	4
; [etc...]
```

我们添加一个短的、非全局的过程，使用一个长名称，这样我们就不会在我们的代码中不小心重用该名称：

```
section	.text
align 4
access.the.bsd.kernel:
	int	80h
	ret
```

我们创建一个宏，它接受一个参数，即系统调用号码：

```
%macro	system	1
	mov	eax, %1
	call	access.the.bsd.kernel
%endmacro
```

最后，我们为每个系统调用创建宏。这些宏不带参数。

```
%macro	sys.exit	0
	system	SYS_exit
%endmacro

%macro	sys.fork	0
	system	SYS_fork
%endmacro

%macro	sys.read	0
	system	SYS_read
%endmacro

%macro	sys.write	0
	system	SYS_write
%endmacro

; [etc...]
```

继续，将其输入到您的编辑器中，并将其保存为 system.inc。随着我们讨论更多系统调用，我们将添加更多内容。

## A.6.我们的第一个程序

我们现在准备好进行我们的第一个程序，必不可少的 Hello, World！

```
	%include	'system.inc'

	section	.data
	hello	db	'Hello, World!', 0Ah
	hbytes	equ	$-hello

	section	.text
	global	_start
_start:
	push	dword hbytes
	push	dword hello
	push	dword stdout
	sys.write

	push	dword 0
	sys.exit
```

这里是它的功能：第 1 行包括了定义、宏以及来自 system.inc 的代码。

第 3-5 行是数据：第 3 行开始数据部分/段。第 4 行包含字符串"Hello, World!"，后跟一个换行符（ 0Ah ）。第 5 行创建一个包含第 4 行字符串长度（以字节计）的常量。

第 7-16 行包含代码。请注意，FreeBSD 使用 elf 文件格式作为其可执行文件，这要求每个程序都从标记为 _start 的点开始（更准确地说，链接器期望如此）。此标签必须是全局的。

第 10-13 行要求系统将 hello 字符串的 hbytes 字节写入 stdout 。

第 15-16 行要求系统以 0 的返回值结束程序。 SYS_exit 系统调用永远不会返回，因此代码在那里结束。

|  | 如果您来自 MS-DOS®汇编语言背景转向 UNIX®, 您可能习惯于直接编写视频硬件。在 FreeBSD 或任何其他 UNIX®版本中，您永远不必担心这一点。就您而言，您正在写入一个名为 stdout 的文件。这可以是视频屏幕，telnet 终端，实际文件，甚至另一个程序的输入。它是哪一个，是由系统来确定的。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### A.6.1。汇编代码

在编辑器中输入代码，并将其保存在名为 hello.asm 的文件中。您需要 nasm 来进行汇编。

#### 安装 nasm

如果您没有 nasm，请键入：

```
% su
Password:your root password
# cd /usr/ports/devel/nasm
# make install
# exit
%
```

如果你不想保留 nasm 源代码，可以键入 make install clean 而不仅仅是 make install 。

无论如何，FreeBSD 都将自动从互联网下载 nasm，并在您的系统上进行编译和安装。

|  | 如果您的系统不是 FreeBSD，则需要从其官方网站获取 nasm。您仍然可以使用它来汇编 FreeBSD 代码。 |
| -- | ---------------------------------------------------------------------------------------------- |

现在您可以汇编，链接和运行代码：

```
% nasm -f elf hello.asm
% ld -s -o hello hello.o
% ./hello
Hello, World!
%
```

## A.7. 编写 UNIX® 过滤器

常见的 UNIX® 应用程序类型是过滤器——一种从 stdin 读取数据，进行某种处理，然后将结果写入 stdout 的程序。

在本章中，我们将开发一个简单的过滤器，并学习如何从 stdin 读取和写入 stdout。这个过滤器将把输入的每个字节转换成一个十六进制数，并在其后跟一个空格。

```
%include	'system.inc'

section	.data
hex	db	'0123456789ABCDEF'
buffer	db	0, 0, ' '

section	.text
global	_start
_start:
	; read a byte from stdin
	push	dword 1
	push	dword buffer
	push	dword stdin
	sys.read
	add	esp, byte 12
	or	eax, eax
	je	.done

	; convert it to hex
	movzx	eax, byte [buffer]
	mov	edx, eax
	shr	dl, 4
	mov	dl, [hex+edx]
	mov	[buffer], dl
	and	al, 0Fh
	mov	al, [hex+eax]
	mov	[buffer+1], al

	; print it
	push	dword 3
	push	dword buffer
	push	dword stdout
	sys.write
	add	esp, byte 12
	jmp	short _start

.done:
	push	dword 0
	sys.exit
```

在数据部分，我们创建一个名为 hex 的数组。它按升序包含 16 个十六进制数字。该数组后面是一个缓冲区，我们将用于输入和输出。缓冲区的前两个字节最初设置为 0 。这是我们将写入两个十六进制数字的地方（第一个字节也是我们将读取输入的地方）。第三个字节是一个空格。

代码部分包括四个部分：读取字节、将其转换为十六进制数、写入结果，并最终退出程序。

要读取字节，我们要求系统从标准输入中读取一个字节，并将其存储在 buffer 的第一个字节中。系统返回读取的字节数为 EAX 。当数据正在传输时，这将是 1 ，或者当没有更多的输入数据可用时，这将是 0 。因此，我们检查 EAX 的值。如果是 0 ，我们跳转到 .done ，否则我们继续。

|  | 为简单起见，我们目前忽略错误条件的可能性。 |
| -- | -------------------------------------------- |

十六进制转换从 buffer 读取字节到 EAX ，或者实际上只是 AL ，同时将 EAX 的其余位清零。我们还将字节复制到 EDX ，因为我们需要分别转换上四位（半字节）和下四位。我们将结果存储在缓冲区的前两个字节中。

接下来，我们要求系统将缓冲区的三个字节，即两个十六进制数字和空格，写入标准输出。然后我们跳回程序的开头并处理下一个字节。

一旦没有更多的输入，我们要求系统退出我们的程序，返回零，这是传统意义上表示程序成功的值。

继续，将代码保存在名为 hex.asm 的文件中，然后输入以下内容（ ^D 表示按住控制键并在按住控制键的同时按 D ）：

```
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A ^D %
```

|  | 如果您从 MS-DOS®迁移到 UNIX®，您可能想知道为什么每行结尾都是 0A 而不是 0D 0A 。这是因为 UNIX®不使用 cr/lf 约定，而是使用“换行”约定，这在十六进制中是 0A 。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |

我们能改进这个吗？首先，有点令人困惑，因为一旦我们转换了一行文本，我们的输入就不再从行的开头开始了。我们可以修改它，在每个 0A 后打印一个新行而不是一个空格。

```
%include	'system.inc'

section	.data
hex	db	'0123456789ABCDEF'
buffer	db	0, 0, ' '

section	.text
global	_start
_start:
	mov	cl, ' '

.loop:
	; read a byte from stdin
	push	dword 1
	push	dword buffer
	push	dword stdin
	sys.read
	add	esp, byte 12
	or	eax, eax
	je	.done

	; convert it to hex
	movzx	eax, byte [buffer]
	mov	[buffer+2], cl
	cmp	al, 0Ah
	jne	.hex
	mov	[buffer+2], al

.hex:
	mov	edx, eax
	shr	dl, 4
	mov	dl, [hex+edx]
	mov	[buffer], dl
	and	al, 0Fh
	mov	al, [hex+eax]
	mov	[buffer+1], al

	; print it
	push	dword 3
	push	dword buffer
	push	dword stdout
	sys.write
	add	esp, byte 12
	jmp	short .loop

.done:
	push	dword 0
	sys.exit
```

我们已经将空格存储在 CL 寄存器中。我们可以这样做是因为，与 Microsoft® Windows®不同，UNIX®系统调用不会修改任何未使用的寄存器的值来返回值。

这意味着我们只需要设置 CL 一次。因此，我们添加了一个新标签 .loop 并跳转到下一个字节，而不是跳转到 _start 。我们还添加了 .hex 标签，这样我们可以在 buffer 的第三个字节中有一个空格或一个新行。

一旦您已经更改了 hex.asm 以反映这些更改，请键入：

```
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

看起来更好了。但这段代码效率很低！我们为每个字节两次（一次读取，一次写入输出）进行系统调用。

## A.8. 缓冲输入和输出

通过缓冲输入和输出，我们可以提高代码的效率。我们创建一个输入缓冲区，并一次性读取一整个字节序列。然后我们逐个从缓冲区中获取它们。

我们还创建一个输出缓冲区。我们将输出存储在其中，直到它满为止。在那时，我们要求内核将缓冲区的内容写入标准输出。

当没有更多输入时，程序结束。但我们仍然需要要求内核最后一次将输出缓冲区的内容写入标准输出，否则一些输出将进入输出缓冲区，但永远不会被发送出去。不要忘记这一点，否则你会想知道为什么有些输出丢失了。

```
%include	'system.inc'

%define	BUFSIZE	2048

section	.data
hex	db	'0123456789ABCDEF'

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
global	_start
_start:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

.loop:
	; read a byte from stdin
	call	getchar

	; convert it to hex
	mov	dl, al
	shr	al, 4
	mov	al, [hex+eax]
	call	putchar

	mov	al, dl
	and	al, 0Fh
	mov	al, [hex+eax]
	call	putchar

	mov	al, ' '
	cmp	dl, 0Ah
	jne	.put
	mov	al, dl

.put:
	call	putchar
	jmp	short .loop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword stdin
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; flush output buffer
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	sub	edi, ecx	; start of buffer
	push	ecx
	push	edi
	push	dword stdout
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; buffer is empty now
	ret
```

我们现在在源代码中有第三个部分，名为 .bss 。这个部分不包括在我们的可执行文件中，因此无法初始化。我们使用 resb 而不是 db 。它只是为我们保留请求大小的未初始化内存供我们使用。

我们利用系统不修改寄存器的事实：我们将寄存器用于原本需要存储在 .data 部分的全局变量。这也是为什么 UNIX®传递参数给系统调用的惯例是在堆栈上，优于 Microsoft 传递参数给寄存器的惯例：我们可以保留寄存器供我们自己使用。

我们使用 EDI 和 ESI 作为指向下一个要读取或写入的字节的指针。我们使用 EBX 和 ECX 来计算两个缓冲区中的字节数，这样我们就知道何时将输出转储到系统中，或者从系统中读取更多输入。

让我们看看现在它是如何工作的：

```
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
Here I come!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

不是你期望的吗？程序直到我们按下 ^D 后才打印输出。通过插入三行代码，每次我们将新行转换为 0A 时写出输出很容易解决。我已经用 > 标记了这三行（在您的 hex.asm 中不要复制 >）。

```
%include	'system.inc'

%define	BUFSIZE	2048

section	.data
hex	db	'0123456789ABCDEF'

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
global	_start
_start:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

.loop:
	; read a byte from stdin
	call	getchar

	; convert it to hex
	mov	dl, al
	shr	al, 4
	mov	al, [hex+eax]
	call	putchar

	mov	al, dl
	and	al, 0Fh
	mov	al, [hex+eax]
	call	putchar

	mov	al, ' '
	cmp	dl, 0Ah
	jne	.put
	mov	al, dl

.put:
	call	putchar
>	cmp	al, 0Ah
>	jne	.loop
>	call	write
	jmp	short .loop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword stdin
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; flush output buffer
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	sub	edi, ecx	; start of buffer
	push	ecx
	push	edi
	push	dword stdout
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; buffer is empty now
	ret
```

现在，让我们看看它是如何工作的：

```
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

对于一个 644 字节的可执行文件来说，还不错吧！

|  | 缓冲输入/输出的这种方法仍然存在隐患。当我谈论缓冲的负面影响时，我会讨论并修复它。 |
| -- | ----------------------------------------------------------------------------------- |

### A.8.1. 如何撤消字符

|  | 这可能是一个相对较高级的话题，主要适合熟悉编译器理论的程序员。如果您愿意，您可以跳到下一节，或者稍后阅读。 |
| -- | ------------------------------------------------------------------------------------------------------------ |

虽然我们的示例程序不需要，但更复杂的过滤器通常需要向前查看。换句话说，它们可能需要查看下一个字符是什么（甚至是多个字符）。如果下一个字符具有特定值，则它是当前正在处理的标记的一部分。否则，不是。

例如，您可能正在解析文本字符串的输入流（例如，在实现语言编译器时）：如果一个字符后面跟着另一个字符，或者可能是一个数字，则它是您正在处理的标记的一部分。如果它后面是空白字符或其他值，则它不是当前标记的一部分。

这提出了一个有趣的问题：如何将下一个字符返回给输入流，以便以后可以再次读取？

一个可能的解决方案是将其存储在一个字符变量中，然后设置一个标志。我们可以修改 getchar 来检查标志，如果设置了标志，就从该变量中获取字节，而不是从输入缓冲区中获取，并重新设置标志。但是，这当然会减慢我们的速度。

C 语言有一个 ungetc() 函数，专门用于这个目的。在阅读接下来的段落之前，是否有一种快速的方法可以在我们的代码中实现它？我希望你可以往上滚动一下，看看 getchar 过程，然后看看是否可以在阅读下一段之前找到一个好的、快速的解决方案。然后再回到这里，看看我的解决方案。

把字符返回到流中的关键在于我们最初如何获取字符：

首先，通过测试 EBX 的值来检查缓冲区是否为空。如果它为零，我们调用 read 过程。

如果我们有一个可用的字符，我们使用 lodsb ，然后减少 EBX 的值。 lodsb 指令与下面的指令实质上是相同的：

```
mov	al, [esi]
	inc	esi
```

我们获取的字节会一直保存在缓冲区中，直到下一次调用 read 。我们不知道何时会发生，但我们知道直到下一次调用 getchar 之前都不会发生。因此，要将最后读取的字节返回到流中，我们所需要做的就是减少 ESI 的值并增加 EBX 的值：

```
ungetc:
	dec	esi
	inc	ebx
	ret
```

但是，要小心！如果我们一次检查的先行字符多于一个，并连续调用 ungetc 多次，大多数情况下是可以工作的，但并非总是如此（而且调试起来会很困难）。为什么？

因为只要 getchar 不必调用 read ，所有预读取的字节仍然保存在缓冲区中，我们的 ungetc 就可以毫无问题地运行。但一旦 getchar 调用 read ，缓冲区的内容就会发生变化。

我们总是可以依赖于我们已读取的最后一个字符上的 ungetc 正常工作，但在那之前读取的任何内容上不能。

如果您的程序向前读取超过一个字节，您至少有两个选择：

如果可能，修改程序，使其只向前读取一个字节。这是最简单的解决方案。

如果该选项不可用，首先确定您的程序需要一次返回输入流的最大字符数。将该数字略微增加，以确保，最好是 16 的倍数，这样它就会很好地对齐。然后修改您的代码 .bss 部分，并在输入缓冲区之前创建一个小的“备用”缓冲区，类似于这样：

```
section	.bss
	resb	16	; or whatever the value you came up with
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE
```

您还需要修改 ungetc 以将要传递给 AL 的字节值传递进去：

```
ungetc:
	dec	esi
	inc	ebx
	mov	[esi], al
	ret
```

通过这种修改，您可以安全地连续调用 ungetc 高达 17 次（第一次调用仍在缓冲区内，其余 16 次可以在缓冲区内或“备用”内）。

## A.9. 命令行参数

如果我们的十六进制程序能够从命令行读取输入和输出文件的名称，那么它将会更加有用。但是...它们在哪里呢？

在 UNIX® 系统启动程序之前，它会在堆栈上存储一些数据，然后跳转到程序的标签处。是的，我说的是跳转，而不是调用。这意味着数据可以通过读取来访问，或者简单地 ping 它。

栈顶的值包含命令行参数的数量。传统上称为 argc ，代表“参数计数”。

接下来是命令行参数，共 argc 个。这些通常被称为 argv ，代表“参数值”。也就是说，我们得到 argv[0] ， argv[1] ， … ， argv[argc-1] 。这些不是实际的参数，而是参数的指针，即实际参数的内存地址。参数本身是以 NUL 结尾的字符串。

argv 列表后面跟着一个空指针，简单地说就是一个 0 。还有更多内容，但对我们目前的目的来说就足够了。

|  | 如果您来自 MS-DOS®编程环境，主要区别在于每个参数都是单独的字符串。第二个区别是，可以有无限数量的参数。 |
| -- | --------------------------------------------------------------------------------------------------------- |

有了这些知识，我们几乎已经准备好迎接 hex.asm 的下一个版本。但首先，我们需要向 system.inc 添加几行代码：

首先，我们需要向我们的系统调用号列表添加两个新条目：

```
%define	SYS_open	5
%define	SYS_close	6
```

然后我们在文件末尾添加两个新的宏：

```
%macro	sys.open	0
	system	SYS_open
%endmacro

%macro	sys.close	0
	system	SYS_close
%endmacro
```

因此，这是我们修改过的源代码：

```
%include	'system.inc'

%define	BUFSIZE	2048

section	.data
fd.in	dd	stdin
fd.out	dd	stdout
hex	db	'0123456789ABCDEF'

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
align 4
err:
	push	dword 1		; return failure
	sys.exit

align 4
global	_start
_start:
	add	esp, byte 8	; discard argc and argv[0]

	pop	ecx
	jecxz	.init		; no more arguments

	; ECX contains the path to input file
	push	dword 0		; O_RDONLY
	push	ecx
	sys.open
	jc	err		; open failed

	add	esp, byte 8
	mov	[fd.in], eax

	pop	ecx
	jecxz	.init		; no more arguments

	; ECX contains the path to output file
	push	dword 420	; file mode (644 octal)
	push	dword 0200h | 0400h | 01h
	; O_CREAT | O_TRUNC | O_WRONLY
	push	ecx
	sys.open
	jc	err

	add	esp, byte 12
	mov	[fd.out], eax

.init:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

.loop:
	; read a byte from input file or stdin
	call	getchar

	; convert it to hex
	mov	dl, al
	shr	al, 4
	mov	al, [hex+eax]
	call	putchar

	mov	al, dl
	and	al, 0Fh
	mov	al, [hex+eax]
	call	putchar

	mov	al, ' '
	cmp	dl, 0Ah
	jne	.put
	mov	al, dl

.put:
	call	putchar
	cmp	al, dl
	jne	.loop
	call	write
	jmp	short .loop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword [fd.in]
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; flush output buffer

	; close files
	push	dword [fd.in]
	sys.close

	push	dword [fd.out]
	sys.close

	; return success
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	sub	edi, ecx	; start of buffer
	push	ecx
	push	edi
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; buffer is empty now
	ret
```

在我们的 .data 部分，现在有两个新变量， fd.in 和 fd.out 。我们在这里存储输入和输出文件描述符。

在 .text 部分，我们已用 [fd.in] 和 [fd.out] 替换了对 stdin 和 stdout 的引用。

.text 部分现在以一个简单的错误处理程序开始，该处理程序只是以 1 的返回值退出程序。错误处理程序位于 _start 之前，因此我们离错误发生的地方不远。

当然，程序执行仍然从 _start 开始。首先，我们从堆栈中移除 argc 和 argv[0] ：它们对我们没有兴趣（至少在这个程序中是这样）。

我们将 argv[1] 弹出到 ECX 。这个寄存器特别适用于指针，因为我们可以处理空指针 jecxz 。如果 argv[1] 不为空，我们尝试打开第一个参数中命名的文件。否则，我们继续程序如前：从 stdin 读取，写入 stdout 。如果我们无法打开输入文件（例如，文件不存在），我们跳转到错误处理程序并退出。

如果一切顺利，我们现在检查第二个参数。如果存在，我们打开输出文件。否则，我们将输出发送到 stdout 。如果我们无法打开输出文件（例如，文件已存在且我们没有写入权限），我们再次跳转到错误处理程序。

代码的其余部分与之前相同，除了在退出之前关闭输入和输出文件，并且如前所述，我们使用 [fd.in] 和 [fd.out] 。

我们的可执行文件现在长达 768 字节。

我们还能改进它吗？当然可以！每个程序都可以改进。以下是我们可以做的一些想法：

* 让我们的错误处理程序向 stderr 打印一条消息。
* 将错误处理程序添加到 read 和 write 函数中。
* 当打开输入文件时关闭 stdin ，当打开输出文件时关闭 stdout 。
* 添加命令行开关，比如 -i 和 -o ，这样我们可以以任何顺序列出输入和输出文件，或者从 stdin 读取并写入文件。
* 如果命令行参数不正确，请打印使用消息。

我将把这些增强功能留给读者练习：你已经知道实现它们所需的一切。

## A.10. UNIX® 环境

UNIX®的一个重要概念是环境，由环境变量定义。一些是系统设置的，另一些是您设置的，还有一些是由shell或加载另一个程序的任何程序设置的。

### A.10.1. 如何查找环境变量

我之前说过，当一个程序开始执行时，堆栈包含 argc ，后跟以 NULL 结尾的 argv 数组，然后是其他内容。这个“其他内容”是环境，或者更准确地说，是指向环境变量的指针的以 NULL 结尾的数组。这经常被称为 env 。

env 的结构与 argv 的结构相同，都是一系列内存地址，后跟一个空值（ 0 ）。在这种情况下，没有 "envc" -我们通过搜索最终的空值来确定数组的结束位置。

变量通常以 name=value 格式出现，但有时 =value 部分可能丢失。我们需要考虑这种可能性。

### A.10.2. webvars

我可以展示一些代码，以与 UNIX® env 命令相同的方式打印环境。但我觉得编写一个简单的汇编语言 CGI 实用程序会更有趣。

#### A.10.2.1. CGI：快速概述

我在我的网站上有一个详细的 CGI 教程，但这里是 CGI 的一个非常快速概述：

* Web 服务器通过设置环境变量与 CGI 程序通信。
* CGI 程序将其输出发送到 stdout。Web 服务器从那里读取。
* 必须以 HTTP 标头开头，然后是两个空行。
* 然后打印 HTML 代码，或者它正在生成的其他类型的数据。

|  | 虽然某些环境变量使用标准名称，但其他变量取决于 Web 服务器。这使得 webvars 成为一种非常有用的诊断工具。 |
| -- | -------------------------------------------------------------------------------------------------------- |

#### A.10.2.2. 代码

因此，我们的 webvars 程序必须发送 HTTP 头，然后跟一些 HTML 标记。然后，它必须逐个读取环境变量，并将它们作为 HTML 页面的一部分发送出去。

代码如下。我将注释和解释直接放在代码内部：

```
;;;;;;; webvars.asm ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Copyright (c) 2000 G. Adam Stanislav
; All rights reserved.
;
; Redistribution and use in source and binary forms, with or without
; modification, are permitted provided that the following conditions
; are met:
; 1. Redistributions of source code must retain the above copyright
;    notice, this list of conditions and the following disclaimer.
; 2. Redistributions in binary form must reproduce the above copyright
;    notice, this list of conditions and the following disclaimer in the
;    documentation and/or other materials provided with the distribution.
;
; THIS SOFTWARE IS PROVIDED BY THE AUTHOR AND CONTRIBUTORS ``AS IS'' AND
; ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
; IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
; ARE DISCLAIMED.  IN NO EVENT SHALL THE AUTHOR OR CONTRIBUTORS BE LIABLE
; FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
; DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
; OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
; HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
; LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
; OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
; SUCH DAMAGE.
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Version 1.0
;
; Started:	 8-Dec-2000
; Updated:	 8-Dec-2000
;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
%include	'system.inc'

section	.data
http	db	'Content-type: text/html', 0Ah, 0Ah
	db	'<?xml version="1.0" encoding="utf-8"?>', 0Ah
	db	'<!DOCTYPE html PUBLIC "-//W3C/DTD XHTML Strict//EN" '
	db	'"DTD/xhtml1-strict.dtd">', 0Ah
	db	'<html xmlns="http://www.w3.org/1999/xhtml" '
	db	'xml.lang="en" lang="en">', 0Ah
	db	'<head>', 0Ah
	db	'<title>Web Environment</title>', 0Ah
	db	'<meta name="author" content="G. Adam Stanislav" />', 0Ah
	db	'</head>', 0Ah, 0Ah
	db	'<body bgcolor="#ffffff" text="#000000" link="#0000ff" '
	db	'vlink="#840084" alink="#0000ff">', 0Ah
	db	'<div class="webvars">', 0Ah
	db	'<h1>Web Environment</h1>', 0Ah
	db	'<p>The following <b>environment variables</b> are defined '
	db	'on this web server:</p>', 0Ah, 0Ah
	db	'<table align="center" width="80" border="0" cellpadding="10" '
	db	'cellspacing="0" class="webvars">', 0Ah
httplen	equ	$-http
left	db	'<tr>', 0Ah
	db	'<td class="name"><tt>'
leftlen	equ	$-left
middle	db	'</tt></td>', 0Ah
	db	'<td class="value"><tt><b>'
midlen	equ	$-middle
undef	db	'<i>(undefined)</i>'
undeflen	equ	$-undef
right	db	'</b></tt></td>', 0Ah
	db	'</tr>', 0Ah
rightlen	equ	$-right
wrap	db	'</table>', 0Ah
	db	'</div>', 0Ah
	db	'</body>', 0Ah
	db	'</html>', 0Ah, 0Ah
wraplen	equ	$-wrap

section	.text
global	_start
_start:
	; First, send out all the http and xhtml stuff that is
	; needed before we start showing the environment
	push	dword httplen
	push	dword http
	push	dword stdout
	sys.write

	; Now find how far on the stack the environment pointers
	; are. We have 12 bytes we have pushed before "argc"
	mov	eax, [esp+12]

	; We need to remove the following from the stack:
	;
	;	The 12 bytes we pushed for sys.write
	;	The  4 bytes of argc
	;	The EAX*4 bytes of argv
	;	The  4 bytes of the NULL after argv
	;
	; Total:
	;	20 + eax * 4
	;
	; Because stack grows down, we need to ADD that many bytes
	; to ESP.
	lea	esp, [esp+20+eax*4]
	cld		; This should already be the case, but let's be sure.

	; Loop through the environment, printing it out
.loop:
	pop	edi
	or	edi, edi	; Done yet?
	je	near .wrap

	; Print the left part of HTML
	push	dword leftlen
	push	dword left
	push	dword stdout
	sys.write

	; It may be tempting to search for the '=' in the env string next.
	; But it is possible there is no '=', so we search for the
	; terminating NUL first.
	mov	esi, edi	; Save start of string
	sub	ecx, ecx
	not	ecx		; ECX = FFFFFFFF
	sub	eax, eax
repne	scasb
	not	ecx		; ECX = string length + 1
	mov	ebx, ecx	; Save it in EBX

	; Now is the time to find '='
	mov	edi, esi	; Start of string
	mov	al, '='
repne	scasb
	not	ecx
	add	ecx, ebx	; Length of name

	push	ecx
	push	esi
	push	dword stdout
	sys.write

	; Print the middle part of HTML table code
	push	dword midlen
	push	dword middle
	push	dword stdout
	sys.write

	; Find the length of the value
	not	ecx
	lea	ebx, [ebx+ecx-1]

	; Print "undefined" if 0
	or	ebx, ebx
	jne	.value

	mov	ebx, undeflen
	mov	edi, undef

.value:
	push	ebx
	push	edi
	push	dword stdout
	sys.write

	; Print the right part of the table row
	push	dword rightlen
	push	dword right
	push	dword stdout
	sys.write

	; Get rid of the 60 bytes we have pushed
	add	esp, byte 60

	; Get the next variable
	jmp	.loop

.wrap:
	; Print the rest of HTML
	push	dword wraplen
	push	dword wrap
	push	dword stdout
	sys.write

	; Return success
	push	dword 0
	sys.exit
```

这段代码生成了一个 1,396 字节的可执行文件。其中大部分是数据，即我们需要发送的 HTML 标记。

照常组装和链接：

```
% nasm -f elf webvars.asm
% ld -s -o webvars webvars.o
```

要使用它，您需要将 webvars 上传到您的 Web 服务器。根据您的 Web 服务器设置的方式，您可能需要将其存储在一个特殊的 cgi-bin 目录中，或者可能需要使用 .cgi 扩展名重新命名它。

然后您需要使用浏览器查看其输出。要在我的 Web 服务器上查看其输出，请访问 http://www.int80h.org/webvars/。如果对密码保护的 Web 目录中存在的其他环境变量感到好奇，请访问 http://www.int80h.org/private/，使用用户名 asm 和密码 programmer 。

## A.11. 与文件一起工作

我们已经完成了一些基本的文件操作：我们知道如何打开和关闭它们，如何使用缓冲区读取和写入它们。但是在涉及文件时，UNIX®提供了更多功能。在本节中，我们将研究其中一些功能，并最终得到一个不错的文件转换实用程序。

的确，让我们从最后开始，也就是从文件转换实用程序开始。当我们从一开始就知道最终产品应该做什么时，编程变得更加容易。

我为 FreeBSD 编写的最早的程序之一是 tuc，一个文本到 FreeBSD 文件转换器。它将来自其他操作系统的文本文件转换为 FreeBSD 文本文件。换句话说，它会将不同类型的行结束符更改为 FreeBSD 的换行约定。它将输出保存在另一个文件中。可选地，它将 UNIX 文本文件转换为 DOS 文本文件。

我广泛使用 tuc，但始终只是将其他操作系统转换为 FreeBSD，从未反过来。我一直希望它可以直接覆盖文件，而不是我必须将输出发送到另一个文件。大多数时候，我最终会这样使用它：

```
% tuc myfile tempfile
% mv tempfile myfile
```

拥有一个 ftuc，即快速 tuc，然后像这样使用它：

```
% ftuc myfile
```

在这一章中，我们将用汇编语言编写 ftuc（原始 tuc 是用 C 编写的），并在此过程中研究各种面向文件的内核服务。

乍一看，这样的文件转换非常简单：你所要做的就是去掉回车，对吗？

如果你回答是，再想想：这种方法大多数时候都有效（至少对于 MS DOS 文本文件），但偶尔会失败。

问题在于，并非所有非 UNIX®文本文件都以回车/换行序列结束其行。 有些使用带有回车符但不带换行符。 其他将多个空行组合为单个回车符，然后是多个换行符。 诸如此类。

因此，文本文件转换器必须能够处理任何可能的行结束方式：

* 回车/换行
* 回车符
* 换行符 / 回车符
* 换行符

它还应该处理使用上述某种组合的文件（例如，回车符后面跟着几个换行符）。

### A.11.1. 有限状态机

问题很容易通过一种称为有限状态机的技术来解决，这种技术最初是由数字电子电路的设计者开发的。有限状态机是依赖于其上一个输入而不仅仅是依赖于其输入的数字电路，即依赖于其状态的数字电路。微处理器是有限状态机的一个例子：我们的汇编语言代码被组装成机器语言，其中一些汇编语言代码产生一个字节的机器语言，而其他则产生几个字节。当微处理器逐个从内存中提取字节时，其中一些字节仅仅改变其状态而不产生任何输出。当操作码的所有字节都被提取时，微处理器将产生一些输出，或者改变寄存器的值等。

由于这个原因，所有软件本质上都是微处理器的状态指令序列。然而，在软件设计中，有限状态机的概念也是有用的。

我们的文本文件转换器可以被设计为一个具有三种可能状态的有限状态机。我们可以称它们为状态 0-2，但如果我们给它们符号名称，将会让我们的生活更轻松：

* 普通
* cr
* lf

我们的程序将在普通状态下启动。在这种状态下，程序的动作取决于其输入，如下所示：

* 如果输入不是回车或换行符，则输入将被简单地传递到输出。状态保持不变。
* 如果输入是回车，则状态更改为"cr"。然后丢弃输入，即不产生输出。
* 如果输入是换行符，则状态更改为"lf"。然后丢弃输入。

每当我们处于 cr 状态时，那是因为上一个输入是回车符，而且未被处理。我们的软件在这种状态下的操作取决于当前的输入：

* 如果输入不是回车符或换行符，输出一个换行符，然后输出输入，然后将状态更改为普通状态。
* 如果输入是回车符，那么我们连续收到两个（或更多）回车符。我们丢弃输入，输出一个换行符，并保持状态不变。
* 如果输入是换行符，我们输出换行符并将状态更改为普通状态。请注意，这与上面的第一种情况不同 - 如果我们尝试将它们合并，我们将输出两个换行符而不是一个。

最后，在接收到不是由回车符前导的换行符后，我们处于 lf 状态。当我们的文件已经处于 UNIX®格式或者连续多行由单个回车符后跟多个换行符表示时，或者行以换行符/回车符序列结尾时，将会发生这种情况。这是我们在此状态下需要处理输入的方法：

* 如果输入不是回车符或换行符，我们输出一个换行符，然后输出输入，然后将状态更改为普通状态。这与在 cr 状态接收到相同类型的输入时完全相同的动作。
* 如果输入是回车符，则丢弃输入，输出换行符，然后将状态更改为普通状态。
* 如果输入是换行符，则输出换行符，并保持状态不变。

#### 最终状态

上面的有限状态机适用于整个文件，但会留下一个可能性，即最后一行的结尾会被忽略。当文件以单个回车符或单个换行符结尾时，这种情况会发生。我在编写 tuc 时没有考虑到这一点，后来才发现它偶尔会去掉最后一行的结尾。

这个问题很容易通过检查整个文件处理后的状态来解决。如果状态不正常，我们只需要输出一个最后的换行符。

|  | 现在我们已将我们的算法表达为一个有限状态机，我们很容易设计一个专用的数字电子电路（“芯片”）来为我们做转换。当然，这样做要比编写汇编语言程序昂贵得多。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### A.11.1.2. 输出计数器

因为我们的文件转换程序可能将两个字符合并为一个，所以我们需要使用一个输出计数器。我们将其初始化为 0 ，并在每次将字符发送到输出时增加它。在程序结束时，计数器将告诉我们需要将文件设置为多大。

### A.11.2. 在软件中实现有限状态机

使用有限状态机的最困难的部分是分析问题并将其表达为有限状态机。一旦完成，软件几乎可以自己编写。

在高级语言（如 C）中，有几种主要方法。一种是使用 switch 语句，选择应该运行哪个函数。例如，

```
switch (state) {
	default:
	case REGULAR:
		regular(inputchar);
		break;
	case CR:
		cr(inputchar);
		break;
	case LF:
		lf(inputchar);
		break;
	}
```

另一种方法是使用函数指针数组，类似于这样：

```
(output[state])(inputchar);
```

另一种方法是将 state 设置为函数指针，指向适当的函数：

```
(*state)(inputchar);
```

这是我们将在程序中使用的方法，因为它在汇编语言中非常容易实现，而且速度非常快。我们将简单地将正确过程的地址保存在 EBX 中，然后只需执行：

```
call	ebx
```

这可能比在代码中硬编码地址更快，因为微处理器不必从内存中获取地址——它已经存储在其寄存器之一中。我说可能是因为现代微处理器的缓存，无论哪种方式可能都一样快。

### A.11.3. 内存映射文件

由于我们的程序只能在单个文件上运行，我们无法使用以前适用于我们的方法，即从输入文件读取并写入输出文件。

UNIX®允许我们将文件或文件的一部分映射到内存中。为此，我们首先需要以适当的读/写标志打开文件。然后我们使用 mmap 系统调用将其映射到内存中。关于 mmap 的一个好处是它自动与虚拟内存一起工作：我们可以将文件的更多部分映射到内存中，即使我们的物理内存不足，仍然可以通过常规内存操作码（如 mov ， lods 和 stos ）访问它。我们对文件的内存映像所做的任何更改都将由系统写入文件。我们甚至不必保持文件处于打开状态：只要它保持映射状态，我们就可以从中读取并向其中写入。

32 位英特尔微处理器可以访问高达四千兆字节的内存 - 物理或虚拟。FreeBSD 系统允许我们将其中的一半用于文件映射。

为简单起见，在本教程中，我们将仅转换可以完全映射到内存中的文件。可能没有太多超过两千兆字节大小的文本文件。如果我们的程序遇到一个，它将简单地显示一条消息，建议我们使用原始 tuc。

如果您检查 syscalls.master 的副本，您将找到两个名为 mmap 的单独的系统调用。这是因为 UNIX®的演变：有传统的 BSD mmap ，系统调用 71。那个被 POSIX® mmap 取代，系统调用 197。FreeBSD 系统支持两者，因为旧程序是使用原始的 BSD 版本编写的。但新软件使用 POSIX®版本，这是我们将使用的版本。

syscalls.master 列出 POSIX® 版本的方式如下：

```
197	STD	BSD	{ caddr_t mmap(caddr_t addr, size_t len, int prot, \
			    int flags, int fd, long pad, off_t pos); }
```

这与 mmap(2) 所说的有轻微不同。这是因为 mmap(2) 描述的是 C 版本。

差别在于参数 long pad ，这在 C 版本中不存在。然而，FreeBSD syscalls 在 push 之后增加了一个 32 位的填充，用于填充 64 位参数。在这种情况下， off_t 是一个 64 位值。

当我们完成使用内存映射文件后，我们使用 munmap 系统调用取消映射：

|  | 要深入了解 mmap ，请参阅 W. Richard Stevens 的 Unix 网络编程，第 2 卷，第 12 章。 |
| -- | ----------------------------------------------------------------------------------- |

### A.11.4. 确定文件大小

因为我们需要告诉 mmap 要映射文件中多少字节到内存中，而且因为我们想要映射整个文件，我们需要确定文件的大小。

我们可以使用 fstat 系统调用来获取系统可以提供的有关打开文件的所有信息。这包括文件大小。

同样，syscalls.master 列出了 fstat 的两个版本，一个是传统版本（系统调用 62），另一个是 POSIX®版本（系统调用 189）。当然，我们将使用 POSIX®版本：

```
189	STD	POSIX	{ int fstat(int fd, struct stat *sb); }
```

这是一个非常直接的调用: 我们向它传递一个 stat 结构的地址和一个打开文件的描述符。它将填充 stat 结构的内容。

但是，我必须说，我试图在 .bss 部分声明 stat 结构，并且 fstat 不喜欢它: 它设置了指示错误的进位标志。在我将代码更改为在堆栈上分配结构之后，一切都正常了。

### A.11.5. 更改文件大小

由于我们的程序可能会将回车/换行序列合并为直接换行符，因此我们的输出可能会比输入小。但是，由于我们将输出放入与读取输入相同的文件中，我们可能需要更改文件的大小。

ftruncate 系统调用允许我们做到这一点。尽管其名称有些误导， ftruncate 系统调用既可以用于截断文件（使其变小），也可以用于扩展文件。

是的，我们将在 syscalls.master 中找到 ftruncate 的两个版本，一个是旧版本（130），另一个是新版本（201）。我们将使用新版本：

```
201	STD	BSD	{ int ftruncate(int fd, int pad, off_t length); }
```

请注意，这个再次包含一个 int pad 。

### A.11.6. ftuc

我们现在知道写入 ftuc 所需的一切。我们首先在 system.inc 中添加一些新行。首先，在文件的开头或附近定义一些常量和结构：

```
;;;;;;; open flags
%define	O_RDONLY	0
%define	O_WRONLY	1
%define	O_RDWR	2

;;;;;;; mmap flags
%define	PROT_NONE	0
%define	PROT_READ	1
%define	PROT_WRITE	2
%define	PROT_EXEC	4
;;
%define	MAP_SHARED	0001h
%define	MAP_PRIVATE	0002h

;;;;;;; stat structure
struc	stat
st_dev		resd	1	; = 0
st_ino		resd	1	; = 4
st_mode		resw	1	; = 8, size is 16 bits
st_nlink	resw	1	; = 10, ditto
st_uid		resd	1	; = 12
st_gid		resd	1	; = 16
st_rdev		resd	1	; = 20
st_atime	resd	1	; = 24
st_atimensec	resd	1	; = 28
st_mtime	resd	1	; = 32
st_mtimensec	resd	1	; = 36
st_ctime	resd	1	; = 40
st_ctimensec	resd	1	; = 44
st_size		resd	2	; = 48, size is 64 bits
st_blocks	resd	2	; = 56, ditto
st_blksize	resd	1	; = 64
st_flags	resd	1	; = 68
st_gen		resd	1	; = 72
st_lspare	resd	1	; = 76
st_qspare	resd	4	; = 80
endstruc
```

我们定义新的系统调用：

```
%define	SYS_mmap	197
%define	SYS_munmap	73
%define	SYS_fstat	189
%define	SYS_ftruncate	201
```

我们添加了宏以供使用：

```
%macro	sys.mmap	0
	system	SYS_mmap
%endmacro

%macro	sys.munmap	0
	system	SYS_munmap
%endmacro

%macro	sys.ftruncate	0
	system	SYS_ftruncate
%endmacro

%macro	sys.fstat	0
	system	SYS_fstat
%endmacro
```

这是我们的代码：

```
;;;;;;; Fast Text-to-Unix Conversion (ftuc.asm) ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;
;; Started:	21-Dec-2000
;; Updated:	22-Dec-2000
;;
;; Copyright 2000 G. Adam Stanislav.
;; All rights reserved.
;;
;;;;;;; v.1 ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
%include	'system.inc'

section	.data
	db	'Copyright 2000 G. Adam Stanislav.', 0Ah
	db	'All rights reserved.', 0Ah
usg	db	'Usage: ftuc filename', 0Ah
usglen	equ	$-usg
co	db	"ftuc: Can't open file.", 0Ah
colen	equ	$-co
fae	db	'ftuc: File access error.', 0Ah
faelen	equ	$-fae
ftl	db	'ftuc: File too long, use regular tuc instead.', 0Ah
ftllen	equ	$-ftl
mae	db	'ftuc: Memory allocation error.', 0Ah
maelen	equ	$-mae

section	.text

align 4
memerr:
	push	dword maelen
	push	dword mae
	jmp	short error

align 4
toolong:
	push	dword ftllen
	push	dword ftl
	jmp	short error

align 4
facerr:
	push	dword faelen
	push	dword fae
	jmp	short error

align 4
cantopen:
	push	dword colen
	push	dword co
	jmp	short error

align 4
usage:
	push	dword usglen
	push	dword usg

error:
	push	dword stderr
	sys.write

	push	dword 1
	sys.exit

align 4
global	_start
_start:
	pop	eax		; argc
	pop	eax		; program name
	pop	ecx		; file to convert
	jecxz	usage

	pop	eax
	or	eax, eax	; Too many arguments?
	jne	usage

	; Open the file
	push	dword O_RDWR
	push	ecx
	sys.open
	jc	cantopen

	mov	ebp, eax	; Save fd

	sub	esp, byte stat_size
	mov	ebx, esp

	; Find file size
	push	ebx
	push	ebp		; fd
	sys.fstat
	jc	facerr

	mov	edx, [ebx + st_size + 4]

	; File is too long if EDX != 0 ...
	or	edx, edx
	jne	near toolong
	mov	ecx, [ebx + st_size]
	; ... or if it is above 2 GB
	or	ecx, ecx
	js	near toolong

	; Do nothing if the file is 0 bytes in size
	jecxz	.quit

	; Map the entire file in memory
	push	edx
	push	edx		; starting at offset 0
	push	edx		; pad
	push	ebp		; fd
	push	dword MAP_SHARED
	push	dword PROT_READ | PROT_WRITE
	push	ecx		; entire file size
	push	edx		; let system decide on the address
	sys.mmap
	jc	near memerr

	mov	edi, eax
	mov	esi, eax
	push	ecx		; for SYS_munmap
	push	edi

	; Use EBX for state machine
	mov	ebx, ordinary
	mov	ah, 0Ah
	cld

.loop:
	lodsb
	call	ebx
	loop	.loop

	cmp	ebx, ordinary
	je	.filesize

	; Output final lf
	mov	al, ah
	stosb
	inc	edx

.filesize:
	; truncate file to new size
	push	dword 0		; high dword
	push	edx		; low dword
	push	eax		; pad
	push	ebp
	sys.ftruncate

	; close it (ebp still pushed)
	sys.close

	add	esp, byte 16
	sys.munmap

.quit:
	push	dword 0
	sys.exit

align 4
ordinary:
	cmp	al, 0Dh
	je	.cr

	cmp	al, ah
	je	.lf

	stosb
	inc	edx
	ret

align 4
.cr:
	mov	ebx, cr
	ret

align 4
.lf:
	mov	ebx, lf
	ret

align 4
cr:
	cmp	al, 0Dh
	je	.cr

	cmp	al, ah
	je	.lf

	xchg	al, ah
	stosb
	inc	edx

	xchg	al, ah
	; fall through

.lf:
	stosb
	inc	edx
	mov	ebx, ordinary
	ret

align 4
.cr:
	mov	al, ah
	stosb
	inc	edx
	ret

align 4
lf:
	cmp	al, ah
	je	.lf

	cmp	al, 0Dh
	je	.cr

	xchg	al, ah
	stosb
	inc	edx

	xchg	al, ah
	stosb
	inc	edx
	mov	ebx, ordinary
	ret

align 4
.cr:
	mov	ebx, ordinary
	mov	al, ah
	; fall through

.lf:
	stosb
	inc	edx
	ret
```

|  | 不要在由 MS-DOS® 或 Windows® 格式化的磁盘上使用此程序。 当在 FreeBSD 下使用 mmap 挂载在这些驱动器上时，可能存在一个细微的错误： 如果文件超过一定大小， mmap 将仅使用零填充内存，然后将其复制到文件中覆盖其内容。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## A.12. 一心一意

作为禅宗的学生，我喜欢一心一意的概念：一次只做一件事，并且做得好。

事实上，这正是 UNIX®的工作原理。虽然典型的 Windows®应用程序试图做任何可能的事情（因此充满错误），典型的 UNIX®程序只做一件事，并且做得很好。

典型的 UNIX®用户通过编写一个shell脚本，将各种现有程序组合在一起，将一个程序的输出导入到另一个程序的输入来实现自己的应用程序。

在编写您自己的 UNIX®软件时，通常建议查看您需要解决的问题的哪些部分可以通过现有程序处理，只为您没有现有解决方案的问题部分编写自己的程序。

### A.12.1. CSV

我将用我最近面对的一个具体的现实例子来说明这个原则：

我需要从我从一个网站下载的数据库中提取每条记录的第 11 个字段。该数据库是一个 CSV 文件，即一个逗号分隔值列表。这是一个在可能使用不同数据库软件的人之间共享数据的标准格式。

文件的第一行包含由逗号分隔的各种字段列表。文件的其余部分包含逐行列出的数据，值之间用逗号分隔。

我尝试使用逗号作为分隔符来使用 awk。但由于有几行包含带引号的逗号，awk 从这些行中提取了错误的字段。

因此，我需要编写自己的软件来从 CSV 文件中提取第 11 个字段。然而，遵循 UNIX® 精神，我只需要编写一个简单的过滤器，该过滤器将执行以下操作：

* 删除文件的第一行;
* 将所有未加引号的逗号更改为不同的字符;
* 删除所有引号。

严格来说，我可以使用 sed 从文件中删除第一行，但在我自己的程序中这样做非常容易，所以我决定这样做并减少流水线的大小。

无论如何，编写这样的程序大约花了我 20 分钟的时间。编写一个从 CSV 文件中提取第 11 个字段的程序会需要更长的时间，并且我无法重用它来从其他数据库提取其他字段。

这次我决定让它比典型的教程程序多做一点工作:

* 它解析其命令行以获取选项;
* 如果找到错误的参数，它会显示正确的用法;
* 它生成有意义的错误消息.

这是它的使用消息：

```
Usage: csv [-t<delim>] [-c<comma>] [-p] [-o <outfile>] [-i <infile>]
```

所有参数都是可选的，可以以任何顺序出现。

-t 参数声明用于替换逗号的内容。 tab 是默认设置。例如， -t; 将用分号替换所有未引用的逗号。

我不需要 -c 选项，但它将来可能会派上用场。它让我声明要用其他字符替换逗号。例如， -c@ 将替换所有的 at 符号（如果你想将电子邮件地址列表拆分为用户名和域名，这是很有用的）。

-p 选项保留了第一行，即不删除它。默认情况下，我们删除第一行，因为在 CSV 文件中它包含字段名而不是数据。

-i 和 -o 选项让我指定输入和输出文件。默认是 stdin 和 stdout，所以这是一个常规的 UNIX®过滤器。

我确保 -i filename 和 -ifilename 都被接受。我还确保只能指定一个输入文件和一个输出文件。

要获取每个记录的第 11 个字段，我现在可以这样做：

```
% csv '-t;' data.csv | awk '-F;' '{print $11}'
```

代码将选项（除了文件描述符）存储在 EDX 中： DH 中的逗号， DL 中的新分隔符，以及 -p 选项的标志位于 EDX 的最高位，因此检查其符号将让我们快速决定要做什么。

这里是代码：

```
;;;;;;; csv.asm ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Convert a comma-separated file to a something-else separated file.
;
; Started:	31-May-2001
; Updated:	 1-Jun-2001
;
; Copyright (c) 2001 G. Adam Stanislav
; All rights reserved.
;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

%include	'system.inc'

%define	BUFSIZE	2048

section	.data
fd.in	dd	stdin
fd.out	dd	stdout
usg	db	'Usage: csv [-t<delim>] [-c<comma>] [-p] [-o <outfile>] [-i <infile>]', 0Ah
usglen	equ	$-usg
iemsg	db	"csv: Can't open input file", 0Ah
iemlen	equ	$-iemsg
oemsg	db	"csv: Can't create output file", 0Ah
oemlen	equ	$-oemsg

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
align 4
ierr:
	push	dword iemlen
	push	dword iemsg
	push	dword stderr
	sys.write
	push	dword 1		; return failure
	sys.exit

align 4
oerr:
	push	dword oemlen
	push	dword oemsg
	push	dword stderr
	sys.write
	push	dword 2
	sys.exit

align 4
usage:
	push	dword usglen
	push	dword usg
	push	dword stderr
	sys.write
	push	dword 3
	sys.exit

align 4
global	_start
_start:
	add	esp, byte 8	; discard argc and argv[0]
	mov	edx, (',' << 8) | 9

.arg:
	pop	ecx
	or	ecx, ecx
	je	near .init		; no more arguments

	; ECX contains the pointer to an argument
	cmp	byte [ecx], '-'
	jne	usage

	inc	ecx
	mov	ax, [ecx]

.o:
	cmp	al, 'o'
	jne	.i

	; Make sure we are not asked for the output file twice
	cmp	dword [fd.out], stdout
	jne	usage

	; Find the path to output file - it is either at [ECX+1],
	; i.e., -ofile --
	; or in the next argument,
	; i.e., -o file

	inc	ecx
	or	ah, ah
	jne	.openoutput
	pop	ecx
	jecxz	usage

.openoutput:
	push	dword 420	; file mode (644 octal)
	push	dword 0200h | 0400h | 01h
	; O_CREAT | O_TRUNC | O_WRONLY
	push	ecx
	sys.open
	jc	near oerr

	add	esp, byte 12
	mov	[fd.out], eax
	jmp	short .arg

.i:
	cmp	al, 'i'
	jne	.p

	; Make sure we are not asked twice
	cmp	dword [fd.in], stdin
	jne	near usage

	; Find the path to the input file
	inc	ecx
	or	ah, ah
	jne	.openinput
	pop	ecx
	or	ecx, ecx
	je near usage

.openinput:
	push	dword 0		; O_RDONLY
	push	ecx
	sys.open
	jc	near ierr		; open failed

	add	esp, byte 8
	mov	[fd.in], eax
	jmp	.arg

.p:
	cmp	al, 'p'
	jne	.t
	or	ah, ah
	jne	near usage
	or	edx, 1 << 31
	jmp	.arg

.t:
	cmp	al, 't'		; redefine output delimiter
	jne	.c
	or	ah, ah
	je	near usage
	mov	dl, ah
	jmp	.arg

.c:
	cmp	al, 'c'
	jne	near usage
	or	ah, ah
	je	near usage
	mov	dh, ah
	jmp	.arg

align 4
.init:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

	; See if we are to preserve the first line
	or	edx, edx
	js	.loop

.firstline:
	; get rid of the first line
	call	getchar
	cmp	al, 0Ah
	jne	.firstline

.loop:
	; read a byte from stdin
	call	getchar

	; is it a comma (or whatever the user asked for)?
	cmp	al, dh
	jne	.quote

	; Replace the comma with a tab (or whatever the user wants)
	mov	al, dl

.put:
	call	putchar
	jmp	short .loop

.quote:
	cmp	al, '"'
	jne	.put

	; Print everything until you get another quote or EOL. If it
	; is a quote, skip it. If it is EOL, print it.
.qloop:
	call	getchar
	cmp	al, '"'
	je	.loop

	cmp	al, 0Ah
	je	.put

	call	putchar
	jmp	short .qloop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	jecxz	.read
	call	write

.read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword [fd.in]
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; flush output buffer

	; close files
	push	dword [fd.in]
	sys.close

	push	dword [fd.out]
	sys.close

	; return success
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	jecxz	.ret	; nothing to write
	sub	edi, ecx	; start of buffer
	push	ecx
	push	edi
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; buffer is empty now
.ret:
	ret
```

其中大部分取自上面的 hex.asm。但有一个重要的区别：我不再在输出换行符时调用 write 。然而，代码可以用于交互式使用。

自从我开始写这一章以来，我已经找到了交互性问题的更好解决方案。我希望确保每一行仅在需要时单独打印出来。毕竟，在非交互式使用时，没有必要冲洗每一行。

我现在使用的新解决方案是每次发现输入缓冲区为空时调用 write 。这样，在交互模式下运行时，程序会从用户键盘读取一行，处理它，并检查其输入缓冲区是否为空。它会刷新其输出并读取下一行。

#### A.12.1.1. 缓冲区的黑暗面

这个变化防止了一个非常特定情况下的神秘死机。我把它称为缓冲区的黑暗面，主要是因为它呈现了一个不太明显的危险。

它不太可能发生在像上面的 csv 程序中，因此让我们考虑另一个过滤器：在这种情况下，我们期望我们的输入是代表颜色值的原始数据，例如像素的红色、绿色和蓝色强度。我们的输出将是我们输入的负值。

这样的滤波器将非常容易编写。它的大部分看起来就像我们迄今为止编写的所有其他滤波器一样，所以我只会向您展示它的内部循环：

```
.loop:
	call	getchar
	not	al		; Create a negative
	call	putchar
	jmp	short .loop
```

因为这个滤波器使用原始数据，所以不太可能被交互使用。

但它可能被图像处理软件调用。而且，除非在每次调用 read 之前调用 write ，否则它可能会锁定。

这里可能会发生什么：

1. 图像编辑器将使用 C 函数 popen() 加载我们的滤镜。
2. 它将从位图或像素图中读取第一行像素。
3. 它将将第一行像素写入到通往我们滤波器的管道。
4. 我们的滤波器将从其输入中读取每个像素，将其转换为负像素，并将其写入其输出缓冲区。
5. 我们的过滤器将调用 getchar 来获取下一个像素。
6. getchar 将找到一个空的输入缓冲区，因此它将调用 read 。
7. read 将调用 SYS_read 系统调用。
8. 内核将暂停我们的过滤器，直到图像编辑器向管道发送更多数据。
9. 图像编辑器将从连接到我们过滤器的另一个管道中读取，以便在向我们发送输入的第二行之前，它可以设置输出图像的第一行。
10. 内核暂停图像编辑器，直到它从我们的过滤器接收到一些输出，以便将其传递给图像编辑器。

在这一点上，我们的过滤器等待图像编辑器发送更多数据以进行处理，而图像编辑器正在等待我们的过滤器发送第一行处理结果。但结果存储在我们的输出缓冲区中。

过滤器和图像编辑器将继续无限期地等待彼此（或者至少直到它们被终止）。我们的软件刚刚进入了竞争条件。

如果我们的过滤器在请求内核提供更多输入数据之前刷新其输出缓冲区，则不会出现此问题。

## 使用 FPU

奇怪的是，大部分汇编语言文献甚至没有提到 FPU（浮点运算单元）的存在，更不用说讨论如何编程了。

然而，当我们通过汇编语言做一些只有汇编语言才能做到的事情，创建高度优化的 FPU 代码时，汇编语言的光芒就会显现出来。

### FPU 的组织

FPU 由 8 个 80 位浮点寄存器组成。这些寄存器以堆栈方式组织-您可以在 TOS（堆栈顶部）上 push 一个值，也可以 pop 它。

也就是说，汇编语言操作码不是 push 和 pop ，因为它们已经被占用。

您可以通过使用 fld 、 fild 和 fbld 在 TOS 上设置一个值。 几个其他操作码让您在 TOS 上设置许多常见的常量-比如 pi。

类似地，您可以通过使用 fst 、 fstp 、 fist 、 fistp 和 fbstp 来设置一个值。 实际上，只有以 p 结尾的操作码才会直接设置该值， 其他操作码会将其移动到另一个地方而不从 TOS 中移除。

我们可以在 TOS 和计算机内存之间传输数据，格式可以是 32 位、64 位或 80 位实数，16 位、32 位或 64 位整数，或 80 位打包十进制数。

80 位打包十进制是二进制编码十进制的特例，在将数据的 ASCII 表示和 FPU 内部数据之间转换时非常方便。它允许我们使用 18 个有效数字。

无论我们如何在内存中表示数据，FPU 始终将其存储在其寄存器中的 80 位实数格式中。

其内部精度至少为 19 位十进制数字，因此即使我们选择以 ASCII 形式以完整的 18 位精度显示结果，我们仍在显示正确的结果。

我们可以对 TOS 执行数学运算：我们可以计算它的正弦，我们可以缩放它（即，我们可以乘以或除以 2 的幂），我们可以计算它的以 2 为底的对数，以及许多其他事情。

我们还可以将其乘以或除以，加到或从 FPU 寄存器中减去（包括它本身）。

TOS 的官方英特尔操作码是 st ，寄存器 st(0) - st(7) 的操作码是 st 和 st(0) ，因此，它们指的是同一个寄存器。

无论出于何种原因，nasm 的原始作者决定使用不同的操作码，即 st0 - st7 。换句话说，没有括号，TOS 总是 st0 ，而不是仅仅 st 。

#### A.13.1.1。压缩十进制格式

压缩十进制格式使用 10 字节（80 位）的内存来表示 18 位数。那里代表的数字总是一个整数。

|  | 通过首先将 TOS 乘以 10 的幂来获得小数位。 |
| -- | ------------------------------------------- |

最高字节（第 9 字节）的最高位是符号位：如果设置了，数字为负，否则为正。此字节的其余位未使用/被忽略。

剩余的 9 个字节存储数字的 18 位数：每个字节 2 位数。

更高位的数字存储在高半字节（4 位）中，较低位的数字存储在低半字节中。

也就是说，您可能会认为 -1234567 会以这种方式存储在内存中（使用十六进制表示法）：

```
80 00 00 00 00 00 01 23 45 67
```

可惜不是！与其他所有英特尔制造的东西一样，即使是打包的十进制数也是小端序的。

这意味着我们的 -1234567 存储方式如下：

```
67 45 23 01 00 00 00 00 00 80
```

记住这一点，否则你会绝望得抓狂！

|  | 如果你能找到的话，推荐阅读的书是 Richard Startz 的《8087/80287/80387 for the IBM PC & Compatibles》。尽管它似乎默认了紧凑十进制的小端存储事实。在我之前展示的过滤器出问题之前，我真的不是在开玩笑，试图弄清楚哪里出了问题，直到我意识到即使对于这种类型的数据，我也应该尝试小端顺序。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### A.13.2. 寻找针孔摄影之旅

要写出有意义的软件，我们不仅必须了解我们的编程工具，还必须了解我们为之创建软件的领域。

我们下一个滤镜会在我们想要构建针孔相机时帮助我们，因此在继续之前，我们需要一些针孔摄影方面的背景知识。

#### A.13.2.1. 相机

描述任何已建造的相机最简单的方法是将其描述为一些空间被一些防光材料包围，包围物中有一个小孔。

包围物通常很坚固（例如，一个盒子），但有时它是灵活的（比如折叠式相机）。相机内部非常黑暗。然而，孔让光线通过单个点进入（尽管在某些情况下可能有几个）。这些光线形成一个图像，在孔前面呈现相机外部的任何东西的表示。

如果将一些光敏材料（如胶片）放入相机中，它可以捕捉图像。

孔往往包含一个透镜，或一个透镜组件，通常称为物镜。

#### A.13.2.2. 孔针

但严格来说，镜头并非必需：原始相机并不使用镜头，而是针孔。即使在今天，针孔仍然被用作研究相机工作原理的工具，并实现特殊类型的图像。

针孔产生的图像完全清晰。或模糊。针孔的理想尺寸是有的：如果太大或太小，图像就会失去清晰度。

#### A.13.2.3. 焦距

这个理想的小孔直径是焦距的平方根的函数，焦距是小孔到胶片的距离。

```
D = PC * sqrt(FL)
```

在这里， D 是小孔的理想直径， FL 是焦距， PC 是小孔常数。根据杰伊·本德的说法，其值为 0.04 ，而肯尼斯·康纳斯确定为 0.037 。其他人提出了其他值。此值仅适用于白天：其他类型的光将需要不同的常数，其值只能通过实验确定。

#### A.13.2.4. 光圈数

f 数是测量光线照射胶片的非常有用的指标。例如，光度计可以确定，为了以 f5.6 的光圈值暴露具有特定感光度的胶片，可能需要曝光持续 1/1000 秒。

无论是 35 毫米相机、6x9 厘米相机等，都无关紧要。只要我们知道 f 数，就可以确定适当的曝光。

f 数很容易计算：

```
F = FL / D
```

换句话说，焦距除以针孔直径等于光圈数。这也意味着更高的光圈数意味着更小的针孔或更大的焦距，或者两者都有。这反过来暗示，光圈数越高，曝光时间就越长。

此外，虽然针孔直径和焦距是一维测量，但胶片和针孔都是二维的。这意味着如果您以 A 的光圈数测量了曝光为 t ，那么 B 的曝光为：

```
t * (B / A)²
```

#### A.13.2.5. 标准化光圈数

尽管许多现代相机可以平滑而逐渐地改变他们的针孔直径，从而改变其光圈值，但并非总是如此。

为了允许不同的光圈值，相机通常包含一个金属板，上面钻有几个不同尺寸的孔。

这些尺寸是根据上述公式选择的，以使得最终的光圈值是所有相机上都使用的标准光圈值之一。例如，我手头上有一台非常老旧的柯达 Duaflex IV 相机，它有三个这样的孔，用于光圈值 8、11 和 16。

最近制造的相机可能提供 2.8、4、5.6、8、11、16、22 和 32（以及其他）的光圈值。这些数值并非随意选择：它们都是 2 的平方根的幂，尽管它们可能会有所四舍五入。

#### A.13.2.6. 光圈

典型相机设计成设置任何标准化光圈值会改变拨盘的感觉。它会自然停在那个位置。因此，这些拨盘位置被称为光圈值。

由于每个停止点的 f 数是 2 的平方根的幂，将表盘移动 1 个停止点将使所需的适当曝光量加倍。将其移动 2 个停止点将使所需的曝光量增加 4 倍。将表盘移动 3 个停止点将使曝光量增加 8 倍，依此类推。

### A.13.3. 设计针孔软件

现在我们准备决定我们的针孔软件究竟应该做什么。

#### 处理程序输入

由于其主要目的是帮助我们设计一个有效的针孔相机，我们将焦距作为程序的输入。这是我们可以在没有软件的情况下确定的事情：适当的焦距由胶片的大小和拍摄“常规”照片、广角照片或长焦照片的需要确定。

到目前为止，我们编写的大多数程序都是使用单个字符或字节作为它们的输入：十六进制程序将单个字节转换为十六进制数，csv 程序要么让一个字符通过，要么删除它，要么将其更改为不同的字符，等等。

一个程序，ftuc 使用状态机一次考虑最多两个输入字节。

但我们的针孔程序不能只处理单个字符，它必须处理更大的句法单元。

例如，如果我们希望程序在焦距为 100 mm ， 150 mm 和 210 mm 时计算针孔直径（以及我们稍后将讨论的其他值），我们可能想输入类似于这样的内容：

```
 100, 150, 210
```

我们的程序需要一次考虑不止一个字节的输入。当它看到第一个 1 时，它必须理解它正在看到一个十进制数字的第一个数字。当它看到 0 和其他 0 时，它必须知道它正在看到同一数字的更多数字。

当它遇到第一个逗号时，它必须知道它不再接收第一个数字的数字。它必须能够将第一个数字的数字转换为 100 的值。第二个数字的数字转换为 150 的值。当然，第三个数字的数字转换为 210 的数值。

我们需要决定接受哪些分隔符：输入的数字必须用逗号分隔吗？如果是这样，我们如何处理由其他东西分隔的两个数字？

就我个人而言，我喜欢保持简单。东西要么是一个数字，所以我处理它。要么不是一个数字，所以我丢弃它。当我明明是多输入了一个字符时，我不喜欢计算机抱怨我。唉！

而且，它让我打破了单调的计算，而不是只输入一个数字：

```
What is the best pinhole diameter for the
	    focal length of 150?
```

计算机没有理由吐出一堆抱怨：

```
Syntax error: What
Syntax error: is
Syntax error: the
Syntax error: best
```

等等，等等，等等。

其次，我喜欢使用 # 字符来表示从该行开始到末尾的注释。这样做不需要太多的编码工作，并且让我可以将我的软件的输入文件视为可执行脚本。

在我们的情况下，我们还需要决定输入应该以什么单位进行：我们选择毫米，因为这是大多数摄影师测量焦距的方式。

最后，我们需要决定是否允许使用小数点（在这种情况下，我们还必须考虑到世界上许多地方使用小数逗号）。

在我们的情况下，允许使用小数点/逗号会提供一种虚假的精确感： 50 和 51 的焦距几乎没有什么明显的区别，因此允许用户输入类似 50.5 这样的内容并不是一个好主意。这只是我的观点，当然，我是写这个程序的人。在你的程序中，你可以做出其他选择。

#### A.13.3.2. 提供选项

构建针孔相机时，我们需要知道的最重要的事情是针孔的直径。由于我们希望拍摄清晰的图像，我们将使用上述公式从焦距计算针孔直径。由于专家们为 PC 常数提供了几个不同的值，我们需要做出选择。

在 UNIX®编程中，传统做法是有两种主要选择程序参数的方式，以及在用户没有做出选择时有一个默认值。

为什么要有两种选择方式？

一个选择是允许一个（相对）永久的选择，每次软件运行时都会自动应用，而无需一遍又一遍地告诉它我们想要它做什么。

永久选择可能存储在配置文件中，通常位于用户的主目录中。该文件通常与应用程序同名，但以点号开头。通常在文件名后面添加"rc"。因此，我们的文件可能是~/.pinhole 或~/.pinholerc。（~表示当前用户的主目录。）

配置文件主要由具有许多可配置参数的程序使用。那些只有一个（或几个）可配置参数的程序通常使用不同的方法：它们希望在环境变量中找到该参数。在我们的情况下，我们可能会查看名为 PINHOLE 的环境变量。

通常，程序使用以上方法之一。否则，如果配置文件说一件事，但环境变量说另一件事，程序可能会感到困惑（或者只是太复杂了）。

因为我们只需要选择一个这样的参数，我们将使用第二种方法并搜索环境变量，查找名为 PINHOLE 的变量。

另一种方式允许我们做即兴决定：“虽然我通常希望你使用 0.039，但是这一次我希望使用 0.03872。” 换句话说，它允许我们覆盖永久选择。

这种类型的选择通常是通过命令行参数完成的。

最后，程序总是需要一个默认值。用户可能不做任何选择。也许他不知道该选择什么。也许他只是"随便看看"。最好，默认值将是大多数用户可能选择的值。这样他们就不需要选择了。或者，更确切地说，他们可以毫不费力地选择默认值。

鉴于这个系统，程序可能会发现冲突的选项，并以这种方式处理它们：

1. 如果找到临时选择（例如命令行参数），应接受该选择。必须忽略任何永久选择和任何默认值。
2. 否则，如果找到永久选项（例如环境变量），应接受它，并忽略默认值。
3. 否则，应使用默认设置。

我们还需要决定我们的 PC 选项应该采用什么格式。

乍一看，使用 PINHOLE=0.04 格式作为环境变量似乎是显而易见的，而 -p0.04 则用于命令行。

允许这样做实际上是一种安全风险。 PC 常数是一个非常小的数字。当然，我们将使用各种小值的 PC 来测试我们的软件。但是如果有人选择一个巨大的值来运行程序会发生什么？

它可能会使程序崩溃，因为我们没有设计它来处理大数字。

或者，我们可能会花更多的时间来让程序能够处理大数字。如果我们为计算机文盲受众编写商业软件，我们可能会这样做。

或者，我们可以说，“难道不应该是用户更了解吗？”

或者，我们可能会让用户无法输入一个巨大的数字。这是我们将采取的方法：我们将使用一个隐含的 0. 前缀。

换句话说，如果用户想要 0.04 ，我们将期望他输入 -p04 ，或在他的环境中设置 PINHOLE=04 。所以，如果他说 -p9999999 ，我们将解释为 0.9999999 - 仍然荒谬，但至少更安全。

其次，许多用户只想选择贝德常数或康纳斯常数。为了让他们更容易，我们将解释 -b 为与 -p04 相同， -c 为与 -p037 相同。

#### A.13.3.3. 输出

我们需要决定我们希望软件发送到输出的内容，以及以何种格式。

由于我们的输入允许未指定数量的焦距条目，因此最好使用传统的数据库风格输出，即在单独的行上显示每个焦距的计算结果，同时通过 tab 字符在一行上分隔所有值。

可选的，我们还应该允许用户指定我们之前研究过的 CSV 格式的使用。在这种情况下，我们将打印出一行逗号分隔的名称，描述每行的每个字段，然后像以前一样显示我们的结果，但用 comma 替换 tab 。

我们需要一个用于 CSV 格式的命令行选项。我们不能使用 -c ，因为那已经意味着使用康纳斯常数。出于某种奇怪的原因，许多网站将 CSV 文件称为“Excel 电子表格”（尽管 CSV 格式比 Excel 更早）。因此，我们将使用 -e 开关通知我们的软件我们希望以 CSV 格式输出。

我们将在输出的每一行开头写上焦距。起初，这可能听起来有些重复，特别是在交互模式中：用户输入焦距，我们正在重复它。

但用户可以在一行上输入多个焦距。输入也可以来自文件或另一个程序的输出。在这种情况下，用户根本看不到输入。

同样，输出可以写入文件，我们将稍后检查，或者可以发送到打印机，或成为另一个程序的输入。

因此，每一行以用户输入的焦距为开头是完全合理的。

不，等等！不是由用户输入的。如果用户输入类似这样的东西：

```
 00000000150
```

显然，我们需要去掉那些前导零。

因此，我们可以考虑原样读取用户输入，在 FPU 内将其转换为二进制，然后从那里打印出来。

 但是...

如果用户输入类似这样的内容：

```
 17459765723452353453534535353530530534563507309676764423
```

哈！打包的十进制浮点数格式让我们能输入 18 位数。但用户输入了超过 18 位数。我们该如何处理？

好吧，我们可以修改我们的代码，读取前 18 位数字，输入到 FPU 中，然后读取更多，将我们已经在 TOS 上拥有的数字乘以 10 的附加数字数量，然后 add 到它。

是的，我们可以这样做。但在这个程序中这将是荒谬的（在另一个程序中可能是应该做的事情）：即使以毫米表示的地球周长只有 11 位数字。显然，我们无法制造那么大的相机（至少目前还不能）。

因此，如果用户输入如此巨大的数字，他要么是无聊的，要么是在测试我们，要么是试图入侵系统，要么是在玩游戏——做任何事情，但不是设计针孔相机。

我们将做什么？

我们会打他的脸，就说的方式：

```
17459765723452353453534535353530530534563507309676764423	???	???	???	???	???
```

为了实现这一点，我们将简单地忽略任何前导零。一旦我们找到一个非零数字，我们将初始化一个计数器为 0 ，并开始采取三个步骤：

1. 发送数字到输出。
2. 将数字附加到缓冲区，稍后我们将使用它来生成发送至 FPU 的打包十进制数。
3. 增加计数器。

现在，在我们采取这三个步骤的同时，我们也需要注意两种情况之一：

* 如果计数器增长超过 18，我们停止将内容附加到缓冲区。我们继续阅读数字并将它们发送到输出。
* 如果，或者更准确地说，下一个输入字符不是数字，那么我们暂时停止输入。顺便说一句，我们可以简单地丢弃非数字，除非它是 # ，这时我们必须返回到输入流。它标志着一条评论，因此在我们生成输出并开始查找更多输入之后，必须看到它。

这仍然留下了一个未被发现的可能性：如果用户输入的全部是零（或者是多个零），我们将永远找不到一个非零数来显示。

每当我们的计数器停留在 0 时，我们可以确定发生了这种情况。在这种情况下，我们需要将 0 发送到输出，并执行另一个“打击面部”：

```
0	???	???	???	???	???
```

一旦我们显示了焦距并确定它有效（大于 0 但不超过 18 位数），我们可以计算针孔直径。

并非巧合，针孔中包含“针”这个词。实际上，许多针孔确实是针孔，是用针尖小心打孔的。

那是因为典型的针孔非常小。我们的公式得到的结果是毫米。我们将其乘以 1000 ，以便我们可以输出微米的结果。

此时，我们面临另一个陷阱：过多的精度。

是的，FPU 是为高精度数学设计的。但我们不是在处理高精度数学。我们正在处理物理学（特别是光学）。

假设我们想要将一辆卡车改装成针孔相机（我们不会是第一个这样做的人！）。假设它的箱子长 12 米，那么我们有焦距 12000 。好吧，使用贝德尔常数，它给出了 12000 的平方根乘以 0.04 ，这是 4.381780460 毫米，或 4381.780460 微米。

无论如何陈述，结果都是荒谬地精确。我们的卡车不确切是 12000 毫米长。我们没有用如此精确的方式测量它的长度，所以声明我们需要直径为 4.381780460 毫米的针孔是，嗯，具有欺骗性的。 4.4 毫米完全足够。

|  | 在上面的例子中，我只使用了十个数字。想象一下去使用所有 18 个数字的荒谬之处！ |
| -- | ------------------------------------------------------------------------------ |

我们需要限制结果的有效数字位数。一种方法是使用表示微米的整数。因此，我们的卡车需要直径为 4382 微米的针孔。看着那个数字，我们仍然决定 4400 微米，或 4.4 毫米足够接近。

另外，我们可以决定无论结果有多大，我们只想显示四个有效数字（当然也可以是其他数字）。遗憾的是，FPU 不提供将数字四舍五入到特定位数的功能（毕竟，它不将数字视为十进制，而是视为二进制）。

因此，我们必须设计一种算法来减少有效数字的数量。

这是我的（我觉得很尴尬-如果您知道一个更好的，请告诉我）：

1. 将计数器初始化为 0 。
2. 当数字大于或等于 10000 时，将其除以 10 并增加计数器。
3. 输出结果。
4. 当计数器大于 0 时，输出 0 并减少计数器。

|  | 如果您只需要四个有效数字，那么 10000 就很好。对于其他数量的有效数字，请用 10 的 10 次幂替换 10000 。 |
| -- | ------------------------------------------------------------------------------------------------------ |

然后，我们将输出以微米为单位四个有效数字的针孔直径。

此时，我们已知焦距和针孔直径。这意味着我们有足够的信息来计算光圈值。

我们将显示四个有效数字的 f 数，四舍五入。f 数很可能告诉我们很少。为了使其更有意义，我们可以找到最接近的标准化 f 数，即最接近的平方根 2 的幂。

我们通过将实际 f 数乘以自身来做到这一点，这当然会给我们 square 。然后我们将计算其以 2 为底的对数，这比计算以平方根 2 为底的对数要容易得多！我们将结果四舍五入到最接近的整数。接下来，我们将 2 提高到结果。实际上，FPU 为我们提供了一个很好的快捷方式来做到这一点：我们可以使用 fscale op 代码来“缩放”1，这类似于 shift 一个整数向左。最后，我们计算所有这些的平方根，然后我们就有了最接近的标准化 f 数。

如果所有这些听起来令人不知所措——或者工作太多，也许——如果您看到代码，一切都会变得更加清晰。总共需要 9 个操作码：

```
fmul	st0, st0
	fld1
	fld	st1
	fyl2x
	frndint
	fld1
	fscale
	fsqrt
	fstp	st1
```

第一行， fmul st0, st0 ，平方了 TOS（堆栈顶部，与 st 相同，由 nasm 称为 st0 ）。 fld1 将 1 推送到 TOS 上。

接下来一行， fld st1 ，将平方推送回 TOS。此时，平方同时位于 st 和 st(2) 中（为什么我们在堆栈上留下第二个副本将很快明白）。 st(1) 包含 1 。

接下来， fyl2x 计算 st 乘以 st(1) 的以 2 为底的对数。这就是为什么我们之前将 1 放在 st(1) 上的原因。

到这一点， st 包含我们刚刚计算的对数， st(1) 包含我们以后保存的实际 f-number 的平方。

frndint 将 TOS 四舍五入到最近的整数。 fld1 推一个 1 。 fscale 将 TOS 上的 1 按 st(1) 中的值移位，有效地将 2 提高到 st(1) 次方。

最后， fsqrt 计算结果的平方根，即最近的归一化 f-number。

我们现在在 TOS 上有了最接近标准化的 f-数，以 st(1) 为底的对数四舍五入到最接近的整数，实际 f-数的平方取值为 st(2) 。我们将值保存在 st(2) 中以待后用。

但我们不再需要 st(1) 的内容。最后一行， fstp st1 ，将 st 的内容放入 st(1) 中，并弹出。结果，原本是 st(1) 的现在是 st ，原本是 st(2) 的现在是 st(1) ，依此类推。新的 st 包含了标准化的 f-数。新的 st(1) 包含了我们为后人存储的实际 f-数的平方。

此时，我们已经准备好输出标准化的 f-数。由于它已经标准化，我们不会将其四舍五入到四个有效数字，而是以完整精度发送出去。

标准化光圈值在光度计上很有用，只要它足够小并且可以找到。否则，我们需要另一种方法来确定适当的曝光。

我们之前已经找出了在任意光圈值处计算适当曝光的公式，该公式是根据在不同光圈值处测得的曝光值得出的。

我见过的每个光度计都可以确定 f5.6 处的适当曝光。因此，我们将计算“f5.6 倍增器”，即我们需要将在 f5.6 处测得的曝光乘以多少来确定我们针孔相机的适当曝光。

根据上述公式，我们知道这个因子可以通过将我们的 f 数（实际的数，而不是标准化的数）除以 5.6 ，然后将结果平方来计算。

从数学上讲，将我们的 f 数的平方除以 5.6 的平方将给我们相同的结果。

在计算上，当我们只能平方一个数字时，我们不想平方两个数字。因此，第一个解决方案一开始似乎更好。

 但是…

5.6 是一个常数。我们不必让我们的 FPU 浪费宝贵的周期。我们可以告诉它将 f-数的平方除以 5.6² 等于多少。或者我们可以将 f-数除以 5.6 ，然后平方结果。现在这两种方法看起来是相等的。

但是，它们并不相等！

经过以上摄影原理的研究，我们记得 5.6 实际上是 2 的平方根的五次方。一个无理数。这个数的平方恰好是 32 。

32 不仅是一个整数，它是 2 的幂。我们不需要将光圈值的平方除以 32 。我们只需要用 fscale 右移五位。在 FPU 计算中，这意味着我们将{{$3}}与 st(1) 相乘，等于 -5 。这比除法快得多。

现在我们清楚为什么将光圈值的平方保存在 FPU 堆栈的顶部。计算 f5.6 倍率是整个程序中最容易的计算！我们将输出它四个有效数字四舍五入。

我们可以计算一个更有用的数字：我们的光圈数与 f5.6 相差的档数。如果我们的光圈数恰好在测光表范围之外，但我们的快门可以设置不同的速度，并且这个快门使用档位，这个数字可能会对我们有所帮助。

假设我们的光圈数与 f5.6 相差 5 档，测光表显示我们应该使用 1/1000 秒。那么我们可以先将快门速度设置为 1/1000，然后将刻度拨动 5 档。

这个计算也相当简单。我们只需要计算刚刚计算出的 f5.6 倍数的以 2 为底的对数（尽管我们需要在四舍五入之前知道它的值）。然后将结果四舍五入到最接近的整数。在这个计算中，我们不需要担心有超过四个有效数字：结果很可能只有一到两位数字。

### A.13.4 FPU 优化

在汇编语言中，我们可以优化 FPU 代码，而这在高级语言（包括 C 语言）中是不可能的。

每当 C 函数需要计算浮点值时，它会将所有必要的变量和常量加载到 FPU 寄存器中。然后，它会完成必要的计算以得到正确的结果。优秀的 C 编译器能够对代码的这部分进行很好地优化。

它通过将结果留在 TOS 上来“返回”值。 但是，在返回之前，它会清理。 其在计算中使用的任何变量和常量现在从 FPU 中消失了。

它不能做到我们刚才所做的那样：我们计算了 f-数的平方，并将其保留在堆栈上，以便另一个函数稍后使用。

我们知道我们稍后会需要那个值。 我们还知道我们的堆栈（仅有 8 个数字的空间）有足够的空间将其存储在那里。

C 编译器无法知道栈上的值在不久的将来会再次被需要。

当然，C 程序员可能知道。但他唯一的补救措施就是将该值存储在内存变量中。

这意味着，首先，该值将从 FPU 内部使用的 80 位精度更改为 C double（64 位）甚至 single（32 位）。

这也意味着该值必须从 TOS 移动到内存，然后再次移动。遗憾的是，所有 FPU 操作中，访问计算机内存的操作最慢。

因此，每当在汇编语言中编程 FPU 时，请寻找在 FPU 堆栈上保留中间结果的方法。

我们甚至可以进一步发展这个想法！在我们的程序中，我们使用一个常量（我们命名为 PC ）。

我们计算的针孔直径数量无关紧要：1，10，20，1000，我们总是使用相同的常数。因此，我们可以通过始终将常数保留在堆栈上来优化程序。

在我们的程序早期，我们计算上述常数的值。我们需要对常数中的每个数字将输入除以 10 。

乘法比除法快得多。所以，在我们程序的开始，我们将 10 除以 1 以得到 0.1 ，然后将其保留在堆栈上：与其对每个数字将输入除以 10 ，我们将其乘以 0.1 。

顺便说一下，我们不直接输入 0.1 ，尽管我们可以。我们有一个理由：虽然 0.1 只有一位小数，但我们不知道需要多少个二进制位。因此，我们让 FPU 以自己的高精度计算它的二进制值。

我们在使用其他常数：我们将针孔直径乘以 1000 ，将其从毫米转换为微米。当我们将数字四舍五入到四个有效数字时，我们将其与 10000 进行比较。因此，我们在堆栈上保留 1000 和 10000 。当将数字四舍五入到四位数时，当然我们会重复使用 0.1 。

最后一点，我们在堆栈上保留 -5 。我们需要它来缩放光圈数的平方，而不是将其除以 32 。我们最后加载这个常数并非巧合。这使得它成为堆栈顶部，当堆栈上只有常数时。因此，当光圈数的平方正在被缩放时， -5 位于 st(1) ，正好是 fscale 所期望的位置。

通常，我们会从头开始创建某些常量，而不是从内存中加载它们。这就是我们正在用 -5 做的事情：

```
	fld1			; TOS =  1
	fadd	st0, st0	; TOS =  2
	fadd	st0, st0	; TOS =  4
	fld1			; TOS =  1
	faddp	st1, st0	; TOS =  5
	fchs			; TOS = -5
```

我们可以将所有这些优化归纳为一个规则：将重复值保留在堆栈上！

|  | PostScript®是一种基于堆栈的编程语言。关于 PostScript®的书籍比关于 FPU 汇编语言的书籍多得多：精通 PostScript®将帮助您精通 FPU。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------- |

### A.13.5. pinhole-代码

```
;;;;;;; pinhole.asm ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Find various parameters of a pinhole camera construction and use
;
; Started:	 9-Jun-2001
; Updated:	10-Jun-2001
;
; Copyright (c) 2001 G. Adam Stanislav
; All rights reserved.
;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

%include	'system.inc'

%define	BUFSIZE	2048

section	.data
align 4
ten	dd	10
thousand	dd	1000
tthou	dd	10000
fd.in	dd	stdin
fd.out	dd	stdout
envar	db	'PINHOLE='	; Exactly 8 bytes, or 2 dwords long
pinhole	db	'04,', 		; Bender's constant (0.04)
connors	db	'037', 0Ah	; Connors' constant
usg	db	'Usage: pinhole [-b] [-c] [-e] [-p <value>] [-o <outfile>] [-i <infile>]', 0Ah
usglen	equ	$-usg
iemsg	db	"pinhole: Can't open input file", 0Ah
iemlen	equ	$-iemsg
oemsg	db	"pinhole: Can't create output file", 0Ah
oemlen	equ	$-oemsg
pinmsg	db	"pinhole: The PINHOLE constant must not be 0", 0Ah
pinlen	equ	$-pinmsg
toobig	db	"pinhole: The PINHOLE constant may not exceed 18 decimal places", 0Ah
biglen	equ	$-toobig
huhmsg	db	9, '???'
separ	db	9, '???'
sep2	db	9, '???'
sep3	db	9, '???'
sep4	db	9, '???', 0Ah
huhlen	equ	$-huhmsg
header	db	'focal length in millimeters,pinhole diameter in microns,'
	db	'F-number,normalized F-number,F-5.6 multiplier,stops '
	db	'from F-5.6', 0Ah
headlen	equ	$-header

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE
dbuffer	resb	20		; decimal input buffer
bbuffer	resb	10		; BCD buffer

section	.text
align 4
huh:
	call	write
	push	dword huhlen
	push	dword huhmsg
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	ret

align 4
perr:
	push	dword pinlen
	push	dword pinmsg
	push	dword stderr
	sys.write
	push	dword 4		; return failure
	sys.exit

align 4
consttoobig:
	push	dword biglen
	push	dword toobig
	push	dword stderr
	sys.write
	push	dword 5		; return failure
	sys.exit

align 4
ierr:
	push	dword iemlen
	push	dword iemsg
	push	dword stderr
	sys.write
	push	dword 1		; return failure
	sys.exit

align 4
oerr:
	push	dword oemlen
	push	dword oemsg
	push	dword stderr
	sys.write
	push	dword 2
	sys.exit

align 4
usage:
	push	dword usglen
	push	dword usg
	push	dword stderr
	sys.write
	push	dword 3
	sys.exit

align 4
global	_start
_start:
	add	esp, byte 8	; discard argc and argv[0]
	sub	esi, esi

.arg:
	pop	ecx
	or	ecx, ecx
	je	near .getenv		; no more arguments

	; ECX contains the pointer to an argument
	cmp	byte [ecx], '-'
	jne	usage

	inc	ecx
	mov	ax, [ecx]
	inc	ecx

.o:
	cmp	al, 'o'
	jne	.i

	; Make sure we are not asked for the output file twice
	cmp	dword [fd.out], stdout
	jne	usage

	; Find the path to output file - it is either at [ECX+1],
	; i.e., -ofile --
	; or in the next argument,
	; i.e., -o file

	or	ah, ah
	jne	.openoutput
	pop	ecx
	jecxz	usage

.openoutput:
	push	dword 420	; file mode (644 octal)
	push	dword 0200h | 0400h | 01h
	; O_CREAT | O_TRUNC | O_WRONLY
	push	ecx
	sys.open
	jc	near oerr

	add	esp, byte 12
	mov	[fd.out], eax
	jmp	short .arg

.i:
	cmp	al, 'i'
	jne	.p

	; Make sure we are not asked twice
	cmp	dword [fd.in], stdin
	jne	near usage

	; Find the path to the input file
	or	ah, ah
	jne	.openinput
	pop	ecx
	or	ecx, ecx
	je near usage

.openinput:
	push	dword 0		; O_RDONLY
	push	ecx
	sys.open
	jc	near ierr		; open failed

	add	esp, byte 8
	mov	[fd.in], eax
	jmp	.arg

.p:
	cmp	al, 'p'
	jne	.c
	or	ah, ah
	jne	.pcheck

	pop	ecx
	or	ecx, ecx
	je	near usage

	mov	ah, [ecx]

.pcheck:
	cmp	ah, '0'
	jl	near usage
	cmp	ah, '9'
	ja	near usage
	mov	esi, ecx
	jmp	.arg

.c:
	cmp	al, 'c'
	jne	.b
	or	ah, ah
	jne	near usage
	mov	esi, connors
	jmp	.arg

.b:
	cmp	al, 'b'
	jne	.e
	or	ah, ah
	jne	near usage
	mov	esi, pinhole
	jmp	.arg

.e:
	cmp	al, 'e'
	jne	near usage
	or	ah, ah
	jne	near usage
	mov	al, ','
	mov	[huhmsg], al
	mov	[separ], al
	mov	[sep2], al
	mov	[sep3], al
	mov	[sep4], al
	jmp	.arg

align 4
.getenv:
	; If ESI = 0, we did not have a -p argument,
	; and need to check the environment for "PINHOLE="
	or	esi, esi
	jne	.init

	sub	ecx, ecx

.nextenv:
	pop	esi
	or	esi, esi
	je	.default	; no PINHOLE envar found

	; check if this envar starts with 'PINHOLE='
	mov	edi, envar
	mov	cl, 2		; 'PINHOLE=' is 2 dwords long
rep	cmpsd
	jne	.nextenv

	; Check if it is followed by a digit
	mov	al, [esi]
	cmp	al, '0'
	jl	.default
	cmp	al, '9'
	jbe	.init
	; fall through

align 4
.default:
	; We got here because we had no -p argument,
	; and did not find the PINHOLE envar.
	mov	esi, pinhole
	; fall through

align 4
.init:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	sub	edx, edx
	mov	edi, dbuffer+1
	mov	byte [dbuffer], '0'

	; Convert the pinhole constant to real
.constloop:
	lodsb
	cmp	al, '9'
	ja	.setconst
	cmp	al, '0'
	je	.processconst
	jb	.setconst

	inc	dl

.processconst:
	inc	cl
	cmp	cl, 18
	ja	near consttoobig
	stosb
	jmp	short .constloop

align 4
.setconst:
	or	dl, dl
	je	near perr

	finit
	fild	dword [tthou]

	fld1
	fild	dword [ten]
	fdivp	st1, st0

	fild	dword [thousand]
	mov	edi, obuffer

	mov	ebp, ecx
	call	bcdload

.constdiv:
	fmul	st0, st2
	loop	.constdiv

	fld1
	fadd	st0, st0
	fadd	st0, st0
	fld1
	faddp	st1, st0
	fchs

	; If we are creating a CSV file,
	; print header
	cmp	byte [separ], ','
	jne	.bigloop

	push	dword headlen
	push	dword header
	push	dword [fd.out]
	sys.write

.bigloop:
	call	getchar
	jc	near done

	; Skip to the end of the line if you got '#'
	cmp	al, '#'
	jne	.num
	call	skiptoeol
	jmp	short .bigloop

.num:
	; See if you got a number
	cmp	al, '0'
	jl	.bigloop
	cmp	al, '9'
	ja	.bigloop

	; Yes, we have a number
	sub	ebp, ebp
	sub	edx, edx

.number:
	cmp	al, '0'
	je	.number0
	mov	dl, 1

.number0:
	or	dl, dl		; Skip leading 0's
	je	.nextnumber
	push	eax
	call	putchar
	pop	eax
	inc	ebp
	cmp	ebp, 19
	jae	.nextnumber
	mov	[dbuffer+ebp], al

.nextnumber:
	call	getchar
	jc	.work
	cmp	al, '#'
	je	.ungetc
	cmp	al, '0'
	jl	.work
	cmp	al, '9'
	ja	.work
	jmp	short .number

.ungetc:
	dec	esi
	inc	ebx

.work:
	; Now, do all the work
	or	dl, dl
	je	near .work0

	cmp	ebp, 19
	jae	near .toobig

	call	bcdload

	; Calculate pinhole diameter

	fld	st0	; save it
	fsqrt
	fmul	st0, st3
	fld	st0
	fmul	st5
	sub	ebp, ebp

	; Round off to 4 significant digits
.diameter:
	fcom	st0, st7
	fstsw	ax
	sahf
	jb	.printdiameter
	fmul	st0, st6
	inc	ebp
	jmp	short .diameter

.printdiameter:
	call	printnumber	; pinhole diameter

	; Calculate F-number

	fdivp	st1, st0
	fld	st0

	sub	ebp, ebp

.fnumber:
	fcom	st0, st6
	fstsw	ax
	sahf
	jb	.printfnumber
	fmul	st0, st5
	inc	ebp
	jmp	short .fnumber

.printfnumber:
	call	printnumber	; F number

	; Calculate normalized F-number
	fmul	st0, st0
	fld1
	fld	st1
	fyl2x
	frndint
	fld1
	fscale
	fsqrt
	fstp	st1

	sub	ebp, ebp
	call	printnumber

	; Calculate time multiplier from F-5.6

	fscale
	fld	st0

	; Round off to 4 significant digits
.fmul:
	fcom	st0, st6
	fstsw	ax
	sahf

	jb	.printfmul
	inc	ebp
	fmul	st0, st5
	jmp	short .fmul

.printfmul:
	call	printnumber	; F multiplier

	; Calculate F-stops from 5.6

	fld1
	fxch	st1
	fyl2x

	sub	ebp, ebp
	call	printnumber

	mov	al, 0Ah
	call	putchar
	jmp	.bigloop

.work0:
	mov	al, '0'
	call	putchar

align 4
.toobig:
	call	huh
	jmp	.bigloop

align 4
done:
	call	write		; flush output buffer

	; close files
	push	dword [fd.in]
	sys.close

	push	dword [fd.out]
	sys.close

	finit

	; return success
	push	dword 0
	sys.exit

align 4
skiptoeol:
	; Keep reading until you come to cr, lf, or eof
	call	getchar
	jc	done
	cmp	al, 0Ah
	jne	.cr
	ret

.cr:
	cmp	al, 0Dh
	jne	skiptoeol
	ret

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	clc
	ret

read:
	jecxz	.read
	call	write

.read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword [fd.in]
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.empty
	sub	eax, eax
	ret

align 4
.empty:
	add	esp, byte 4
	stc
	ret

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	jecxz	.ret	; nothing to write
	sub	edi, ecx	; start of buffer
	push	ecx
	push	edi
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; buffer is empty now
.ret:
	ret

align 4
bcdload:
	; EBP contains the number of chars in dbuffer
	push	ecx
	push	esi
	push	edi

	lea	ecx, [ebp+1]
	lea	esi, [dbuffer+ebp-1]
	shr	ecx, 1

	std

	mov	edi, bbuffer
	sub	eax, eax
	mov	[edi], eax
	mov	[edi+4], eax
	mov	[edi+2], ax

.loop:
	lodsw
	sub	ax, 3030h
	shl	al, 4
	or	al, ah
	mov	[edi], al
	inc	edi
	loop	.loop

	fbld	[bbuffer]

	cld
	pop	edi
	pop	esi
	pop	ecx
	sub	eax, eax
	ret

align 4
printnumber:
	push	ebp
	mov	al, [separ]
	call	putchar

	; Print the integer at the TOS
	mov	ebp, bbuffer+9
	fbstp	[bbuffer]

	; Check the sign
	mov	al, [ebp]
	dec	ebp
	or	al, al
	jns	.leading

	; We got a negative number (should never happen)
	mov	al, '-'
	call	putchar

.leading:
	; Skip leading zeros
	mov	al, [ebp]
	dec	ebp
	or	al, al
	jne	.first
	cmp	ebp, bbuffer
	jae	.leading

	; We are here because the result was 0.
	; Print '0' and return
	mov	al, '0'
	jmp	putchar

.first:
	; We have found the first non-zero.
	; But it is still packed
	test	al, 0F0h
	jz	.second
	push	eax
	shr	al, 4
	add	al, '0'
	call	putchar
	pop	eax
	and	al, 0Fh

.second:
	add	al, '0'
	call	putchar

.next:
	cmp	ebp, bbuffer
	jb	.done

	mov	al, [ebp]
	push	eax
	shr	al, 4
	add	al, '0'
	call	putchar
	pop	eax
	and	al, 0Fh
	add	al, '0'
	call	putchar

	dec	ebp
	jmp	short .next

.done:
	pop	ebp
	or	ebp, ebp
	je	.ret

.zeros:
	mov	al, '0'
	call	putchar
	dec	ebp
	jne	.zeros

.ret:
	ret
```

代码遵循与我们之前看到的所有其他过滤器相同的格式，只有一个细微的例外：

> 我们不再假设输入结束意味着事情的结束，这在面向字符的过滤器中我们认为是理所当然的。
>
> 此过滤器不处理字符。它处理一种语言（尽管非常简单，仅由数字组成）。
>
> 当我们没有更多输入时，可能意味着两种情况之一：
>
> * 我们已经完成并可以退出。这与以前一样。
> * 我们读取的最后一个字符是一个数字。我们已将它存储在我们的 ASCII 转浮点转换缓冲区的末尾。现在我们需要将该缓冲区的内容转换为数字，并写入我们输出的最后一行。
>
> 由于这个原因，我们已经修改了我们的 getchar 和我们的 read 例程，每当我们从输入中获取另一个字符时返回 carry flag 清除，或者每当没有更多输入时设置 carry flag 。
>
> 当然，我们仍然在使用汇编语言魔法来实现这一点！好好看看 getchar 。它总是返回与 carry flag 清除。
>
> 然而，我们的主要代码依赖于 carry flag 来告诉它何时退出-而它起作用。
>
> 魔法就在于 read 。每当它从系统接收到更多输入时，它只是返回到 getchar ，后者从输入缓冲区中获取一个字符，清除 carry flag 并返回。
>
> 但是当 read 不再从系统接收到更多输入时，它根本不返回到 getchar 。相反， add esp, byte 4 操作码将 4 加到 ESP 上，设置 carry flag ，然后返回。
>
> 所以，它返回到哪里呢？每当一个程序使用 call 操作码时，微处理器 push 返回地址，即将其存储在堆栈顶部（不是 FPU 堆栈，而是内存中的系统堆栈）。当一个程序使用 ret 操作码时，微处理器 pop 从堆栈中取回返回值，并跳转到存储在那里的地址。
>
> 但是，由于我们将 4 添加到 ESP （即堆栈指针寄存器），实际上给微处理器带来了轻微的健忘症：它不再记得是 getchar 使 call 了 read 。
>
> 而且，由于 getchar 在 call 之前从未 push 过任何东西 read ，堆栈顶部现在包含了返回地址，指向 call 或 getchar 的任何内容。就那个调用者而言，他 call 了 getchar ，并带着 ret 返回！

除此之外， bcdload 例程陷入了大端和小端之间的利利普特冲突中。

它正在将数字的文本表示转换为该数字：文本以大端顺序存储，但打包的十进制是小端的。

为了解决冲突，我们在早期使用 std 操作码。我们稍后用 cld 取消它：在 std 激活时，我们不要做任何可能依赖方向标志默认设置的事情是非常重要的。

这段代码中的其他内容应该很清楚，只要你已经阅读完前面的整章。

这是编程需要大量思考而只需少量编码的经典例子。一旦我们仔细考虑了每一个细节，代码几乎可以自己写出来。

### A.13.6. 使用针孔

因为我们决定让程序忽略除数字之外的任何输入（甚至包括在注释中的数字），我们实际上可以执行文本查询。我们不必这样做，但我们可以。

依我拙见，形成文本查询，而不必遵循非常严格的语法，会让软件更加用户友好。

假设我们想要制作一个用于使用 4x5 英寸胶片的小孔相机。该胶片的标准焦距约为 150 毫米。我们想要微调我们的焦距，以使小孔直径尽可能为一个圆整数。我们还假设我们对相机非常熟悉，但对计算机有些畏惧。与其只需输入一堆数字，我们想问几个问题。

我们的会话可能是这样的：

```
% pinhole

Computer,

What size pinhole do I need for the focal length of 150?
150	490	306	362	2930	12
Hmmm... How about 160?
160	506	316	362	3125	12
Let's make it 155, please.
155	498	311	362	3027	12
Ah, let's try 157...
157	501	313	362	3066	12
156?
156	500	312	362	3047	12
That's it! Perfect! Thank you very much!
^D
```

我们发现，对于焦距为 150 的情况，我们的针孔直径应为 490 微米，或 0.49 毫米，如果我们选择几乎相同的焦距为 156 毫米，我们可以使用直径正好为半毫米的针孔。

### A.13.7. 脚本编写

因为我们选择了 # 字符来表示评论的开头，所以我们可以将我们的小孔软件视为一种脚本语言。

你可能见过以shell开头的脚本。

```
#! /bin/sh
```

 …或…

```
#!/bin/sh
```

因为 #! 后的空格是可选的。

每当 UNIX®被要求运行以 #! 开头的可执行文件时，它会假定该文件是一个脚本。它会将该命令添加到脚本第一行的其余部分，并尝试执行该命令。

现在假设我们已经在/usr/local/bin/中安装了 pinhole，我们现在可以编写一个脚本，以计算适用于 120 胶片常用的各种焦距的不同孔径。

脚本可能看起来像这样：

```
#! /usr/local/bin/pinhole -b -i
# Find the best pinhole diameter
# for the 120 film

### Standard
80

### Wide angle
30, 40, 50, 60, 70

### Telephoto
100, 120, 140
```

因为 120 是一部中等大小的电影，我们可以将此文件命名为 medium。

我们可以将其权限设置为执行，并像运行程序一样运行它：

```
% chmod 755 medium
% ./medium
```

UNIX® 将解释最后一个命令为：

```
% /usr/local/bin/pinhole -b -i ./medium
```

它将运行该命令并显示：

```
80	358	224	256	1562	11
30	219	137	128	586	9
40	253	158	181	781	10
50	283	177	181	977	10
60	310	194	181	1172	10
70	335	209	181	1367	10
100	400	250	256	1953	11
120	438	274	256	2344	11
140	473	296	256	2734	11
```

现在，让我们输入：

```
% ./medium -c
```

UNIX® 将把它视为：

```
% /usr/local/bin/pinhole -b -i ./medium -c
```

这给它两个冲突的选项： -b 和 -c （使用贝德尔常数和使用康纳斯常数）。我们已编程，所以后面的选项将覆盖前面的选项-我们的程序将使用康纳斯常数来计算所有内容：

```
80	331	242	256	1826	11
30	203	148	128	685	9
40	234	171	181	913	10
50	262	191	181	1141	10
60	287	209	181	1370	10
70	310	226	256	1598	11
100	370	270	256	2283	11
120	405	296	256	2739	11
140	438	320	362	3196	12
```

我们决定毕竟使用贝德尔常数。我们想将其值保存为逗号分隔的文件：

```
% ./medium -b -e > bender
% cat bender
focal length in millimeters,pinhole diameter in microns,F-number,normalized F-number,F-5.6 multiplier,stops from F-5.6
80,358,224,256,1562,11
30,219,137,128,586,9
40,253,158,181,781,10
50,283,177,181,977,10
60,310,194,181,1172,10
70,335,209,181,1367,10
100,400,250,256,1953,11
120,438,274,256,2344,11
140,473,296,256,2734,11
%
```

## A.14. 注意事项

在 MS-DOS® 和 Windows® 下“成长”的汇编语言程序员往往倾向于走捷径。阅读键盘扫描码并直接写入视频内存是两个经典的例子，在 MS-DOS® 下这些做法并不受到指责，反而被认为是正确的做法。

原因是什么？PC BIOS 和 MS-DOS® 在执行这些操作时都特别慢。

您可能会被诱惑在 FreeBSD 环境中继续类似的做法。例如，我曾看到一个网站，解释如何访问流行的 FreeBSD 克隆版本上的键盘扫描码。

这在 FreeBSD 环境中通常是一个非常糟糕的主意！让我解释为什么。

### A.14.1. FreeBSD 受到保护。

首先，这可能根本不可能。UNIX®在受保护模式下运行。只有内核和设备驱动程序被允许直接访问硬件。也许某个特定的 UNIX®克隆版允许您读取键盘的扫描码，但现实情况是真正的 UNIX®操作系统可能不会允许。而且，即使某个版本可能让您这样做，下一个版本可能不会，因此您精心编写的软件可能会一夜之间变成一只恐龙。

### A.14.2. UNIX®是一个抽象

但是有一个更重要的原因不要尝试直接访问硬件（当然，除非您正在编写设备驱动程序），即使是在让您这样做的 UNIX®类似系统中。

* UNIX® 是一个抽象！*

在设计哲学上，MS-DOS® 和 UNIX® 之间存在重大差异。MS-DOS® 被设计为单用户系统。它在连接键盘和视频屏幕的计算机上运行。用户输入几乎肯定来自该键盘。您程序的输出几乎总是显示在那个屏幕上。

在 UNIX® 下，这绝对不是保证。UNIX® 用户经常会使用管道和重定向程序的输入和输出：

```
% program1 | program2 | program3 > file1
```

如果您已经编写了程序 2，则您的输入不来自键盘，而来自程序 1 的输出。同样，您的输出不会显示在屏幕上，而会成为程序 3 的输入，该程序的输出又会写入 file1。

但还有更多！即使您确保您的输入来自终端，您的输出传送到终端，也不能保证该终端是 PC：它的视频内存可能不在您期望的位置，键盘也可能不会产生 PC 风格的扫描码。它可能是苹果电脑®，或任何其他计算机。

现在您可能会摇头：我的软件是使用 PC 汇编语言编写的，它怎么可能在苹果电脑®上运行？但我并没有说您的软件会在苹果电脑®上运行，只是说它的终端可能是苹果电脑®。

在 UNIX® 下，终端不必直接连接到运行您软件的计算机，它甚至可以在另一个大陆，又或者在另一个行星上。很可能一名澳大利亚的 Macintosh® 用户通过 telnet 连接到北美（或其他任何地方）的 UNIX® 系统。软件在一个计算机上运行，而终端在另一台计算机上：如果您尝试读取扫描码，您将得到错误的输入！

同样适用于任何其他硬件：您正在阅读的文件可能在您无法直接访问的磁盘上。您正在从一个太空船上连接的相机中读取图像，通过卫星与您连接。

这就是为什么在 UNIX® 下您绝对不应该对于数据从何处来源和到何处去做出任何假设。永远让系统处理对硬件的物理访问。

|  | 这些是警告，而不是绝对规则。例外是可能的。例如，如果文本编辑器确定正在运行在本地机器上，它可能希望直接读取扫描码以获得更好的控制。我提到这些警告并不是要告诉你该做什么或不该做什么，只是让你意识到如果你刚从 MS-DOS® 转到 UNIX®，会遇到一些潜在的问题。当然，有创造力的人经常打破规则，只要他们知道他们在打破规则以及为什么，那就没问题。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## A.15. 致谢

没有来自 FreeBSD 技术讨论邮件列表中许多经验丰富的 FreeBSD 程序员的帮助，这个教程将是不可能的。他们中的许多人耐心地回答了我的问题，并在我尝试探索 UNIX® 系统编程的内部工作以及 FreeBSD 特别是正确的方向。

Thomas M. Sommers 为我开了门。他的《在 FreeBSD 汇编语言中如何编写“Hello, world”》网页是我第一次接触在 FreeBSD 下汇编语言编程示例。

Jake Burkholder 一直保持着门敞开，乐意回答我所有的问题，并提供给我示例汇编语言源代码。

版权所有 © 2000-2001 G. Adam Stanislav。保留所有权利。
