# 第 11 章 x86 汇编语言程序设计

*本章由 G. Adam Stanislav 编写 \[[adam@redprince.net](mailto:adam@redprince.net)]。*

## A.1. 概述

在 UNIX® 下进行汇编语言编程的文献资料非常有限。通常认为没有人会使用汇编语言，因为各种 UNIX® 系统运行在不同的微处理器上，所以一切应该使用 C 语言编写以保证可移植性。

实际上，C 语言的可移植性实际上是一个神话。即使是 C 程序在从一个 UNIX® 移植到另一个 UNIX® 时，也需要修改，无论它们运行在哪种处理器上。通常，这样的程序充满了依赖于其编译系统的条件语句。

即使我们相信所有 UNIX® 软件都应该使用 C 或其他高级语言编写，我们仍然需要汇编语言程序员：谁来编写访问内核的 C 库部分呢？

在本章中，我将尝试向您展示如何使用汇编语言编写 UNIX® 程序，特别是在 FreeBSD 下。

本章并不解释汇编语言的基础知识。关于这一点，有足够的资源（想要学习完整的在线汇编语言课程，请参见 Randall Hyde 的 [《64 位汇编语言的编程艺术》](http://webster.cs.ucr.edu/)；如果您更喜欢打印版书籍，请查看 Jeff Duntemann 的《汇编语言 基于 Linux 环境》（ISBN: 0471375233））。然而，完成本章后，任何汇编语言程序员都将能够快速高效地为 FreeBSD 编写程序。

版权® 2000-2001 G. Adam Stanislav。保留所有权利。

## A.2. 工具

### A.2.1. 汇编器

进行汇编语言编程最重要的工具是汇编器，它是将汇编语言代码转换为机器语言的软件。

FreeBSD 提供了三种非常不同的汇编器。`llvm-as(1)`（包含在 [devel/llvm](https://cgit.freebsd.org/ports/tree/devel/llvm/) 中）和 `as(1)`（包含在 [devel/binutils](https://cgit.freebsd.org/ports/tree/devel/binutils/) 中）使用传统的 UNIX® 汇编语言语法。

另一方面，`nasm(1)`（通过 [devel/nasm](https://cgit.freebsd.org/ports/tree/devel/nasm/) 安装）使用 Intel 语法。它的主要优点是可以为许多操作系统生成汇编代码。

本章使用 nasm 语法，因为大多数从其他操作系统转到 FreeBSD 的汇编语言程序员会觉得这种语法更易理解。而且，坦率地说，这也是我习惯的语法。

### A.2.2. 链接器

汇编器的输出文件与任何编译器的输出文件一样，需要通过链接器来生成可执行文件。

FreeBSD 提供了标准的 `ld(1)` 链接器。它可以与任何汇编器生成的代码一起使用。

## A.3. 系统调用

### A.3.1. 默认调用约定

默认情况下，FreeBSD 内核使用 C 调用约定。此外，虽然内核是通过 `int 80h` 进行访问的，但程序会调用一个发出 `int 80h` 的函数，而不是直接发出 `int 80h`。

这种约定非常方便，并且比 MS-DOS® 使用的 Microsoft® 调用约定更为优越。为什么？因为 UNIX® 约定允许任何用任何语言编写的程序访问内核。

汇编语言程序也可以这样做。例如，我们可以打开一个文件：

```asm
kernel:
	int	80h	; 调用内核
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

这是非常简洁和可移植的编码方式。如果您需要将代码移植到使用不同中断或不同传递参数方式的 UNIX® 系统，只需要修改内核程序即可。

但汇编语言程序员通常喜欢优化性能。上面的例子需要 `call/ret` 组合。我们可以通过 `push` 一个额外的 dword 来消除它：

```asm
open:
	push	dword mode
	push	dword flags
	push	dword path
	mov	eax, 5
	push	eax		; 或任何其他 dword
	int	80h
	add	esp, byte 16
```

我们将 `5` 放入 `EAX` 寄存器中，以标识内核函数，此处为 `open`。

### A.3.2. 替代调用约定

FreeBSD 是一款非常灵活的系统。它提供了其他访问内核的方式。然而，系统必须安装 Linux 模拟才能正常工作。

Linux 是一款类 UNIX® 系统。然而，它的内核使用与 MS-DOS® 相同的系统调用约定，即通过寄存器传递参数。与 UNIX® 约定类似，函数编号放在 `EAX` 中，参数则不通过堆栈传递，而是放在 `EBX`、`ECX`、`EDX`、`ESI`、`EDI` 和 `EBP` 中：

```asm
open:
	mov	eax, 5
	mov	ebx, path
	mov	ecx, flags
	mov	edx, mode
	int	80h
```

这种约定相对于 UNIX® 方式有一个很大的缺点，至少对汇编语言编程来说：每次进行内核调用时，您必须 `push` 寄存器，然后稍后再 `pop` 它们。这使得代码更庞大且运行较慢。尽管如此，FreeBSD 仍然提供了选择权。

如果您选择了 Linux 调用约定，您必须告诉系统。程序汇编并链接后，您需要为可执行文件打上品牌：

```sh
% brandelf -t Linux filename
```

### A.3.3. 您应该使用哪种约定？

如果您专门为 FreeBSD 编程，您应该始终使用 UNIX® 约定：它更快，您可以将全局变量存储在寄存器中，不需要对可执行文件进行品牌化，也不需要在目标系统上安装 Linux 模拟包。

如果您希望创建可以在 Linux 上运行的可移植代码，您可能仍然希望为 FreeBSD 用户提供尽可能高效的代码。我将在解释基本内容之后，向您展示如何实现这一点。

### A.3.4. 调用号

要告诉内核您正在调用哪个系统服务，请将其编号放入 `EAX` 中。当然，您需要知道这个编号是什么。

#### A.3.4.1. **syscalls** 文件

这些编号列在 **syscalls** 文件中。使用 `locate syscalls` 可以找到这个文件的多个不同格式，所有格式都从 **syscalls.master** 自动生成。

您可以在 **/usr/src/sys/kern/syscalls.master** 中找到默认 UNIX® 调用约定的主文件。如果您需要使用 Linux 模拟模式中实现的另一种约定，请阅读 **/usr/src/sys/i386/linux/syscalls.master**。

>**注意**
>
>  不仅 FreeBSD 和 Linux 使用不同的调用约定，它们有时对相同的功能使用不同的编号。

**syscalls.master** 描述了如何进行调用：

```sh
0	STD	NOHIDE	{ int nosys(void); } syscall nosys_args int
1	STD	NOHIDE	{ void exit(int rval); } exit rexit_args void
2	STD	POSIX	{ int fork(void); }
3	STD	POSIX	{ ssize_t read(int fd, void *buf, size_t nbyte); }
4	STD	POSIX	{ ssize_t write(int fd, const void *buf, size_t nbyte); }
5	STD	POSIX	{ int open(char *path, int flags, int mode); }
6	STD	POSIX	{ int close(int fd); }
etc...
```

最左边的列告诉我们将哪个数字放入 `EAX`。

最右边的列告诉我们需要 `push` 什么参数。它们是从右到左依次 `push` 的。

例如，要 `open` 一个文件，我们需要首先 `push` `mode`，然后是 `flags`，最后是存储 `path` 地址的变量。

## A.4. 返回值

如果系统调用没有返回某种类型的值，大多数情况下是没有用的：例如打开文件的文件描述符、读取到缓冲区的字节数、系统时间等。

此外，系统还需要告知我们是否发生了错误：例如文件不存在、系统资源耗尽、传递了无效参数等。

### A.4.1. 手册页

在 UNIX® 系统下，传统的查看各种系统调用信息的地方是手册页。FreeBSD 在第 2 节中描述其系统调用，有时在第 3 节中。

例如，[open(2)](https://man.freebsd.org/cgi/man.cgi?query=open&sektion=2&format=html) 说：

如果成功，`open()` 返回一个非负整数，称为文件描述符。如果失败，返回 `-1`，并设置 `errno` 来指示错误。

对于刚接触 UNIX® 和 FreeBSD 的汇编语言程序员来说，立刻会产生一个令人困惑的问题：`errno` 到底在哪里，如何访问它？

>**注意**
>
> 手册页中提供的信息适用于 C 程序。汇编语言程序员需要额外的信息。

### A.4.2. 返回值在哪里？

不幸的是，这取决于……对于大多数系统调用，返回值在 `EAX` 中，但并非所有系统调用都如此。一个好的经验法则是，当首次处理一个系统调用时，先检查返回值是否在 `EAX` 中。如果不在那儿，您需要进一步的研究。

>**注意**
>
> 我知道有一个系统调用将值返回在 `EDX` 中：`SYS_fork`。其他我处理过的系统调用都使用 `EAX`。但我还没有处理所有系统调用。

>**技巧**
>
>  如果您在这里找不到答案或其他地方没有答案，可以研究 libc 源代码，看看它是如何与内核交互的。 

### A.4.3. `errno` 在哪里？

实际上，`errno` 根本不存在……

`errno` 是 C 语言的一部分，而不是 UNIX® 内核的一部分。当直接访问内核服务时，错误代码会返回到 `EAX` 中，这与通常存放返回值的寄存器相同。

这完全合理。如果没有错误，就没有错误代码。如果发生了错误，就没有返回值。一个寄存器可以同时存放这两者。

### A.4.4. 如何判断是否发生了错误

在使用标准 FreeBSD 调用约定时，成功时 `carry flag` 会被清除，失败时会被设置。

在使用 Linux 模拟模式时，`EAX` 中的有符号值在成功时为非负值，并包含返回值。如果发生错误，值为负数，即 `-errno`。

## A.5. 创建可移植代码

可移植性通常不是汇编语言的强项。然而，编写适用于不同平台的汇编语言程序是可能的，尤其是使用 nasm。我已经编写了可以在多个操作系统（如 Windows® 和 FreeBSD）上汇编的汇编语言库。

当您希望代码能够在两个不同平台上运行时，这尤其可行，这两个平台尽管有所不同，但基于类似的架构。

例如，FreeBSD 是 UNIX®，Linux 是 UNIX® 类似的操作系统。我只提到了它们之间的三个区别（从汇编语言程序员的角度看）：调用约定、函数号和返回值的方式。

### A.5.1. 处理函数号

在许多情况下，函数号是相同的。然而，即使它们不相同，问题也很容易处理：不要在代码中直接使用数字，而是使用常量，根据目标架构不同进行定义：

```c
%ifdef	LINUX
%define	SYS_execve	11
%else
%define	SYS_execve	59
%endif
```

### A.5.2. 处理约定

调用约定和返回值（`errno` 问题）都可以通过宏来解决：

```c
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

上述解决方案可以处理大部分在 FreeBSD 和 Linux 之间编写可移植代码的情况。然而，对于某些内核服务，差异更为深入。

在这种情况下，您需要为那些特定的系统调用编写两个不同的处理程序，并使用条件汇编。幸运的是，您的大部分代码做的工作与调用内核无关，因此通常您只需要在代码中添加几个这样的条件部分。

### A.5.4. 使用库

您可以通过编写一个系统调用库，完全避免主代码中的可移植性问题。为 FreeBSD 编写一个单独的库，为 Linux 编写另一个不同的库，甚至为更多操作系统编写其他库。

在您的库中，为每个系统调用编写一个单独的函数（或者，如果您喜欢传统的汇编语言术语，可以称之为过程）。使用 C 调用约定来传递参数，但仍然使用 `EAX` 来传递调用号。在这种情况下，您的 FreeBSD 库可以非常简单，因为许多看似不同的函数实际上只是指向相同代码的标签：

```c
sys.open:
sys.close:
[等等...]
	int	80h
	ret
```

您的 Linux 库将需要更多不同的函数。但是即便如此，您也可以根据相同的参数数量来分组系统调用：

```c
sys.exit:
sys.close:
[等等... 单参数函数]
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

最初，使用库的方法可能看起来不方便，因为它需要您生成一个代码依赖的单独文件。但它有许多优点：首先，您只需要编写一次，并且可以将其用于所有程序。您甚至可以让其他汇编语言程序员使用它，或者使用由他人编写的库。但或许库的最大优势是，您的代码可以通过简单编写一个新的库而无需更改代码，即可移植到其他系统，甚至其他程序员也能完成此操作。

如果您不喜欢使用库的想法，您至少可以将所有系统调用放在一个单独的汇编语言文件中，并将其与主程序链接。在这种情况下，移植者只需要创建一个新的目标文件来与主程序链接。

### A.5.5. 使用包含文件

如果您将软件作为（或与）源代码一起发布，您可以使用宏并将它们放在一个单独的文件中，然后在代码中包含这个文件。

您的软件移植者只需编写一个新的包含文件。这样就不需要库或外部目标文件，但您的代码依然可以在无需编辑代码的情况下实现可移植性。

>**注意**
>
>这是我们将在本章中使用的方法。我们将把包含文件命名为 **system.inc**，并在处理新的系统调用时不断添加内容。

我们可以通过声明标准的文件描述符来开始我们的 **system.inc**：

```c
%define	stdin	0
%define	stdout	1
%define	stderr	2
```

接下来，为每个系统调用创建一个符号名称：

```c
%define	SYS_nosys	0
%define	SYS_exit	1
%define	SYS_fork	2
%define	SYS_read	3
%define	SYS_write	4
; [等等...]
```

我们添加一个短小、非全局的过程，命名为长名称，这样我们就不会在代码中不小心重复使用它：

```asm
section	.text
align 4
access.the.bsd.kernel:
	int	80h
	ret
```

然后，我们创建一个宏，它接收一个参数，即系统调用编号：

```asm
%macro	system	1
	mov	eax, %1
	call	access.the.bsd.kernel
%endmacro
```

最后，我们为每个系统调用创建宏，这些宏不接受任何参数。

```asm
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

; [等等...]
```

接下来，输入并保存它为 **system.inc**。随着我们讨论更多的系统调用，内容还将继续添加到其中。


## A.6. 我们的第一个程序

现在，我们准备好编写第一个程序——必备的“Hello, World!”程序。

```asm
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

这段代码的功能如下：第一行包含了 **system.inc** 中的定义、宏和代码。

第3-5行是数据部分：第3行开始了数据段。第4行包含了字符串“Hello, World!”以及一个换行符 (`0Ah`)。第5行创建了一个常量，表示第4行字符串的字节长度。

第7-16行是代码部分。需要注意的是，FreeBSD 使用 *elf* 文件格式来处理其可执行文件，这要求每个程序都从标签 `_start` 开始（更准确地说，链接器期望这样做）。该标签必须是全局的。

第10-13行请求系统将 `hbytes` 字节的 `hello` 字符串写入 `stdout`。

第15-16行请求系统用返回值 `0` 结束程序。由于 `SYS_exit` 系统调用不会返回，因此代码在此结束。

>**注意**
>
> 如果你是从 MS-DOS® 汇编语言背景转到 UNIX®，你可能习惯了直接写入视频硬件。在 FreeBSD 或任何其他 UNIX® 系统中，你不必担心这个问题。对你来说，你是在写入一个名为 **stdout** 的文件。这个文件可以是视频屏幕、telnet 终端、实际文件，甚至是另一个程序的输入。至于它是什么，交给系统来处理。

### A.6.1. 汇编代码

将代码输入编辑器，并将其保存为 **hello.asm** 文件。你需要使用 nasm 来汇编它。

#### A.6.1.1. 安装 nasm

如果你没有安装 nasm，可以输入：

```sh
% su
Password:你的 root 密码
# cd /usr/ports/devel/nasm
# make install
# exit
%
```

如果你不想保留 nasm 源代码，可以输入 `make install clean`，而不是单纯的 `make install`。

无论哪种方式，FreeBSD 会自动从互联网上下载 nasm，进行编译，并安装到你的系统上。

>**注意**
>
> 如果你的系统不是 FreeBSD，你可以从其 [主页](https://sourceforge.net/projects/nasm) 获取 nasm。你仍然可以使用它来汇编 FreeBSD 的代码。


现在你可以汇编、链接并运行代码：

```sh
% nasm -f elf hello.asm
% ld -s -o hello hello.o
% ./hello
Hello, World!
%
```

## A.7. 编写 UNIX® 过滤器

一种常见的 UNIX® 应用程序类型是过滤器——它是一个从 **stdin** 读取数据，进行某种处理，然后将结果写入 **stdout** 的程序。

在本章中，我们将开发一个简单的过滤器，学习如何从 **stdin** 读取数据并写入 **stdout**。这个过滤器将把输入的每个字节转换为一个十六进制数，并在后面加上一个空格。

```asm
%include	'system.inc'

section	.data
hex	db	'0123456789ABCDEF'
buffer	db	0, 0, ' '

section	.text
global	_start
_start:
	; 从 stdin 读取一个字节
	push	dword 1
	push	dword buffer
	push	dword stdin
	sys.read
	add	esp, byte 12
	or	eax, eax
	je	.done

	; 转换为十六进制
	movzx	eax, byte [buffer]
	mov	edx, eax
	shr	dl, 4
	mov	dl, [hex+edx]
	mov	[buffer], dl
	and	al, 0Fh
	mov	al, [hex+eax]
	mov	[buffer+1], al

	; 打印出来
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

在数据部分，我们创建了一个名为 `hex` 的数组，包含了 16 个十六进制数字，按升序排列。数组后面是一个缓冲区，我们将用它来存储输入和输出。缓冲区的前两个字节最初设置为 `0`，用于存储两个十六进制数字（第一个字节同时用于读取输入）。第三个字节是一个空格。

代码部分包含了四个部分：读取字节、将其转换为十六进制、写入结果，以及最终退出程序。

为了读取字节，我们请求系统从 **stdin** 读取一个字节，并将其存储在 `buffer` 的第一个字节中。系统返回读取的字节数，存储在 `EAX` 中。当有数据时，它的值为 `1`，而当没有更多输入数据时，它的值为 `0`。因此，我们检查 `EAX` 的值。如果它为 `0`，则跳转到 `.done`，否则继续执行。

>**注意**
>
>为了简单起见，我们暂时忽略了错误条件。 

十六进制转换部分将字节从 `buffer` 读入 `EAX`（实际上只读 `AL`），同时将 `EAX` 的其余位清零。我们还将字节复制到 `EDX` 中，因为我们需要分别处理高四位（nibble）和低四位。转换结果存储在缓冲区的前两个字节中。

接下来，我们请求系统将缓冲区的三个字节（即两个十六进制数字和空格）写入 **stdout**。然后，我们跳转回程序的开始，处理下一个字节。

一旦没有更多输入数据，我们请求系统退出程序，返回值为 `0`，这是表示程序成功的传统值。

接下来，保存代码为 **hex.asm**，然后输入以下命令（`^D` 代表按住控制键并同时按 `D`）：

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A ^D %
```

>**注意**
>
>如果你是从 MS-DOS® 迁移到 UNIX®，你可能会好奇为什么每行以 `0A` 结尾，而不是 `0D 0A`。这是因为 UNIX® 不使用 CR/LF（回车/换行）约定，而是使用“新行”约定，该新行用十六进制 `0A` 表示。 

我们能改进这个程序吗？首先，它有点混乱，因为一旦我们转换了一行文本，输入就不再从行首开始了。我们可以修改它，在每个 `0A` 后打印一个新行，而不是空格：

```asm
%include	'system.inc'

section	.data
hex	db	'0123456789ABCDEF'
buffer	db	0, 0, ' '

section	.text
global	_start
_start:
	mov	cl, ' '

.loop:
	; 从 stdin 读取一个字节
	push	dword 1
	push	dword buffer
	push	dword stdin
	sys.read
	add	esp, byte 12
	or	eax, eax
	je	.done

	; 转换为十六进制
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

	; 打印出来
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

我们将空格存储在 `CL` 寄存器中。我们这样做是安全的，因为与 Microsoft® Windows® 不同，UNIX® 系统调用不会修改它们没有使用来返回值的寄存器。

这意味着我们只需设置一次 `CL` 寄存器。因此，我们添加了一个新的标签 `.loop`，并跳转到它以处理下一个字节，而不是跳转到 `_start`。我们还添加了 `.hex` 标签，这样我们就可以在 `buffer` 的第三个字节中放置一个空格或一个新行。

修改 **hex.asm** 后，再次执行：

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

这看起来好多了。但这个程序效率不高！我们对每个字节都进行了两次系统调用（一次读取，另一次写入输出）。

## A.8. 缓冲输入和输出

通过对输入和输出进行缓冲，我们可以提高代码的效率。我们创建一个输入缓冲区，一次读取一整段字节，然后逐个从缓冲区中获取这些字节。

我们还创建一个输出缓冲区。我们将输出存储在缓冲区中，直到它满了。这时，我们请求内核将缓冲区的内容写入 **stdout**。

程序在没有更多输入时结束。但我们仍然需要请求内核最后一次将输出缓冲区的内容写入 **stdout**，否则一些输出可能会被写入输出缓冲区，但永远不会被发送出去。不要忘记这一点，否则你会发现某些输出丢失了。

```asm
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
	; 从 stdin 读取一个字节
	call	getchar

	; 转换为十六进制
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
	call	write		; 刷新输出缓冲区
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
	sub	edi, ecx	; 缓冲区的开始
	push	ecx
	push	edi
	push	dword stdout
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; 缓冲区现在是空的
	ret
```

现在，我们的源代码中有了第三个部分，命名为 `.bss`。该部分不包含在可执行文件中，因此不能初始化。我们使用 `resb` 而不是 `db`，它仅为我们保留了请求的大小的未初始化内存。

我们利用了系统不会修改寄存器的特性：我们使用寄存器来存储本应作为全局变量存储在 `.data` 部分的值。这也是 UNIX® 在系统调用中通过栈传递参数的约定优于 Microsoft® 约定的原因：我们可以将寄存器保留给自己的使用。

我们使用 `EDI` 和 `ESI` 作为指向下一个要读取或写入字节的指针。我们使用 `EBX` 和 `ECX` 来保持两个缓冲区中字节的计数，以便知道何时将输出转储到系统中，或从系统中读取更多输入。

让我们看看现在它是如何工作的：

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
Here I come!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

不是你预期的结果吗？程序直到我们按下 `^D` 后才打印输出。这很容易修复，只需插入三行代码，每次我们将一行转换为 `0A` 时就写入输出。我已用 `>` 标记了三行（不要在你的 **hex.asm** 中复制 `>`）。

```asm
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

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

对于一个 644 字节的可执行文件，这还不错，对吧！

>**注意**
>
>这种缓冲输入/输出的方法仍然存在一个隐藏的危险。我将在稍后讨论并修复它，当我谈到 [缓冲的黑暗面](https://docs.freebsd.org/en/books/developers-handbook/x86/#x86-buffered-dark-side) 时。

### A.8.1. 如何将字符重新放回输入流

>**警告**
>
>这是一个相对高级的话题，主要对熟悉编译器理论的程序员感兴趣。如果你愿意，可以 [跳过到下一节](https://docs.freebsd.org/en/books/developers-handbook/x86/#x86-command-line)，稍后再阅读。 

虽然我们的示例程序并不需要这个功能，但更复杂的过滤器通常需要进行前瞻处理。换句话说，它们可能需要查看下一个字符是什么（甚至是几个字符）。如果下一个字符是某个特定值，它就是当前正在处理的标记的一部分，否则就不是。

例如，在解析输入流中的文本字符串时（例如，编写语言编译器时）：如果一个字符后面跟着另一个字符，或者是一个数字，它就是正在处理的标记的一部分。如果后面跟着空白字符或其他值，那么它就不属于当前的标记。

这就引出了一个有趣的问题：如何将下一个字符放回输入流中，以便稍后可以再次读取？

一种可能的解决方案是将其存储在一个字符变量中，然后设置一个标志。我们可以修改 `getchar`，使其检查标志，如果标志被设置，就从该变量中获取字节，而不是从输入缓冲区读取，并重置标志。但是，当然，这会导致程序变慢。

C 语言有一个 `ungetc()` 函数，正是为此目的设计的。那么，我们在代码中有没有什么快速实现的方法呢？我建议你先回头看看 `getchar` 程序，看看你能否在读下一段之前找到一个简洁快速的解决方案。然后再回来看看我自己的解决方案。

将字符重新放回输入流的关键在于我们最初是如何获取字符的：

首先，我们通过检查 `EBX` 的值来确认缓冲区是否为空。如果为零，我们就调用 `read` 程序。

如果确实有字符可用，我们使用 `lodsb`，然后减少 `EBX` 的值。`lodsb` 指令实际上等同于：

```asm
mov	al, [esi]
	inc	esi
```

我们提取的字节将保留在缓冲区中，直到下一次调用 `read`。我们不知道何时会发生，但我们知道直到下一次调用 `getchar` 时，它才会发生。因此，要“返回”上次读取的字节，我们只需减少 `ESI` 的值并增加 `EBX` 的值：

```asm
ungetc:
	dec	esi
	inc	ebx
	ret
```

但是，注意！如果我们的前瞻只检查一个字符，这样做是完全安全的。如果我们检查多个即将到来的字符并连续多次调用 `ungetc`，它通常能正常工作，但并非每次都能成功（而且很难调试）。为什么呢？

因为只要 `getchar` 不需要调用 `read`，所有提前读取的字节仍然保存在缓冲区中，我们的 `ungetc` 就可以顺利工作。但是一旦 `getchar` 调用了 `read`，缓冲区的内容就会发生变化。

我们可以始终依赖 `ungetc` 正确工作在最后一个通过 `getchar` 读取的字符上，但不能依赖它处理之前读取的字符。

如果你的程序需要读取多个字节，你至少有两种选择：

1. 如果可能，修改程序，使其只读取一个字节。这是最简单的解决方案。

2. 如果无法选择此方案，首先确定程序一次需要返回输入流的最大字符数。稍微增加这个值，确保它足够大，最好是 16 的倍数——这样它就可以很好地对齐。然后修改代码的 `.bss` 部分，在输入缓冲区之前创建一个小的“备用”缓冲区，例如：

```asm
section	.bss
	resb	16	; 或者是你计算出来的值
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE
```

你还需要修改 `ungetc`，将要重新放回的字节值传递给 `AL`：

```asm
ungetc:
	dec	esi
	inc	ebx
	mov	[esi], al
	ret
```

通过这种修改，你可以安全地调用 `ungetc` 多达 17 次（第一次调用仍然在缓冲区内，剩余的 16 次可以在缓冲区内或在“备用”缓冲区内）。

## A.9. 命令行参数

如果我们的 hex 程序能从命令行读取输入和输出文件的名称，那么它将变得更加有用，也就是说，它能处理命令行参数。但... 它们在哪里呢？

在 UNIX® 系统启动程序之前，它会将一些数据 `push` 到栈中，然后跳转到程序的 `_start` 标签。是的，我说的是跳转，而不是调用。这意味着这些数据可以通过读取 `[esp+offset]` 来访问，或者通过简单地 `pop` 它们来访问。

栈顶的值包含命令行参数的数量，通常称为 `argc`，即“参数计数”。

命令行参数紧随其后，所有 `argc` 个参数。通常这些被称为 `argv`，即“参数值”。也就是说，我们可以获取 `argv[0]`、`argv[1]`、`…`、`argv[argc-1]`。这些不是实际的参数，而是指向参数的指针，也就是实际参数的内存地址。参数本身是以 NUL 终止的字符字符串。

`argv` 列表后跟一个 NULL 指针，这只是一个 `0`。还有更多的内容，但目前为止，这些已经足够了。

>**注意**
>
>如果你来自 MS-DOS® 编程环境，主要的区别是每个参数都在一个独立的字符串中。第二个区别是对参数数量没有实际的限制。

掌握了这些知识后，我们几乎可以开始编写 **hex.asm** 的下一个版本了。不过，在此之前，我们需要向 **system.inc** 文件中添加几行内容：

首先，我们需要向系统调用号列表中添加两个新的条目：

```asm
%define	SYS_open	5
%define	SYS_close	6
```

接着，在文件末尾添加两个新的宏：

```asm
%macro	sys.open	0
	system	SYS_open
%endmacro

%macro	sys.close	0
	system	SYS_close
%endmacro
```

以下是我们修改后的源代码：

```asm
%include	'system.inc'

%define	BUFSIZE	2048	; 定义缓冲区大小为 2048 字节

section	.data
fd.in	dd	stdin	; 定义输入文件描述符（stdin）
fd.out	dd	stdout	; 定义输出文件描述符（stdout）
hex	db	'0123456789ABCDEF'	; 十六进制字符表

section .bss
ibuffer	resb	BUFSIZE	; 输入缓冲区
obuffer	resb	BUFSIZE	; 输出缓冲区

section	.text
align 4
err:
	push	dword 1		; 返回失败代码 1
	sys.exit			; 退出程序

align 4
global	_start
_start:
	add	esp, byte 8	; 丢弃 argc 和 argv[0]，即去掉命令行参数计数器和程序名

	pop	ecx
	jecxz	.init		; 如果没有更多的参数，跳到初始化部分

	; ECX 现在包含输入文件的路径
	push	dword 0		; O_RDONLY：以只读模式打开文件
	push	ecx			; 文件路径作为参数
	sys.open			; 系统调用打开文件
	jc	err				; 如果打开失败，跳到错误处理

	add	esp, byte 8	; 恢复堆栈
	mov	[fd.in], eax	; 保存输入文件描述符

	pop	ecx
	jecxz	.init		; 如果没有更多的参数，跳到初始化部分

	; ECX 现在包含输出文件的路径
	push	dword 420	; 文件权限（644 八进制）
	push	dword 0200h | 0400h | 01h	; O_CREAT | O_TRUNC | O_WRONLY
	; 创建输出文件，覆盖文件
	push	ecx			; 输出文件路径作为参数
	sys.open			; 系统调用打开文件
	jc	err				; 如果打开失败，跳到错误处理

	add	esp, byte 12	; 恢复堆栈
	mov	[fd.out], eax	; 保存输出文件描述符

.init:
	sub	eax, eax	; 清空寄存器
	sub	ebx, ebx	; 清空寄存器
	sub	ecx, ecx	; 清空寄存器
	mov	edi, obuffer	; 将输出缓冲区的地址移动到 EDI 寄存器

.loop:
	; 从输入文件或标准输入读取一个字节
	call	getchar

	; 将字节转换为十六进制格式
	mov	dl, al			; 保存原始字节到 dl
	shr	al, 4			; 将字节高四位移动到低四位
	mov	al, [hex+eax]	; 查找十六进制字符
	call	putchar		; 输出十六进制字符

	mov	al, dl			; 获取低四位
	and	al, 0Fh			; 清除高四位
	mov	al, [hex+eax]	; 查找低四位的十六进制字符
	call	putchar		; 输出低四位的十六进制字符

	mov	al, ' '			; 输出空格
	cmp	dl, 0Ah			; 如果是换行符
	jne	.put				; 如果不是换行符，跳到 put
	mov	al, dl			; 如果是换行符，输出换行符

.put:
	call	putchar		; 输出字符
	cmp	al, dl			; 比较字符是否已完全输出
	jne	.loop			; 如果未完成，继续循环
	call	write			; 写入缓冲区内容到输出
	jmp	short .loop		; 继续处理下一个字节

align 4
getchar:
	or	ebx, ebx			; 检查 EBX 是否为零
	jne	.fetch				; 如果非零，跳转到 fetch

	call	read				; 如果为零，调用 read 读取数据

.fetch:
	lodsb					; 加载字节并增加 ESI
	dec	ebx					; 减少 EBX（字节计数器）
	ret						; 返回

read:
	push	dword BUFSIZE		; 推入缓冲区大小
	mov	esi, ibuffer		; 设置输入缓冲区地址
	push	esi				; 参数：缓冲区地址
	push	dword [fd.in]		; 参数：输入文件描述符
	sys.read				; 系统调用读取文件
	add	esp, byte 12		; 恢复堆栈
	mov	ebx, eax			; 保存返回值（读取的字节数）
	or	eax, eax			; 检查返回值是否为零
	je	.done				; 如果没有读取到数据，跳到 done
	sub	eax, eax			; 重置 EAX
	ret						; 返回

align 4
.done:
	call	write				; 刷新输出缓冲区

	; 关闭文件
	push	dword [fd.in]		; 关闭输入文件描述符
	sys.close

	push	dword [fd.out]		; 关闭输出文件描述符
	sys.close

	; 返回成功
	push	dword 0			; 返回代码 0
	sys.exit				; 退出程序

align 4
putchar:
	stosb					; 将 AL 存储到输出缓冲区
	inc	ecx					; 增加缓冲区索引
	cmp	ecx, BUFSIZE		; 检查缓冲区是否已满
	je	write				; 如果满了，调用 write 写入
	ret						; 否则返回

align 4
write:
	sub	edi, ecx			; 计算缓冲区的起始地址
	push	ecx					; 推入缓冲区大小
	push	edi					; 推入缓冲区地址
	push	dword [fd.out]		; 推入输出文件描述符
	sys.write				; 系统调用写入文件
	add	esp, byte 12		; 恢复堆栈
	sub	eax, eax			; 清除 EAX
	sub	ecx, ecx			; 清空缓冲区
	ret						; 返回
```

在我们的 `.data` 部分，现在有了两个新变量，`fd.in` 和 `fd.out`。我们在这里存储输入和输出的文件描述符。

在 `.text` 部分，我们将对 `stdin` 和 `stdout` 的引用替换为 `[fd.in]` 和 `[fd.out]`。

`.text` 部分现在以一个简单的错误处理程序开始，它仅仅是退出程序并返回值 `1`。这个错误处理程序位于 `_start` 之前，因此我们可以很接近错误发生的地方。

自然，程序的执行仍然从 `_start` 开始。首先，我们从栈中移除 `argc` 和 `argv[0]`：它们对我们来说不重要（在这个程序中是这样）。

我们将 `argv[1]` 弹出到 `ECX` 寄存器。这个寄存器特别适合存储指针，因为我们可以通过 `jecxz` 来处理 NULL 指针。如果 `argv[1]` 不是 NULL，我们尝试打开第一个参数指定的文件。否则，我们继续像之前一样操作：从 `stdin` 读取，写入 `stdout`。如果我们无法打开输入文件（例如它不存在），我们跳转到错误处理程序并退出。

如果一切顺利，我们接着检查第二个参数。如果它存在，我们打开输出文件。否则，我们将输出发送到 `stdout`。如果我们无法打开输出文件（例如它存在但我们没有写入权限），我们再次跳转到错误处理程序。

其余的代码与之前相同，唯一不同的是我们在退出之前关闭输入和输出文件，并且，如前所述，我们使用 `[fd.in]` 和 `[fd.out]`。

我们的可执行文件现在已经达到 768 字节。

我们还能改进它吗？当然！每个程序都可以改进。以下是一些可以做的改进：

* 让我们的错误处理程序打印一条消息到 `stderr`。
* 为 `read` 和 `write` 函数添加错误处理程序。
* 当我们打开输入文件时关闭 `stdin`，当我们打开输出文件时关闭 `stdout`。
* 添加命令行开关，比如 `-i` 和 `-o`，这样我们就可以任意顺序列出输入和输出文件，或者从 `stdin` 读取并写入到文件。
* 如果命令行参数不正确，打印使用帮助信息。

我将把这些改进留给读者：你已经知道实现它们所需要了解的一切。

## A.10. UNIX® 环境

UNIX® 中一个重要的概念是环境，它由 *环境变量* 定义。有些环境变量由系统设置，有些由你设置，还有一些由 shell 或任何加载其他程序的程序设置。

### A.10.1. 如何查找环境变量

我之前提到，当程序开始执行时，栈中包含 `argc` 后跟一个以 NULL 结束的 `argv` 数组，接着还有其他内容。这个“其他内容”就是 *环境*，更确切地说，是一个以 NULL 结束的指针数组，指向 *环境变量*。这通常被称为 `env`。

`env` 的结构与 `argv` 相同，是一系列内存地址后跟一个 NULL（`0`）。在这种情况下，没有 `"envc"` —— 我们通过查找最终的 NULL 来确定数组的结束。

这些变量通常以 `name=value` 格式出现，但有时 `=value` 部分可能缺失。我们需要考虑到这种可能性。

### A.10.2. webvars

我本可以直接展示一些代码，像 UNIX® 的 `env` 命令那样打印环境变量。但我认为编写一个简单的汇编语言 CGI 工具会更有趣。

#### A.10.2.1. CGI：快速概述

我在我的网站上有一个[详细的 CGI 教程](http://www.whizkidtech.redprince.net/cgi-bin/tutorial)，但这里有一个非常简短的概述：

* Web 服务器通过设置 *环境变量* 与 CGI 程序通信。
* CGI 程序将输出发送到 **stdout**。Web 服务器从那里读取输出。
* 它必须以 HTTP 头开始，后面跟着两个空行。
* 然后，它打印 HTML 代码或它正在生成的其他类型的数据。

>**注意**
>
>虽然某些 *环境变量* 使用标准名称，但其他变量会有所不同，具体取决于 Web 服务器。这使得 webvars 成为一个非常有用的诊断工具。

#### A.10.2.2. 代码

我们的 webvars 程序必须先发送 HTTP 头，接着是一些 HTML 标记。然后它必须逐个读取 *环境变量* 并将其作为 HTML 页面的一部分输出。

以下是代码。我在代码中直接插入了注释和解释：


```asm
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
	db	'<td class="name"><tt>'  ; 开始打印HTML表格的每一行（名称列）
leftlen	equ	$-left
middle	db	'</tt></td>', 0Ah
	db	'<td class="value"><tt><b>'  ; 在表格中加入值列
midlen	equ	$-middle
undef	db	'<i>(undefined)</i>'  ; 未定义的值
undeflen	equ	$-undef
right	db	'</b></tt></td>', 0Ah
	db	'</tr>', 0Ah  ; 表格结束的一行
rightlen	equ	$-right
wrap	db	'</table>', 0Ah
	db	'</div>', 0Ah
	db	'</body>', 0Ah
	db	'</html>', 0Ah, 0Ah  ; 完整的HTML结构结束
wraplen	equ	$-wrap

section	.text
global	_start
_start:
	; 首先，发送所有HTTP头部和XHTML内容
	push	dword httplen
	push	dword http
	push	dword stdout
	sys.write

	; 查找栈上环境变量指针的起始位置
	; 我们在“argc”之前已经推送了12个字节
	mov	eax, [esp+12]

	; 从栈中移除以下内容：
	; 1. sys.write所需的12字节
	; 2. argc的4字节
	; 3. argv所需的EAX*4字节
	; 4. argv之后的4字节NULL
	; 总计：
	; 20 + eax * 4
	; 因为栈是向下增长的，我们需要将这些字节数加到ESP中。
	lea	esp, [esp+20+eax*4]
	cld		; 确保标志已设置

	; 循环遍历环境变量并逐个打印
.loop:
	pop	edi
	or	edi, edi	; 检查是否遍历完环境变量
	je	near .wrap

	; 打印HTML表格的左部分（环境变量的名称）
	push	dword leftlen
	push	dword left
	push	dword stdout
	sys.write

	; 虽然可能会想直接查找'='，但是有些环境变量可能没有'='，因此我们先查找NULL字符。
	mov	esi, edi	; 保存字符串的起始位置
	sub	ecx, ecx
	not	ecx		; ECX = FFFFFFFF
	sub	eax, eax
repne	scasb
	not	ecx		; ECX = 字符串长度 + 1
	mov	ebx, ecx	; 将长度保存在EBX

	; 找到'='符号
	mov	edi, esi	; 字符串的起始位置
	mov	al, '='
repne	scasb
	not	ecx
	add	ecx, ebx	; 名称的长度

	; 打印名称部分
	push	ecx
	push	esi
	push	dword stdout
	sys.write

	; 打印HTML表格中间部分
	push	dword midlen
	push	dword middle
	push	dword stdout
	sys.write

	; 查找值的长度
	not	ecx
	lea	ebx, [ebx+ecx-1]

	; 如果值为0，则打印"undefined"
	or	ebx, ebx
	jne	.value

	mov	ebx, undeflen
	mov	edi, undef

.value:
	push	ebx
	push	edi
	push	dword stdout
	sys.write

	; 打印表格行的右部分
	push	dword rightlen
	push	dword right
	push	dword stdout
	sys.write

	; 清除已经推送的60字节
	add	esp, byte 60

	; 获取下一个环境变量
	jmp	.loop

.wrap:
	; 打印HTML的其余部分
	push	dword wraplen
	push	dword wrap
	push	dword stdout
	sys.write

	; 返回成功
	push	dword 0
	sys.exit
```

这段代码生成了一个 1,396 字节的可执行文件。大部分内容是数据，即我们需要发送的 HTML 标记。

按常规方法进行汇编和链接：

```
% nasm -f elf webvars.asm
% ld -s -o webvars webvars.o
```

要使用它，你需要将 **webvars** 上传到你的 Web 服务器。根据你的 Web 服务器配置，可能需要将它存储在一个特殊的 **cgi-bin** 目录中，或者可能需要将其重命名为 **.cgi** 扩展名。

然后，你需要使用浏览器查看它的输出。要查看我服务器上的输出，请访问 [http://www.int80h.org/webvars/](http://www.int80h.org/webvars/)。如果你对密码保护的 Web 目录中的附加环境变量感到好奇，可以访问 [http://www.int80h.org/private/](http://www.int80h.org/private/)，并使用用户名 `asm` 和密码 `programmer`。

## A.11. 处理文件

我们已经做了一些基本的文件操作：我们知道如何打开和关闭文件，如何使用缓冲区读取和写入文件。但 UNIX® 在处理文件时提供了更多的功能。在本节中，我们将研究其中的一些，并最终编写一个很好的文件转换工具。

事实上，让我们从结果开始，也就是文件转换工具。在开始编程时，知道最终产品应该做什么总是能让编程变得更容易。

我为 UNIX® 编写的第一个程序之一是 [tuc](ftp://ftp.int80h.org/unix/tuc/)，这是一个文本到 UNIX® 文件的转换器。它将来自其他操作系统的文本文件转换为 UNIX® 文本文件。换句话说，它将不同的行结束符转换为 UNIX® 的换行符约定。它将输出保存到一个不同的文件中。可选地，它也可以将 UNIX® 文本文件转换为 DOS 文本文件。

我广泛使用 tuc，但始终只是从其他操作系统转换为 UNIX®，从未反过来。我一直希望它能直接覆盖文件，而不是我必须将输出发送到另一个文件。大多数时候，我最终这样使用它：

```
% tuc myfile tempfile
% mv tempfile myfile
```

有了一个名为 `ftuc` 的工具，即 *快速 tuc*，就好了，我可以这样使用：

```
% ftuc myfile
```

因此，在这一章中，我们将用汇编语言编写 `ftuc`（原始的 tuc 是用 C 编写的），并在此过程中研究各种与文件相关的内核服务。

乍一看，文件转换似乎非常简单：你只需要去除回车符，对吗？

如果你回答是的，那就再想一想：这种方法大部分时间有效（至少对于 MS DOS 文本文件），但偶尔会失败。

问题在于，并不是所有非 UNIX® 文本文件的行都以回车符/换行符序列结束。有些文件使用仅回车符而没有换行符。其他文件将几个空行合并为一个回车符后接几个换行符。等等。

因此，文本文件转换器必须能够处理所有可能的行结束符：

* 回车符 / 换行符
* 回车符
* 换行符 / 回车符
* 换行符

它还应该处理使用上述某种组合的文件（例如，回车符后跟几个换行符）。

### A.11.1. 有限状态机

这个问题可以通过一种叫做 *有限状态机*（finite state machine）的技术轻松解决，这种技术最初由数字电子电路的设计师们开发。*有限状态机* 是一种数字电路，其输出不仅依赖于输入，还依赖于其先前的输入，即它的状态。微处理器就是一个 *有限状态机* 的例子：我们的汇编语言代码被组装成机器语言，其中一些汇编语言代码产生一个字节的机器语言，而其他一些则产生多个字节。当微处理器一个一个地从内存中获取字节时，有些字节仅仅改变其状态，而不是产生任何输出。当所有的操作码字节被获取后，微处理器才会产生输出，或者改变寄存器的值，等等。

因此，所有软件本质上都是一系列为微处理器编写的状态指令。尽管如此，*有限状态机* 的概念在软件设计中也非常有用。

我们的文本文件转换器可以被设计成一个 *有限状态机*，有三种可能的状态。我们可以将它们称为状态 0 到 2，但如果我们为它们起个符号名字会更容易：

* ordinary（普通）
* cr（回车）
* lf（换行）

我们的程序将从普通状态开始。在这个状态下，程序的动作取决于输入，具体如下：

* 如果输入是回车符或换行符以外的字符，输入会被直接传递到输出，状态保持不变。
* 如果输入是回车符，状态将切换到 cr，输入将被丢弃，即不输出任何内容。
* 如果输入是换行符，状态将切换到 lf，输入将被丢弃。

当我们处于 cr 状态时，意味着最后的输入是一个未处理的回车符。此时软件的行为依赖于当前输入：

* 如果输入是回车符或换行符以外的字符，则输出一个换行符，然后输出该输入，再将状态切换为普通状态。
* 如果输入是回车符，表示我们接收到了两个（或更多）连续的回车符。我们丢弃输入，输出一个换行符，并保持状态不变。
* 如果输入是换行符，我们输出换行符，并将状态切换为普通状态。注意，这与上面的第一种情况不同——如果我们尝试将它们合并，会导致输出两个换行符而不是一个。

最后，当我们接收到一个没有前置回车符的换行符时，我们会进入 lf 状态。这种情况发生在我们的文件已经是 UNIX® 格式，或者连续几行用一个回车符后跟多个换行符表示，或者行结束符是换行符/回车符序列时。在这个状态下，我们需要按照以下方式处理输入：

* 如果输入是回车符或换行符以外的字符，则输出一个换行符，然后输出该输入，再将状态切换为普通状态。这个操作与我们在 cr 状态下收到相同输入时的行为完全相同。
* 如果输入是回车符，我们丢弃输入，输出一个换行符，然后将状态切换为普通状态。
* 如果输入是换行符，我们输出换行符，并保持状态不变。

#### A.11.1.1. 最终状态

上述 *有限状态机* 适用于整个文件，但存在一个问题，即可能忽略最后一行的结束符。每当文件以单一的回车符或换行符结尾时，就会发生这种情况。我在编写 tuc 时没有考虑到这一点，后来才发现偶尔会剥离掉最后的行结束符。

这个问题可以通过在整个文件处理完毕后检查状态来轻松修复。如果状态不是普通状态，我们只需要输出一个最后的换行符。

>**注意**
>
> 现在我们已经将算法表示为 *有限状态机*，我们完全可以设计一个专用的数字电子电路（"芯片"）来为我们执行转换。当然，这样做的成本要比编写汇编语言程序高得多。

#### A.11.1.2. 输出计数器

由于我们的文件转换程序可能会将两个字符合并为一个字符，因此我们需要使用一个输出计数器。我们将其初始化为 `0`，并在每次将字符发送到输出时增加计数。程序结束时，计数器将告诉我们需要设置文件的大小。

### A.11.2. 在软件中实现 FSM

使用 *有限状态机* 的最大难点在于分析问题并将其表示为 *有限状态机*。一旦完成，软件几乎是自动生成的。

在高级语言中（如 C），有几种主要的方法。一个方法是使用 `switch` 语句来选择应该运行哪个函数。例如：

```c
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

另一种方法是使用一个函数指针数组，类似于这样：

```c
(output[state])(inputchar);
```

还有一种方法是让 `state` 成为一个函数指针，指向适当的函数：

```c
(*state)(inputchar);
```

我们在程序中将使用这种方法，因为它在汇编语言中非常容易实现，并且非常快速。我们将简单地将正确的程序地址存储在 `EBX` 中，然后执行：

```asm
call	ebx
```

这可能比硬编码地址更快，因为微处理器不需要从内存中获取地址——它已经存储在其寄存器之一中。我说 *可能* 更快，因为现代微处理器的缓存技术，任意一种方式可能同样快速。

### A.11.3. 内存映射文件

由于我们的程序操作的是单个文件，因此不能使用之前的方式，即从输入文件读取并将其写入输出文件。

UNIX® 允许我们将一个文件，或文件的某个部分，映射到内存中。为此，我们首先需要使用适当的读写标志打开文件。然后我们使用 `mmap` 系统调用将其映射到内存中。`mmap` 的一个优点是它自动与虚拟内存协作：我们可以将文件的更多部分映射到内存中，即使我们没有足够的物理内存，但仍然可以通过常规的内存操作码，如 `mov`、`lods` 和 `stos`，访问它。我们对文件内存映像所做的任何更改将由系统写入文件中。我们甚至不需要保持文件打开：只要它保持映射，我们就可以读取和写入它。

32 位的英特尔微处理器可以访问最多四千兆字节的内存——无论是物理内存还是虚拟内存。FreeBSD 系统允许我们使用最多一半的内存进行文件映射。

为了简化起见，在本教程中，我们将仅转换那些能够完全映射到内存中的文件。很少有文本文件的大小超过两千兆字节。如果我们的程序遇到这样的文件，它将简单地显示一条消息，建议我们使用原始的 tuc。

如果您检查 **syscalls.master**，您将找到两个名为 `mmap` 的独立系统调用。这是因为 UNIX® 的发展：有传统的 BSD `mmap`，即系统调用 71。这个版本被 POSIX® 的 `mmap`（系统调用 197）所取代。FreeBSD 系统同时支持两者，因为较旧的程序是使用原始的 BSD 版本编写的。但新软件使用的是 POSIX® 版本，这也是我们将使用的版本。

**syscalls.master** 列出了 POSIX® 版本，如下所示：

```c
197	STD	BSD	{ caddr_t mmap(caddr_t addr, size_t len, int prot, \
			    int flags, int fd, long pad, off_t pos); }
```

这与 [mmap(2)](https://man.freebsd.org/cgi/man.cgi?query=mmap&sektion=2&format=html) 中的描述略有不同。那是因为 [mmap(2)](https://man.freebsd.org/cgi/man.cgi?query=mmap&sektion=2&format=html) 描述的是 C 版本。

区别在于 `long pad` 参数，这在 C 版本中没有出现。然而，FreeBSD 系统调用会在 `push` 一个 64 位参数后，添加一个 32 位的填充。此时，`off_t` 是一个 64 位值。

当我们完成对内存映射文件的操作时，我们使用 `munmap` 系统调用来取消映射：

>**技巧**
>
>若想深入了解 `mmap`，请参阅 W. Richard Stevens 的 [Unix 网络编程，第 2 卷，第 12 章](http://www.int80h.org/cgi-bin/isbn?isbn=0130810819)。


### A.11.4. 确定文件大小

因为我们需要告诉 `mmap` 要将多少字节的文件映射到内存中，并且我们希望将整个文件映射到内存中，所以我们需要确定文件的大小。

我们可以使用 `fstat` 系统调用获取有关打开文件的所有信息，其中就包括文件大小。

同样，**syscalls.master** 列出了两个版本的 `fstat`，一个是传统的版本（系统调用 62），另一个是 POSIX® 版本（系统调用 189）。显然，我们将使用 POSIX® 版本：

```sh
189	STD	POSIX	{ int fstat(int fd, struct stat *sb); }
```

这是一个非常简单的调用：我们传入一个 `stat` 结构体的地址和一个打开文件的文件描述符。它会填充 `stat` 结构体的内容。

然而，我必须说我曾尝试将 `stat` 结构体声明在 `.bss` 区域，但 `fstat` 并不喜欢这种做法：它设置了进位标志，表示发生了错误。当我将代码更改为在堆栈上分配该结构体时，一切工作正常。

### A.11.5. 更改文件大小

由于我们的程序可能会将回车/换行序列合并为单一的换行符，因此我们的输出可能会比输入小。然而，由于我们将输出放入与输入文件相同的文件中，我们可能需要更改文件的大小。

`ftruncate` 系统调用允许我们做到这一点。尽管其名称可能会让人误解，但 `ftruncate` 系统调用可以用来既截断文件（使其变小），也可以将文件扩展。

是的，我们会在 **syscalls.master** 中找到两个版本的 `ftruncate`，一个较旧的版本（130），一个较新的版本（201）。我们将使用较新的版本：

```sh
201	STD	BSD	{ int ftruncate(int fd, int pad, off_t length); }
```

请注意，这里再次包含了 `int pad`。

### A.11.6. ftuc

现在我们已经知道了编写 ftuc 所需的一切。我们首先在 **system.inc** 中添加一些新行。首先，我们在文件的开始部分或接近开始的位置定义一些常量和结构体：

```asm
;;;;;;; 打开标志
%define	O_RDONLY	0
%define	O_WRONLY	1
%define	O_RDWR	2

;;;;;;; mmap 标志
%define	PROT_NONE	0
%define	PROT_READ	1
%define	PROT_WRITE	2
%define	PROT_EXEC	4
;;
%define	MAP_SHARED	0001h
%define	MAP_PRIVATE	0002h

;;;;;;; stat 结构体
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

```c
%define	SYS_mmap	197
%define	SYS_munmap	73
%define	SYS_fstat	189
%define	SYS_ftruncate	201
```

我们为它们的使用添加宏：

```asm
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

以下是我们的代码：

```asm
;;;;;;; 快速文本到 Unix 转换 (ftuc.asm) ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;
;; 开始日期：	2000年12月21日
;; 更新日期：	2000年12月22日
;;
;; 版权所有 2000 G. Adam Stanislav.
;; 保留所有权利。
;;
;;;;;;; v.1 ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
%include	'system.inc'

section	.data
	db	'版权所有 2000 G. Adam Stanislav.', 0Ah
	db	'保留所有权利。', 0Ah
usg	db	'用法： ftuc 文件名', 0Ah
usglen	equ	$-usg
co	db	"ftuc: 无法打开文件。", 0Ah
colen	equ	$-co
fae	db	'ftuc: 文件访问错误。', 0Ah
faelen	equ	$-fae
ftl	db	'ftuc: 文件过大，请使用常规的 tuc 工具。', 0Ah
ftllen	equ	$-ftl
mae	db	'ftuc: 内存分配错误。', 0Ah
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
	pop	eax		; 程序名称
	pop	ecx		; 要转换的文件
	jecxz	usage

	pop	eax
	or	eax, eax	; 参数太多？
	jne	usage

	; 打开文件
	push	dword O_RDWR
	push	ecx
	sys.open
	jc	cantopen

	mov	ebp, eax	; 保存文件描述符

	sub	esp, byte stat_size
	mov	ebx, esp

	; 获取文件大小
	push	ebx
	push	ebp		; fd
	sys.fstat
	jc	facerr

	mov	edx, [ebx + st_size + 4]

	; 如果 EDX != 0，则文件太大 ...
	or	edx, edx
	jne	near toolong
	mov	ecx, [ebx + st_size]
	; ... 或者文件超过 2 GB
	or	ecx, ecx
	js	near toolong

	; 如果文件大小为 0 字节，则不做任何操作
	jecxz	.quit

	; 将整个文件映射到内存中
	push	edx
	push	edx		; 从偏移量 0 开始
	push	edx		; 填充
	push	ebp		; fd
	push	dword MAP_SHARED
	push	dword PROT_READ | PROT_WRITE
	push	ecx		; 整个文件大小
	push	edx		; 让系统决定地址
	sys.mmap
	jc	near memerr

	mov	edi, eax
	mov	esi, eax
	push	ecx		; 对于 SYS_munmap
	push	edi

	; 使用 EBX 作为状态机
	mov	ebx, ordinary
	mov	ah, 0Ah
	cld

.loop:
	lodsb
	call	ebx
	loop	.loop

	cmp	ebx, ordinary
	je	.filesize

	; 输出最终的换行符
	mov	al, ah
	stosb
	inc	edx

.filesize:
	; 将文件截断为新大小
	push	dword 0		; 高字
	push	edx		; 低字
	push	eax		; 填充
	push	ebp
	sys.ftruncate

	; 关闭文件（ebp 仍然被压入栈）
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
	; 继续执行

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
	; 继续执行

.lf:
	stosb
	inc	edx
	ret
```

>**警告**
>
> 请勿在由 MS-DOS® 或 Windows® 格式化的磁盘上的文件上使用此程序。当在 FreeBSD 下使用 `mmap` 挂载这些磁盘时，FreeBSD 代码似乎存在一个微妙的 bug：如果文件超过某个大小，`mmap` 会将内存填充为零，然后将这些零复制到文件中，覆盖其内容。

## A.12. 一心一意的心态

作为禅宗的学生，我喜欢“一心一意”的想法：一次做一件事，并且做到最好。

实际上，这正是 UNIX® 的工作方式。典型的 Windows® 应用程序尝试做所有可以想象的事情（因此，充满了 bug），而典型的 UNIX® 程序只做一件事，而且做得很好。

典型的 UNIX® 用户基本上是通过编写一个 shell 脚本，将不同程序的输出通过管道连接起来，从而组装自己的应用程序。

在编写自己的 UNIX® 软件时，通常的好方法是，先看看现有的程序中有哪些部分可以帮助解决问题，然后只为那些没有现成解决方案的部分编写自己的程序。

### A.12.1. CSV

我将通过一个具体的实际例子来说明这个原则：

我需要提取从网站下载的数据库中每条记录的第 11 个字段。这个数据库是一个 CSV 文件，即一个*逗号分隔值*的列表。这是一种常见的数据共享格式，用于不同数据库软件之间的数据交换。

文件的第一行包含以逗号分隔的各种字段列表。文件的其余部分包含逐行列出的数据，每行的值通过逗号分隔。

我尝试使用 awk，将逗号作为分隔符。但因为某些行中包含了带引号的逗号，awk 从这些行中提取到了错误的字段。

因此，我需要编写自己的软件来提取 CSV 文件中的第 11 个字段。然而，遵循 UNIX® 精神，我只需要编写一个简单的过滤程序，完成以下操作：

* 删除文件的第一行；
* 将所有未加引号的逗号替换为其他字符；
* 删除所有引号。

严格来说，我可以使用 sed 删除文件的第一行，但自己编写这个程序非常简单，因此我决定这么做，并减少管道的复杂性。

无论如何，编写这样的程序大约花了我 20 分钟。编写一个提取 CSV 文件第 11 个字段的程序会花费更长时间，而且我无法重用它来提取其他数据库中的其他字段。

这一次，我决定让程序做得比典型的教程程序多一些工作：

* 它解析命令行参数；
* 如果发现错误参数，它会显示正确的用法；
* 它会产生有意义的错误信息。

以下是它的用法信息：

```sh
用法：csv [-t<delim>] [-c<comma>] [-p] [-o <outfile>] [-i <infile>]
```

所有参数都是可选的，可以按任何顺序出现。

`-t` 参数声明用来替换逗号的字符。默认情况下是使用 `tab`。例如，`-t;` 会将所有未加引号的逗号替换为分号。

我并不需要 `-c` 选项，但将来可能会用到。它允许我声明要用其他字符替换逗号。比如，`-c@` 会将所有的 @ 符号替换为其他字符（如果你想将一组电子邮件地址分割成用户名和域名，这非常有用）。

`-p` 选项保留第一行，即不删除它。默认情况下，我们会删除第一行，因为在 CSV 文件中，它包含的是字段名而不是数据。

`-i` 和 `-o` 选项让我指定输入文件和输出文件。默认值是 **stdin** 和 **stdout**，因此它是一个常规的 UNIX® 过滤器。

我确保 `-i filename` 和 `-ifilename` 都可以接受。我还确保只能指定一个输入文件和一个输出文件。


要获取每条记录的第 11 个字段，现在可以这样做：

```sh
% csv '-t;' data.csv | awk '-F;' '{print $11}'
```

该代码将选项（文件描述符除外）存储在 `EDX` 寄存器中：逗号存储在 `DH` 中，新的分隔符存储在 `DL` 中，`-p` 选项的标志存储在 `EDX` 的最高位中，因此检查其符号将快速决定我们应该执行什么操作。

下面是代码：

```asm
;;;;;;; csv.asm ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; 将逗号分隔的文件转换为其他分隔的文件。
;
; 开始时间： 31-May-2001
; 更新时间： 1-Jun-2001
;
; 版权所有 (c) 2001 G. Adam Stanislav
; 保留所有权利。
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
	push	dword 1		; 返回失败
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
	add	esp, byte 8	; 丢弃 argc 和 argv[0]
	mov	edx, (',' << 8) | 9

.arg:
	pop	ecx
	or	ecx, ecx
	je	near .init		; 没有更多参数

	; ECX 包含一个参数的指针
	cmp	byte [ecx], '-'
	jne	usage

	inc	ecx
	mov	ax, [ecx]

.o:
	cmp	al, 'o'
	jne	.i

	; 确保没有要求输出文件两次
	cmp	dword [fd.out], stdout
	jne	usage

	; 查找输出文件路径 - 它可能在 [ECX+1]，
	; 即 -ofile --
	; 或者在下一个参数中，
	; 即 -o file

	inc	ecx
	or	ah, ah
	jne	.openoutput
	pop	ecx
	jecxz	usage

.openoutput:
	push	dword 420	; 文件模式（644 八进制）
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

	; 确保没有要求输入文件两次
	cmp	dword [fd.in], stdin
	jne	near usage

	; 查找输入文件路径
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
	jc	near ierr		; 打开失败

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
	cmp	al, 't'		; 重新定义输出分隔符
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

	; 检查是否需要保留第一行
	or	edx, edx
	js	.loop

.firstline:
	; 去掉第一行
	call	getchar
	cmp	al, 0Ah
	jne	.firstline

.loop:
	; 从 stdin 中读取一个字节
	call	getchar

	; 它是逗号（或用户要求的其他字符）吗？
	cmp	al, dh
	jne	.quote

	; 将逗号替换为制表符（或用户要求的字符）
	mov	al, dl

.put:
	call	putchar
	jmp	short .loop

.quote:
	cmp	al, '"'
	jne	.put

	; 打印直到遇到另一个引号或 EOL。如果是引号，跳过它。如果是 EOL，打印它。
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
	call	write		; 刷新输出缓冲区

	; 关闭文件
	push	dword [fd.in]
	sys.close

	push	dword [fd.out]
	sys.close

	; 返回成功
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
	jecxz	.ret	; 没有什么可写的
	sub	edi, ecx	; 缓冲区起始位置
	push	ecx
	push	edi
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; 缓冲区现在为空
.ret:
	ret
```

其中许多内容取自上文的 **hex.asm**。但有一个重要的不同之处：我不再在输出换行符时每次调用 `write`。然而，这段代码仍然可以交互式使用。

自从我开始写这一章以来，我找到了解决交互式问题的更好方法。我希望确保每一行只在需要时才单独打印出来。毕竟，在非交互式使用时，没有必要每次都刷新每一行。

我现在使用的新解决方案是在发现输入缓冲区为空时每次调用 `write`。这样，当程序在交互模式下运行时，它会从用户的键盘读取一行，处理它，然后发现输入缓冲区为空。接着，它刷新输出并读取下一行。

#### A.12.1.1. 缓冲的黑暗面

这个改变避免了在某些特定情况下出现的神秘锁死问题。我将其称为 *缓冲的黑暗面*，主要是因为它存在一个并不明显的危险。

这种情况在像上面提到的 CSV 程序中不太可能发生，所以我们来考虑另一个过滤器：在这种情况下，我们预期输入是表示颜色值的原始数据，如像素的 *红色*、*绿色* 和 *蓝色* 强度。我们的输出将是输入的负值。

这样的过滤器非常简单。它的大部分代码将与我们之前编写的其他过滤器非常相似，所以我只会展示其内部循环部分：

```asm
.loop:
	call	getchar
	not	al		; 创建负值
	call	putchar
	jmp	short .loop
```

因为这个过滤器处理的是原始数据，它不太可能在交互模式下使用。

但它可能会被图像处理软件调用。如果没有在每次调用 `read` 之前调用 `write`，它很可能会锁死。

以下是可能发生的情况：

1. 图像编辑器使用 C 函数 `popen()` 加载我们的过滤器。
2. 它从位图或像素图中读取第一行像素。
3. 它将第一行像素写入到连接到我们过滤器的 `fd.in` 的 *管道* 中。
4. 我们的过滤器从输入中读取每个像素，将其转为负值，并写入输出缓冲区。
5. 我们的过滤器调用 `getchar` 来获取下一个像素。
6. `getchar` 发现输入缓冲区为空，于是它调用 `read`。
7. `read` 调用 `SYS_read` 系统调用。
8. *内核* 将暂停我们的过滤器，直到图像编辑器将更多数据发送到管道。
9. 图像编辑器从连接到我们过滤器的 `fd.out` 的另一个管道中读取，以便在发送第二行输入之前先设置第一行输出图像。
10. *内核* 暂停图像编辑器，直到它收到来自我们过滤器的某些输出，以便可以将其传递给图像编辑器。

此时，我们的过滤器等待图像编辑器发送更多数据供其处理，而图像编辑器在等待我们过滤器发送处理后的第一行结果。可是，结果仍然停留在输出缓冲区中。

过滤器和图像编辑器将永远互相等待（或者至少，直到它们被终止）。我们的软件已进入一个 [竞争条件](https://docs.freebsd.org/en/books/developers-handbook/secure/#secure-race-conditions)。

如果我们的过滤器在请求 *内核* 获取更多输入数据之前刷新其输出缓冲区，这个问题就不会发生。

## A.13. 使用 FPU

奇怪的是，大多数汇编语言文献甚至没有提到 FPU（*浮点单元*）的存在，更不用说讨论如何编程它了。

然而，汇编语言的光辉从未如此闪耀，尤其是在我们通过做一些只有 *汇编语言* 才能完成的事情来创建高度优化的 FPU 代码时。

### A.13.1. FPU 的组织结构

FPU 包含 8 个 80 位的浮点寄存器。这些寄存器以栈的方式组织——你可以将一个值 `push` 到栈顶（TOS，*top of stack*），也可以将其 `pop` 出来。

不过，汇编语言中的操作码不是 `push` 和 `pop`，因为这些操作码已经被占用了。

你可以通过使用 `fld`、`fild` 和 `fbld` 将值 `push` 到 TOS。还有一些其他的操作码允许你将一些常见的 *常量*（例如 *π*）推送到 TOS。

类似地，你可以使用 `fst`、`fstp`、`fist`、`fistp` 和 `fbstp` 来将值 `pop` 出来。实际上，只有那些以 *p* 结尾的操作码才会真正“弹出”该值，其余的则会将值存储到其他地方，而不会将其从 TOS 中移除。

我们可以将数据在 TOS 和计算机内存之间传输，无论是作为 32 位、64 位或 80 位的 *实数*，16 位、32 位或 64 位的 *整数*，还是 80 位的 *打包十进制*。

80 位 *打包十进制* 是一种特殊的 *二进制编码十进制*，在将数据的 ASCII 表示与 FPU 内部数据进行转换时非常方便。它允许我们使用 18 位有效数字。

无论我们如何在内存中表示数据，FPU 始终将其以 80 位的 *实数* 格式存储在寄存器中。

它的内部精度至少为 19 位十进制数字，因此即使我们选择以完整的 18 位精度显示结果，我们仍然能够显示正确的结果。

我们可以在 TOS 上执行数学运算：我们可以计算其 *正弦*，可以对其进行 *缩放*（即可以将其乘以或除以 2 的幂），我们可以计算其以 2 为底的 *对数*，以及许多其他操作。

我们还可以将其 *乘以* 或 *除以*，*加* 或 *减*，任何 FPU 寄存器中的值（包括它自身）。

官方的 Intel 操作码为 TOS 是 `st`，而寄存器为 `st(0)` 至 `st(7)`。因此，`st` 和 `st(0)` 指代的是同一个寄存器。

出于某些原因，nasm 的原作者决定使用不同的操作码，即 `st0` 至 `st7`。换句话说，没有圆括号，TOS 始终是 `st0`，从不单独使用 `st`。

#### A.13.1.1. 打包十进制格式

*打包十进制* 格式使用 10 字节（80 位）内存来表示 18 位数字。所表示的数字始终是 *整数*。

>**技巧**
>
>你可以通过先将 TOS 乘以 10 的幂来获得小数位。 

最高字节（字节 9）的最高位是 *符号位*：如果设置为 1，表示数字为 *负数*；否则为 *正数*。该字节的其余位未使用/忽略。

剩余的 9 个字节存储数字的 18 位：每个字节存储 2 位数字。

*更高位的数字* 存储在高 *半字节*（4 位），*较低位的数字* 存储在低 *半字节* 中。

话虽如此，你可能会认为 `-1234567` 会以如下方式存储在内存中（使用十六进制表示）：

```sh
80 00 00 00 00 00 01 23 45 67
```

可惜并不是！像所有其他 Intel 的东西一样，即使是 *打包十进制* 也是 *小端* 存储的。

这意味着我们的 `-1234567` 是这样存储的：

```sh
67 45 23 01 00 00 00 00 00 80
```

记住这一点，否则你会因绝望而拔掉头发！

>**注意**
>
> 你可以阅读的书——如果你能找到的话——是 Richard Startz 的 [8087/80287/80387 for the IBM PC & Compatibles](http://www.amazon.com/exec/obidos/ASIN/013246604X/whizkidtechnomag)。尽管它似乎理所当然地假设了 *打包十进制* 的小端存储。我并不夸张地说，在我发现我应该尝试小端顺序来处理这种数据之前，我在搞清楚下面展示的过滤器问题时几乎快要疯掉了。


### A.13.2. 针孔摄影的探索

为了编写有意义的软件，我们不仅需要理解我们的编程工具，还需要理解我们为其开发软件的领域。

我们的下一个过滤器将帮助我们在构建 *针孔相机* 时，所以在继续之前，我们需要了解一些 *针孔摄影* 的背景知识。

#### A.13.2.1. 相机

描述任何相机最简单的方法就是将其视为一个被某种防光材料包围的空腔，腔体上有一个小孔。

这个外壳通常是坚固的（例如一个盒子），有时也可能是柔性的（如伸缩筒）。相机内部相当黑暗。然而，小孔允许光线通过一个点进入（尽管在某些情况下可能有多个点）。这些光线形成了一个图像，表示相机外部的景物，位于小孔前面。

如果相机内部放置一些感光材料（例如胶片），它就能捕捉到图像。

小孔常常包含一个 *镜头*，或镜头组件，通常称为 *物镜*。

#### A.13.2.2. 针孔

但严格来说，镜头并不是必须的：最初的相机并没有使用镜头，而是使用了 *针孔*。即便今天，*针孔* 仍然被用作研究相机工作原理的工具，并用来实现一种特殊的图像效果。

针孔产生的图像是均匀清晰的，或者是 *模糊的*。针孔有一个理想的大小：如果它过大或过小，图像会失去锐度。

#### A.13.2.3. 焦距

这个理想的针孔直径是 *焦距* 的平方根的函数，焦距是针孔到胶片的距离。

```sh
D = PC * sqrt(FL)
```

其中，`D` 是理想的针孔直径，`FL` 是焦距，`PC` 是针孔常数。根据 Jay Bender 的说法，常数的值为 `0.04`，而 Kenneth Connors 确定其值为 `0.037`。其他人也提出了不同的值。而且，这个常数仅适用于日光：其他类型的光线将需要不同的常数，其值只能通过实验确定。

#### A.13.2.4. 光圈数

光圈数是衡量光线达到胶片的多少的一个非常有用的指标。一个光度计可以确定，例如，为了曝光某种特定灵敏度的胶片，f5.6 光圈可能需要曝光 1/1000 秒。

无论是 35 毫米相机，还是 6x9cm 相机等等，只要知道光圈数，我们就能确定适当的曝光时间。

光圈数的计算很简单：

```sh
F = FL / D
```

换句话说，光圈数等于焦距除以针孔直径。这也意味着较高的光圈数要么意味着较小的针孔，要么意味着较大的焦距，或者两者兼有。反过来，这意味着光圈数越高，曝光时间需要越长。

此外，虽然针孔直径和焦距是单维度的度量，但胶片和针孔都是二维的。这意味着，如果你在光圈数 `A` 下测量的曝光时间是 `t`，那么在光圈数 `B` 下的曝光时间就是：

```sh
t * (B / A)²
```

#### A.13.2.5. 标准化光圈数

虽然许多现代相机可以平滑而逐渐地改变针孔的直径，从而改变其光圈数，但并非总是如此。

为了适应不同的光圈数，相机通常包含一块金属板，上面钻有几个不同大小的孔。

这些孔的大小是根据上述公式选择的，以使得最终的光圈数是所有相机上使用的标准光圈数之一。例如，我拥有的一台非常旧的 Kodak Duaflex IV 相机就有三个这样的孔，光圈数分别为 8、11 和 16。

一台较新的相机可能提供的光圈数包括 2.8、4、5.6、8、11、16、22 和 32（以及其他值）。这些数字并不是随意选择的：它们都是 2 的平方根的幂，尽管它们可能被四舍五入了一些。

#### A.13.2.6. 光圈

典型的相机设计方式是，设置任何标准化的光圈数都会改变转盘的感觉。它会自然地在那个位置 *停止*。因此，这些转盘的位置被称为光圈档。

由于每个档的光圈数都是 2 的平方根的幂，因此将转盘移动 1 个档位将使所需的光线量翻倍。移动 2 个档位将使所需的曝光量增加 4 倍。移动 3 个档位则会使曝光量增加 8 倍，依此类推。

### A.13.3. 设计针孔软件

现在，我们可以决定我们的针孔软件到底要做什么。

#### A.13.3.1. 处理程序输入

由于其主要目的是帮助我们设计一个工作的针孔相机，我们将使用 *焦距* 作为程序的输入。这是我们可以在没有软件的情况下确定的：合适的焦距由胶片的大小以及拍摄“常规”照片、广角照片或远摄照片的需求决定。

到目前为止，我们编写的大多数程序都处理单个字符或字节作为输入：hex 程序将单个字节转换为十六进制数字，csv 程序则让一个字符通过，或删除它，或将其转换为另一个字符，等等。

有一个程序，ftuc 使用状态机最多处理两个输入字节。

但是，我们的针孔程序不能仅仅处理单个字符，它必须处理更大的语法单元。

例如，如果我们希望程序在焦距为 `100 mm`、`150 mm` 和 `210 mm` 时计算针孔直径（以及后续讨论的其他值），我们可能希望输入如下内容：

```sh
100, 150, 210
```

我们的程序需要同时处理多个字节的输入。当它看到第一个 `1` 时，它必须理解这是看到一个十进制数字的第一个数字。当它看到 `0` 和另一个 `0` 时，它必须知道这些是同一数字的后续数字。

当它遇到第一个逗号时，它必须知道不再接收第一个数字的数字。它必须能够将第一个数字的数字转换为值 `100`，第二个数字转换为值 `150`，当然，第三个数字转换为数值 `210`。

我们需要决定接受哪些分隔符：输入的数字必须用逗号分隔吗？如果是这样，如何处理由其他字符分隔的两个数字？

就我个人而言，我喜欢保持简单。要么是数字，我就处理它；要么不是数字，我就丢弃它。我不喜欢计算机抱怨我输入了一个额外的字符，特别是当那个字符 *显然* 是多余的时。天呐！

此外，这样做还可以打破计算的单调性，让我输入一个查询，而不仅仅是一个数字：

```sh
What is the best pinhole diameter for the
	    focal length of 150?
```

没有理由让计算机输出一堆抱怨：

```sh
Syntax error: What
Syntax error: is
Syntax error: the
Syntax error: best
```

等等，等等，等等。

其次，我喜欢使用 `#` 字符来表示从此开始到行尾的注释。这不需要太多编码工作，并且允许我将输入文件当作可执行脚本来处理。

在我们的案例中，我们还需要决定输入应使用什么单位：我们选择 *毫米*，因为大多数摄影师都使用这个单位来测量焦距。

最后，我们需要决定是否允许使用小数点（在这种情况下，我们还必须考虑到许多国家使用小数 *逗号*）。

在我们这个情况下，允许使用小数点或逗号会带来一种虚假的精度感：焦距 `50` 和 `51` 之间几乎没有明显的差别，所以允许用户输入像 `50.5` 这样的数值并不是一个好主意。这是我的个人观点，毕竟我是写这个程序的人。当然，你可以在自己的程序中做出不同的选择。

#### A.13.3.2. 提供选项

在构建针孔相机时，我们最需要知道的是针孔的直径。由于我们希望拍摄清晰的图像，我们将使用上述公式根据焦距来计算针孔的直径。由于专家们提供了不同的 `PC` 常数值，我们需要能够选择。

在 UNIX® 编程中，传统上有两种主要的选择程序参数的方法，并且在用户未做选择时，会有一个默认值。

为什么要有两种选择方式？

其中一种是允许（相对）*永久*的选择，这种选择每次运行软件时都会自动应用，而无需我们一遍又一遍地告诉它我们希望它做什么。

永久选择通常保存在配置文件中，通常位于用户的主目录中。该文件通常与应用程序同名，但前面加上一个点。通常会在文件名后加上 *"rc"*。因此，我们的文件可以是 **\~/.pinhole** 或 **\~/.pinholerc**。（**\~/** 表示当前用户的主目录。）

配置文件主要由具有多个可配置参数的程序使用。那些只有一个（或少量）参数的程序则常常使用另一种方法：它们期望在 *环境变量* 中找到该参数。在我们的情况下，我们可以查看名为 `PINHOLE` 的环境变量。

通常，一个程序只会使用上述两种方法之一。如果同时配置文件和环境变量中指定了不同的内容，程序可能会感到困惑（或者变得过于复杂）。

由于我们只需要选择 *一个* 这样的参数，我们将采用第二种方法，查找名为 `PINHOLE` 的环境变量。

另一种方式允许我们做 *临时* 的决策：*“虽然我通常希望你使用 0.039，但这次我想要 0.03872。”* 换句话说，它允许我们 *覆盖* 永久选择。

这种类型的选择通常通过命令行参数来完成。

最后，程序 *总是* 需要一个 *默认值*。用户可能不会做出任何选择。也许他不知道该选择什么，也许他只是“随便看看”。理想情况下，默认值将是大多数用户会选择的值。这样，他们就不需要选择。或者，准确地说，他们可以不做额外的努力而选择默认值。

在这种系统下，程序可能会找到相互冲突的选项，并以以下方式处理它们：

1. 如果它找到一个 *临时* 选择（例如，命令行参数），它应该接受该选择。它必须忽略任何永久选择和默认值。
2. *否则*，如果它找到一个永久选项（例如，环境变量），它应该接受该选项，并忽略默认值。
3. *否则*，它应该使用默认值。

我们还需要决定 `PC` 选项的 *格式* 应该是什么。

乍一看，使用 `PINHOLE=0.04` 格式的环境变量和 `-p0.04` 格式的命令行似乎是显而易见的。

然而，允许这样做实际上是一个安全隐患。`PC` 常数是一个非常小的数字。我们自然会使用不同的小值来测试我们的软件。但是，如果有人运行程序时选择了一个非常大的值，会发生什么呢？

程序可能会崩溃，因为我们没有设计它来处理巨大的数字。

或者，我们可能会花更多时间在程序上，使其能够处理巨大的数字。如果我们是在为计算机文盲的用户编写商业软件，可能会这样做。

或者，我们可能会说：“真倒霉！用户应该更懂得分寸。”

或者，我们可能干脆让用户无法输入巨大的数字。这就是我们要采取的做法：我们将使用一个 *隐式 0.* 前缀。

换句话说，如果用户希望输入 `0.04`，我们将期望他输入 `-p04`，或者在他的环境变量中设置 `PINHOLE=04`。因此，如果他说 `-p9999999`，我们将把它解释为 `0.9999999`——尽管仍然荒谬，但至少更加安全。

其次，许多用户可能只是想使用 Bender 的常数或 Connors 的常数。为了让他们更方便，我们将解释 `-b` 为与 `-p04` 相同，`-c` 为与 `-p037` 相同。

#### A.13.3.3. 输出

我们需要决定我们的软件要发送什么内容到输出，以及使用何种格式。

由于我们的输入允许不指定焦距条目的数量，因此使用传统的数据库样式输出每个焦距计算结果在一行中显示，且将每行中的所有值通过 `tab` 字符分隔是合乎逻辑的。

可选地，我们还应该允许用户指定使用我们之前学习过的 CSV 格式。在这种情况下，我们将首先输出一行由逗号分隔的名称，描述每一行的每个字段，然后像之前一样显示我们的结果，但用 `comma` 替换 `tab`。

我们需要为 CSV 格式提供一个命令行选项。我们不能使用 `-c`，因为它已经意味着 *使用 Connors' 常数*。由于某些奇怪的原因，许多网站将 CSV 文件称为 *“Excel 电子表格”*（尽管 CSV 格式早于 Excel）。因此，我们将使用 `-e` 选项来通知我们的软件我们希望输出为 CSV 格式。

我们将从输出的每一行的焦距开始。这乍一看可能会显得重复，尤其是在交互模式下：用户输入焦距，而我们又重复一遍。

但用户可以在一行中输入多个焦距。输入也可以来自文件，或者来自其他程序的输出。在这种情况下，用户根本看不到输入。

同样，输出可以被保存到一个文件中，我们之后可能会查看它，或者它可以打印出来，或者成为另一个程序的输入。

因此，从每一行开始都显示用户输入的焦距是完全合适的。

等等！不，不能直接按用户输入的方式显示。如果用户输入了像这样的内容：

```sh
00000000150
```

显然，我们需要去掉这些前导零。

所以，我们可能考虑按原样读取用户输入，在 FPU 中将其转换为二进制，然后从那里打印出来。

但是……

如果用户输入了像这样的内容：

```sh
17459765723452353453534535353530530534563507309676764423
```

哈哈！打包十进制 FPU 格式允许我们输入 18 位数字。但是用户输入了超过 18 位的数字。我们该如何处理？

嗯，我们 *可以* 修改代码，读取前 18 位数字，将其输入到 FPU，然后读取更多数字，将我们已经在 TOS 上的结果乘以 10 的幂，然后 `add` 到它。

是的，我们可以这么做。但是在 *这个* 程序中这是荒谬的（在另一个程序中可能正合适）：即使地球的周长用毫米表示也只需要 11 位数字。显然，我们不可能制造出这么大的相机（至少现在不行）。

所以，如果用户输入了如此巨大的数字，他要么是无聊，要么是在测试我们，要么是在尝试破坏系统，或者在玩游戏——总之，做的不是设计一个针孔相机。

我们该怎么做？

从某种意义上说，我们会给他一巴掌：

```sh
17459765723452353453534535353530530534563507309676764423	???	???	???	???	???
```

为了实现这一点，我们将简单地忽略所有前导零。一旦我们找到一个非零数字，我们将初始化一个计数器为 `0`，并开始执行三个步骤：

1. 将数字发送到输出。
2. 将数字附加到一个缓冲区，稍后我们将用它来生成可以发送到 FPU 的打包十进制。
3. 增加计数器。

现在，在执行这三个步骤时，我们还需要警惕以下两种情况之一：

* 如果计数器超过 18，我们停止将数字附加到缓冲区。我们继续读取数字并发送它们到输出。
* 如果，或者说 *当*，下一个输入字符不是数字时，我们就完成输入了。

顺便提一句，我们可以简单地丢弃非数字字符，除非它是 `#`，这种字符必须返回到输入流中。它表示开始一个注释，所以我们必须在完成输出生成后看到它，并开始查找更多的输入。

这仍然有一个未覆盖的情况：如果用户输入的只是零（或多个零），我们将永远找不到非零数字来显示。

我们可以在计数器保持为 `0` 时确定已经发生了这种情况。在这种情况下，我们需要将 `0` 输出，并进行另一次“巴掌”：

```sh
0	???	???	???	???	???
```

一旦我们显示了焦距并确认其有效（大于 `0` 且不超过 18 位数字），我们就可以计算针孔直径。

并非巧合的是，*pinhole* 包含了 *pin* 一词。实际上，许多针孔字面上就是 *pin hole*，即用针尖小心打孔的孔。

这就是因为典型的针孔非常小。我们的公式给出的结果是以毫米为单位的。我们将其乘以 `1000`，以便将结果以 *微米* 为单位输出。

在这时，我们将面临另一个问题：*过高的精度。*

是的，FPU 是为高精度数学设计的。但我们并不是在进行高精度数学计算。我们在处理的是物理学（特别是光学）。

假设我们想把一辆卡车改造成一个针孔相机（我们不会是第一个这么做的人！）。假设它的箱体长度为 `12` 米，所以焦距是 `12000`。使用 Bender 的常数，得出的是 `12000` 的平方根乘以 `0.04`，即 `4.381780460` 毫米，或者 `4381.780460` 微米。

无论哪种方式，结果都显得极为精确。我们的卡车不可能*恰好*是 `12000` 毫米长。我们没有用如此精确的标准来测量它的长度，因此说我们需要一个直径为 `4.381780460` 毫米的针孔是有误导性的。`4.4` 毫米就足够了。

>**注意**
>
>我在上面的例子中只用了十个数字。想象一下，如果我们追求所有 18 位数字的精度，那会有多荒谬！

我们需要限制结果的有效数字位数。一种方法是使用一个整数表示微米。所以，我们的卡车需要一个直径为 `4382` 微米的针孔。看看这个数字，我们仍然可以决定 `4400` 微米或 `4.4` 毫米就足够接近。

此外，我们还可以决定，无论结果多么大，我们只想显示四个有效数字（当然，也可以选择其他数字）。然而，FPU 并不提供四舍五入到特定数字位数的功能（毕竟，它并不是将数字视为十进制，而是视为二进制）。

因此，我们必须设计一个算法来减少有效数字位数。

这是我的算法（我觉得它有点笨拙——如果你知道一个更好的，请告诉我）：

1. 将计数器初始化为 `0`。
2. 当数字大于或等于 `10000` 时，将其除以 `10` 并增加计数器。
3. 输出结果。
4. 当计数器大于 `0` 时，输出 `0` 并减少计数器。

>**注意**
>
>如果你想要 *四个* 有效数字，`10000` 就合适。如果需要其他数量的有效数字，请将 `10000` 替换为 `10` 的对应幂。 
然后，我们将输出以微米为单位的针孔直径，四舍五入到四个有效数字。

此时，我们已经知道了 *焦距* 和 *针孔直径*，这意味着我们也有足够的信息来计算 *f 值*。

我们将显示 f 值，四舍五入到四个有效数字。f 值可能不会告诉我们太多信息。为了让它更有意义，我们可以找出最接近的 *归一化 f 值*，即最接近的平方根 2 的幂。

我们通过将实际的 f 值自乘来实现这一点，这当然会给我们它的 *平方*。然后，我们计算它的以 2 为底的对数，这比计算以平方根 2 为底的对数要容易得多！我们将结果四舍五入到最接近的整数。接下来，我们将 2 乘以这个结果，实际上，FPU 为我们提供了一个很好的捷径：我们可以使用 `fscale` 操作码来“缩放” 1，这类似于将整数左移。最后，我们计算它的平方根，我们就得到了最接近的归一化 f 值。

如果以上内容听起来让人不知所措——或者感觉工作量太大——也许看到代码后会变得更清晰。总共需要 9 条操作码：

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

第一行，`fmul st0, st0`，将 TOS（栈顶，称为 `st0`）的内容平方。`fld1` 将 `1` 推送到 TOS。

接下来，`fld st1` 将平方值再次推送到 TOS。此时，平方值既在 `st` 中，也在 `st(2)` 中（稍后会清楚为什么我们在栈上保留第二个副本）。`st(1)` 包含 `1`。

接下来，`fyl2x` 计算 `st` 与 `st(1)` 相乘后的以 2 为底的对数。这就是为什么我们在 `st(1)` 上放置 `1` 的原因。

此时，`st` 包含我们刚刚计算出的对数，`st(1)` 包含我们保存待用的实际 f 值的平方。

`frndint` 将 TOS 四舍五入到最接近的整数。`fld1` 再次推送 `1`。`fscale` 通过 `st(1)` 中的值来移动 TOS 上的 `1`，有效地将 2 的 `st(1)` 次幂。

最后，`fsqrt` 计算结果的平方根，即最接近的归一化 f 值。

现在，我们在 TOS 上有了最接近的归一化 f 值，`st(1)` 中有四舍五入后的以 2 为底的对数，而 `st(2)` 中仍然保存着我们实际的 f 值的平方。

但是我们不再需要 `st(1)` 中的内容。最后一行，`fstp st1` 将 `st` 中的内容存放到 `st(1)` 中，并将其弹出。结果，`st(1)` 的内容现在变成了 `st`，`st(2)` 变成了 `st(1)`，依此类推。新的 `st` 包含归一化 f 值，新的 `st(1)` 包含我们存储的实际 f 值的平方。

此时，我们准备输出归一化的 f 值。由于它是归一化的，我们将不对其进行四舍五入到四个有效数字，而是将其以完整的精度输出。

归一化的 f 值在它足够小并且可以在光照计上找到的情况下非常有用。否则，我们需要另一种确定合适曝光的方法。

之前我们已经弄清楚了如何从在不同 f 值下测量的曝光中计算适当的曝光。

我见过的所有光照计都能确定在 f5.6 下的适当曝光。因此，我们将计算一个 *“f5.6 乘数”*，即我们需要将 f5.6 下测得的曝光乘以多少，才能确定我们针孔相机的合适曝光。

根据上面的公式，我们知道这个乘数可以通过将我们的 f 值（实际值，而不是归一化值）除以 `5.6` 并平方来计算。

在数学上，将我们的 f 值的平方除以 `5.6` 的平方会得到相同的结果。

从计算的角度看，我们不希望平方两个数字，尤其是当我们可以只平方一个数字时。因此，第一种解决方案看起来更好。

但是……

`5.6` 是一个 *常数*。我们不需要让 FPU 浪费宝贵的周期。我们可以直接告诉它除以 `5.6²` 的结果，或者我们可以先将 f 值除以 `5.6`，然后平方结果。两者现在看起来差不多。

但它们并不相同！

通过上述摄影原理的学习，我们记得 `5.6` 实际上是 2 的平方根的五次方。一个 *无理数*。这个数字的平方正好是 `32`。

不仅 `32` 是一个整数，而且它是 2 的幂。我们不需要将 f 值的平方除以 `32`。我们只需要使用 `fscale` 将其右移五位。用 FPU 术语来说，就是我们将 `fscale` 它，`st(1)` 为 `-5`。这比除法要 *快得多*。

所以，现在已经清楚为什么我们在 FPU 栈上保存了 f 值的平方。计算 f5.6 乘数是这个程序中最简单的计算！我们将将它四舍五入到四个有效数字并输出。

还有一个有用的数字可以计算：我们的 f 值与 f5.6 之间的停距。这可能会帮助我们，如果我们的 f 值刚好超出光照计的范围，但我们有一个可以设置不同速度的快门，并且这个快门使用停距。

假设我们的 f 值与 f5.6 相差 5 停距，而光照计显示我们应该使用 1/1000 秒。那么我们可以首先设置快门速度为 1/1000，然后将拨盘调节 5 停距。

这个计算也很简单。我们所要做的就是计算我们刚刚计算的 f5.6 乘数的以 2 为底的对数（但我们需要的是它的未四舍五入的值）。然后我们将结果四舍五入到最接近的整数。我们不需要担心它有超过四个有效数字，因为结果很可能只有一位或两位数字。

### A.13.4. FPU 优化

在汇编语言中，我们可以通过一些高语言（包括 C）无法做到的方式优化 FPU 代码。

每当 C 函数需要计算一个浮点值时，它会将所有必要的变量和常量加载到 FPU 寄存器中。然后，它执行所需的计算以获得正确的结果。优秀的 C 编译器可以很好地优化代码的这一部分。

它通过将结果保留在 TOS（栈顶）上来“返回”值。然而，在返回之前，它会进行清理。它所使用的任何变量和常量都会从 FPU 中移除。

它不能像我们刚才所做的那样：我们计算了 f 值的平方，并将其保留在栈上，以便稍后由另一个函数使用。

我们 *知道* 我们稍后会需要这个值。我们还知道栈上有足够的空间（栈上最多能存放 8 个数字）来存储这个值。

而 C 编译器无法知道它在栈上的某个值会在不久的将来再次需要。

当然，C 程序员可能知道这一点，但唯一的解决方法就是将值存储到内存变量中。

这意味着：首先，值将从 FPU 内部使用的 80 位精度转换为 C 中的 *double*（64 位）或甚至 *single*（32 位）。

这也意味着该值必须从 TOS 移到内存中，然后再移回来。可惜的是，所有 FPU 操作中，访问计算机内存的操作是最慢的。

因此，在用汇编语言编写 FPU 代码时，尽量保持中间结果在 FPU 栈上。

我们甚至可以进一步优化！在我们的程序中，我们使用了一个 *常量*（我们命名为 `PC`）。

无论我们计算多少个针孔直径：1、10、20、1000，我们始终使用相同的常量。因此，我们可以通过将常量常驻栈上来优化程序。

在程序的早期，我们计算上述常量的值。我们需要将输入除以 `10`，每个常量的每个数字都需要这样做。

乘法比除法要快得多。因此，在程序开始时，我们将 `10` 除以 `1` 得到 `0.1`，然后将其保留在栈上：与其每次为每个数字除以 `10`，不如将其乘以 `0.1`。

顺便说一句，我们并没有直接输入 `0.1`，尽管我们可以这么做。我们这样做是有原因的：虽然 `0.1` 只需要一个小数位就能表示，但我们并不知道它在 *二进制* 中需要多少位。因此，我们让 FPU 计算其二进制值，精度由 FPU 自行决定。

我们还使用了其他常量：我们将针孔直径乘以 `1000`，以将其从毫米转换为微米；在将数字四舍五入到四个有效数字时，我们用 `10000` 来进行比较。所以，我们将 `1000` 和 `10000` 都保留在栈上。当然，我们在将数字四舍五入到四位时，也会重新使用 `0.1`。

最后但同样重要的是，我们将 `-5` 保留在栈上。我们需要它来缩放 f 值的平方，而不是将其除以 `32`。并且并非巧合，我们最后加载这个常量。这使得它在栈上是最顶层的常量。当 f 值的平方被缩放时，`-5` 就在 `st(1)` 上，正是 `fscale` 所期望的位置。

通常我们会从头创建某些常量，而不是从内存中加载它们。这就是我们对 `-5` 所做的事情：

```sh
fld1			; TOS =  1
	fadd	st0, st0	; TOS =  2
	fadd	st0, st0	; TOS =  4
	fld1			; TOS =  1
	faddp	st1, st0	; TOS =  5
	fchs			; TOS = -5
```

我们可以将这些优化总结为一个规则：*将重复的值保留在栈上！*

>**注意**
>
> *PostScript®* 是一种基于栈的编程语言。关于 PostScript® 的书籍要比关于 FPU 汇编语言的书籍多得多：掌握 PostScript® 将帮助你掌握 FPU。

### A.13.5. 针孔-代码

```asm
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

这段代码遵循了与我们之前看到的其他过滤器相同的格式，唯一的微妙例外是：

> 我们不再假设输入的结束意味着所有任务都已完成，这是在 *面向字符* 的过滤器中我们习以为常的做法。
>
> 这个过滤器不处理字符。它处理的是一种 *语言*（尽管是一个非常简单的语言，仅由数字组成）。
>
> 当没有更多输入时，可能意味着两件事之一：
>
> * 我们完成了，可以退出。这与之前相同。
> * 我们读取的最后一个字符是一个数字。我们已经将它存储在我们的 ASCII 到浮点数转换缓冲区的末尾。现在，我们需要将该缓冲区的内容转换为一个数字，并写出最后一行输出。
>
> 因此，我们修改了 `getchar` 和 `read` 例程，使得在我们从输入中获取另一个字符时，`carry flag` 始终为 *清除*，或者在没有更多输入时，`carry flag` 为 *设置*。
>
> 当然，我们仍然使用汇编语言魔法来实现这一点！仔细看看 `getchar`。它 *总是* 在返回时将 `carry flag` *清除*。
>
> 然而，我们的主代码依赖于 `carry flag` 来告诉它何时退出——并且它工作得很好。
>
> 这个魔法出现在 `read` 中。每当它从系统接收到更多输入时，它会返回到 `getchar`，`getchar` 从输入缓冲区获取一个字符，*清除* `carry flag` 并返回。
>
> 但当 `read` 从系统接收到没有更多的输入时，它 *不会* 返回到 `getchar`。相反，`add esp, byte 4` 操作码将 `4` 加到 `ESP` 中，*设置* `carry flag` 并返回。
>
> 那么，它返回到哪里呢？每当程序使用 `call` 操作码时，微处理器会将返回地址 *压入* 栈顶（即将其存储在系统栈中，而不是 FPU 栈中）。当程序使用 `ret` 操作码时，微处理器会从栈中 *弹出* 返回地址，并跳转到存储在该地址的地方。
>
> 但是，由于我们将 `4` 加到 `ESP`（栈指针寄存器），我们实际上给微处理器带来了轻微的 *失忆症*：它不再记得是 `getchar` 调用了 `read`。
>
> 由于 `getchar` 在调用 `read` 之前并没有压入任何内容，因此栈顶现在包含了调用 `getchar` 的程序的返回地址。就该调用者而言，它调用了 `getchar`，而 `getchar` 返回时将 `carry flag` 设置好了！

除此之外，`bcdload` 例程处于大端和小端之间的一个小冲突之中。

它正在将数字的文本表示转换为该数字：文本以大端顺序存储，但 *打包的十进制* 是小端顺序。

为了解决这个冲突，我们在开始时使用了 `std` 操作码。稍后我们使用 `cld` 来取消它：在 `std` 活跃时，我们非常重要的一点是不要调用任何可能依赖于 *方向标志* 默认设置的内容。

代码中的其他部分应该很清晰，前提是你已经阅读了之前的整个章节。

这是一个经典的例子，证明了编程需要大量的思考和很少的编码。只要我们将每个细节都考虑清楚，代码几乎就会自己写出来。


