# 第 2 章 编程工具

## 2.1. 简介

本章是介绍如何使用 FreeBSD 提供的一些编程工具的简介，尽管其中的大部分内容也适用于许多其他版本的 UNIX®。它并不试图详细描述编码。本章的大部分内容假定读者几乎没有编程知识，尽管希望大多数程序员会在其中找到一些有价值的东西。

## 2.2. 引言

FreeBSD 提供了一个出色的开发环境。C 和 C++的编译器以及汇编器与基本系统一起提供，更不用说经典的 UNIX 工具如 sed 和 awk 了。如果这还不够，还有许多编译器和解释器在Ports集合中。下面的部分，编程入门，列出了一些可用的选项。FreeBSD 与 POSIX 和 ANSI C 等标准非常兼容，同时也与其自己的 BSD 传统兼容，因此可以编写在各种平台上几乎不经过或经过很少修改就能编译和运行的应用程序。

但是，如果您以前从未在 UNIX 平台上编写程序，所有这些功能可能会让人感到非常不知所措。本文档旨在帮助您快速上手，而不深入探讨更高级的主题。本文档的目的是给您提供足够的基础知识，以便能够理解文档的内容。

大多数文档无需或几乎无需编程知识，尽管它确实假定您具备使用 UNIX 和学习的基本能力！

## 2.3. 编程入门

程序是一组指令，告诉计算机做各种事情；有时候，它必须执行的指令取决于它执行前一个指令时发生了什么。本节概述了您可以给出这些指令的两种主要方式，通常称为“命令”。一种方式使用解释器，另一种方式使用编译器。由于人类语言对计算机来说太难以以一种明确的方式理解，因此通常会用专门设计用于此目的的语言编写命令。

### 2.3.1. 解释器

有了解释器，语言就成为了一个环境，在这里你在提示符下键入命令，环境会为你执行这些命令。对于更复杂的程序，你可以把命令键入文件，并让解释器加载该文件并执行其中的命令。如果出了问题，许多解释器会将你转到调试器以帮助你跟踪问题。

其优点在于你可以立即看到命令的结果，错误可以很容易地纠正。最大的缺点是当你想与他人共享你的程序时。他们必须有相同的解释器，或者你必须有某种方法将其提供给他们，并且他们需要理解如何使用它。此外，用户可能不喜欢如果他们按错键就被送入调试器！从性能角度来看，解释器可能会使用大量内存，并且通常不像编译器那样高效地生成代码。

在我看来，如果你以前没有进行过任何编程，解释性语言是开始的最佳方式。这种环境通常在诸如 Lisp、Smalltalk、Perl 和 Basic 等语言中找到。也可以说 UNIX®shell ( sh , csh )本身就是一个解释器，并且事实上许多人确实编写shell "脚本"来帮助执行各种在他们的机器上的"家务"任务。事实上，UNIX®原始哲学的一部分是提供许多可以在shell脚本中链接在一起以执行有用任务的小型实用程序。

### 在 FreeBSD 中提供的解释器

这里是从 FreeBSD Ports Collection 可用的解释器列表，简要讨论了一些更受欢迎的解释型语言。

如何从手册的Ports部分找到并安装Ports Collection 中的应用程序的说明。

BASIC 是“初学者通用符号指令代码”的缩写。在上世纪 50 年代为大学生编程教学开发，80 年代随着每台值得尊敬的个人电脑提供，BASIC 是许多程序员的第一种编程语言。它也是 Visual Basic 的基础。

Bywater Basic 解释器可以在Ports集合中的 lang/bwbasic 找到，Phil Cockroft 的 Basic 解释器（前身为 Rabbit Basic）可在 lang/pbasic 中找到。

Lisp 是一种在上世纪 50 年代开发的语言，作为当时流行的“数值计算”语言的替代方案。与基于数字不同，Lisp 是基于列表的；事实上，名称是“List Processing”的缩写。在人工智能领域非常流行。

Lisp 是一种非常强大和复杂的语言，但可能相当庞大和笨重。

可在 FreeBSD 的Ports集合中找到可在 UNIX®系统上运行的各种 Lisp 实现。GNU Common Lisp 可以在 lang/gcl 中找到。由 Bruno Haible 和 Michael Stoll 开发的 CLISP 可作为 lang/clisp 使用。对于 CMUCL，其中还包括一个高度优化的编译器，或者像 SLisp 这样的简单的 Lisp 实现，它在几百行 C 代码中实现了大部分 Common Lisp 构造，分别可以在 lang/cmucl 和 lang/slisp 中找到。

Perl 非常受系统管理员欢迎，用于编写脚本；也经常用于编写 CGI 脚本的 World Wide Web 服务器。

Perl 在Ports集合中作为 lang/perl5.24 对所有 FreeBSD 版本可用。

Scheme 是 Lisp 的一个方言，比 Common Lisp 更紧凑、更干净。在大学中很受欢迎，因为它简单到可以教给本科生作为第一门语言，同时又具有足够高的抽象级别可用于研究工作。

Scheme 可从Ports集合中作为 lang/elk 获取 Elk Scheme 解释器。MIT Scheme 解释器可在 lang/mit-scheme 找到，SCM Scheme 解释器在 lang/scm 中。

IconIcon 是一种具有广泛处理字符串和结构的高级语言。FreeBSD 上的 Icon 版本可以在 Ports 集合中找到，位置为 lang/icon。

LuaLua 是一种轻量级的可嵌入脚本语言。它具有广泛的可移植性，相对简单。Lua 可以在 lang/lua 中的 Ports 集合中找到。它也作为 /usr/libexec/flua 包含在基本系统中，供基本系统组件使用。第三方软件不应依赖 flua。

PythonPython 是一种面向对象的解释性语言。其支持者认为它是开始编程的最佳语言之一，因为相对容易入门，但与其他用于开发大型复杂应用程序的流行解释性语言相比并不受限制（Perl 和 Tcl 是用于此类任务的另外两种流行语言）。

Python 的最新版本可从 lang/python 的Ports集合中获取。

RubyRuby 是一种解释型的、纯面向对象的编程语言。由于其易于理解的语法、编写代码时的灵活性以及轻松开发和维护大型复杂程序的能力，它已经变得非常流行。

Ruby 可从 lang/ruby32 的Ports集合中获取。

Tcl 和 TkTcl 是一种可嵌入的解释性语言，因其在许多平台上的可移植性而变得广泛使用并因此变得流行。它可用于快速编写小型原型应用程序，或者（与 Tk，一个 GUI 工具包相结合）用于编写功能丰富的完整程序。

FreeBSD 提供各种版本的 Tcl 作为 ports。最新版本 Tcl 8.7 可在 lang/tcl87 中找到。

### 2.3.3. 编译器

编译器有所不同。首先，您使用编辑器在文件（或文件）中编写代码。然后运行编译器，看看它是否接受您的程序。如果没有编译成功，请咬紧牙关，回到编辑器；如果编译成功并给您一个程序，您可以在shell命令提示符或调试器中运行它，以查看它是否正常工作。^[ 1]^

很明显，这并不像使用解释器那样直接。然而，它使您可以做很多解释器很难甚至不可能做到的事情，比如编写与操作系统密切交互的代码-甚至编写您自己的操作系统！如果您需要编写非常高效的代码，则编译器非常有用，因为编译器可以花费时间优化代码，而这在解释器中是不可接受的。此外，为编译器编写的程序通常比为解释器编写的程序更简单-您可以简单地给他们一个可执行文件副本，假设他们使用与您相同的操作系统。

由于在使用单独的程序时编辑-编译-运行-调试周期相当乏味，许多商业编译器制造商已经生产了集成开发环境（简称 IDE）。FreeBSD 在基本系统中不包括 IDE，但在Ports集合中提供了 devel/kdevelop，并且许多人使用 Emacs 以此为目的。有关将 Emacs 用作 IDE，请参阅《使用 Emacs 作为开发环境》。

## 2.4. 使用 cc 进行编译

本节介绍用于 C 和 C ++的 clang 编译器，因为它安装在 FreeBSD 基本系统中。 clang 安装为 cc ；GNU 编译器 gcc 可在Ports Collection 中获得。 使用解释器生成程序的细节在解释器的文档和在线帮助中通常有很好的介绍。

一旦您完成了您的杰作，下一步是将其转换为可以（希望！）在 FreeBSD 上运行的东西。 这通常涉及几个步骤，每个步骤由一个单独的程序完成。

1. 预处理您的源代码，删除注释并执行其他技巧，如在 C 中展开宏。
2. 检查代码的语法，看看您是否遵守了语言的规则。如果没有，它会抱怨！
3. 将源代码转换为汇编语言-这非常接近机器代码，但仍可被人类理解。据说。
4. 将汇编语言转换为机器码-是的，我们在谈论位和字节，这里是一和零。
5. 检查您是否以一致的方式使用了诸如函数和全局变量之类的东西。例如，如果您调用了一个不存在的函数，它会报错。
6. 如果您正在尝试从几个源代码文件生成可执行文件，请想办法将它们全部组合在一起。
7. 弄清楚如何生成系统运行时加载程序能够加载到内存并运行的内容。
8. 最后，在文件系统上编写可执行文件。

编译一词通常用于指代步骤 1 至 4，其他步骤则称为链接。有时步骤 1 被称为预处理，步骤 3 至 4 被称为汇编。

幸运的是，几乎所有这些细节对您而言都是隐藏的，因为 cc 是一个前端，它会为您调用所有这些程序并使用正确的参数；只需键入

```
% cc foobar.c
```

将导致 foobar.c 通过上述所有步骤进行编译。如果您有多个文件要编译，只需执行类似操作

```
% cc foo.c bar.c
```

请注意，语法检查只是检查语法而已。它不会检查您可能犯的任何逻辑错误，比如将程序放入无限循环中，或者在本应使用二进制排序时使用冒泡排序。^[ 2]^

有很多选项可供 cc 使用，所有这些选项都在手册页中。这里是一些最重要的选项，以及如何使用它们的示例。

-o<span> </span><em>filename</em> 文件的输出名称。如果你不使用这个选项， cc 将生成一个名为 a.out 的可执行文件。

```
% cc foobar.c               executable is a.out
% cc -o foobar foobar.c     executable is foobar
```

-c 只编译文件，不链接它。这对于你只想检查语法的玩具程序，或者如果你在使用 Makefile 时非常有用。

```
% cc -c foobar.c
```

这将产生一个对象文件（而不是可执行文件）称为 foobar.o。这可以与其他对象文件链接在一起成为一个可执行文件。

-g 创建可执行文件的调试版本。这使编译器将关于哪个源文件的哪一行对应于哪个函数调用的信息放入可执行文件中。调试器可以使用这些信息在您逐步执行程序时显示源代码，这非常有用；缺点是所有这些额外信息使程序变得更大。通常，在开发程序时您使用 -g 进行编译，然后在满意程序正常工作时，不使用 -g 进行编译生成“发布版本”。

```
% cc -g foobar.c
```

这将产生程序的调试版本。 ^[4]^

-O 创建可执行文件的优化版本。编译器会尝试执行各种巧妙的技巧，以产生比正常情况下运行更快的可执行文件。您可以在 -O 后面添加一个数字来指定更高级别的优化，但这经常会暴露编译器优化器中的错误。

```
% cc -O -o foobar foobar.c
```

这将生成 foobar 的优化版本。

以下三个标志将强制 cc 检查您的代码是否符合相关的国际标准，通常称为 ANSI 标准，严格来说它是 ISO 标准。

-Wall 启用 cc 的作者认为值得的所有警告。尽管名称如此，它不会启用 cc 能够的所有警告。

-ansi 关闭 cc 提供的大多数非 ANSI C 特性。尽管名称如此，它并不能严格保证您的代码符合标准。

-pedantic 关闭所有 cc 的非 ANSI C 特性。

没有这些标志， cc 将允许您使用其标准之外的一些非标准扩展。其中一些非常有用，但无法与其他编译器一起使用 - 实际上，标准的主要目标之一是允许人们编写可以在任何系统上的任何编译器上运行的代码。这被称为可移植代码。

通常，您应该尽量使您的代码尽可能具有可移植性，否则您以后可能不得不完全重写程序才能使其在其他地方运行 - 谁知道您未来几年可能会使用什么？

```
% cc -Wall -ansi -pedantic -o foobar foobar.c
```

在检查 foobar.c 是否符合标准后，这将生成一个名为 foobar 的可执行文件。

-l<em>library</em> 指定在链接时要使用的函数库。

最常见的例子是编译使用 C 语言中的一些数学函数的程序。与大多数其他平台不同，这些函数在与标准 C 库不同的库中，您必须告诉编译器将其添加进去。

规则是，如果库被称为 libsomething.a，你给 cc 参数 -l<em>something</em> 。例如，数学库是 libm.a，所以你给 cc 参数 -lm 。数学库的一个常见问题是它必须是命令行中的最后一个库。

```
% cc -o foobar foobar.c -lm
```

这将链接数学库函数到 foobar 中。

如果您正在编译 C++ 代码，请使用 c++。在 FreeBSD 上也可以使用 clang++ 调用 c++。

```
% c++ -o foobar foobar.cc
```

这将从 C++ 源文件 foobar.cc 生成可执行文件 foobar。

### 2.4.1. 常见 cc 查询和问题

#### 2.4.1.1. 我编译了一个名为 foobar.c 的文件，但我找不到一个名为 foobar 的可执行文件。它去哪了？

记住， cc 会调用名为 a.out 的可执行文件，除非你另行指定。使用 -o<span> </span><em>filename</em> 选项：

```
% cc -o foobar foobar.c
```

#### 2.4.1.2. 好的，我有一个名为 foobar 的可执行文件，在运行 ls 时可以看到它，但是当我在命令提示符中输入 foobar 时，它告诉我找不到这个文件。为什么找不到它呢？

与 MS-DOS®不同，UNIX®在尝试找出您要运行的可执行文件时不会查找当前目录，除非您告诉它。键入 ./foobar ，这意味着“运行当前目录中名为 foobar 的文件”。

### 2.4.2. 我将我的可执行文件命名为 test，但是当我运行它时什么都不会发生。出了什么问题？

大多数 UNIX®系统都有一个名为 test 的程序在/usr/bin 中，而shell会在检查当前目录之前先运行它。要么输入：

```
% ./test
```

要么为您的程序选择一个更好的名称！

#### 2.4.2.1. 我编译了我的程序，起初似乎一切正常，然后出现了一个错误，说了一些关于核心转储的东西。那是什么意思？

核心转储的名称可以追溯到 UNIX® 的早期，当时机器使用核心存储数据。 基本上，如果程序在某些条件下失败，系统会将核心存储器的内容写入一个名为 core 的文件中，程序员随后可以查看该文件以找出问题所在。

#### 2.4.2.2。令人着迷的东西，但我现在该怎么办？

使用调试器分析核心（请参阅调试）。

#### 当我的程序转储核心时，它显示关于分段错误的信息。那是什么意思？

这基本上意味着您的程序尝试在内存上执行某种非法操作；UNIX®旨在保护操作系统和其他程序免受恶意程序的侵害。

这种情况的常见原因有：

* 尝试写入空指针，例如

  ```
  char *foo = NULL;
  strcpy(foo, "bang!");
  ```
* 使用未初始化的指针，例如

  ```
  char *foo;
  strcpy(foo, "bang!");
  ```

  指针将具有一些随机值，幸运的话，它将指向内存中不可用于您的程序的区域，内核将在您的程序造成任何损害之前终止您的程序。如果您不幸，它将指向您自己程序内的某个位置，并损坏您的数据结构之一，导致程序神秘地失败。
* 尝试访问数组末尾之后的内容，例如

  ```
  int bar[20];
  bar[27] = 6;
  ```
* 尝试将某物存储在只读内存中，例如

  ```
  char *foo = "My string";
  strcpy(foo, "bang!");
  ```

  UNIX® 编译器通常将类似 "My string" 的字符串文本放入只读内存区域。
* 与 malloc() 和 free() 一起做坏事，例如

  ```
  char bar[80];
  free(bar);
  ```

  要么

  ```
  char *foo = malloc(27);
  free(foo);
  free(foo);
  ```

做这些错误中的一个通常不会导致错误，但这总是一个不好的习惯。一些系统和编译器比其他系统更宽容，这就是为什么在一个系统上运行良好的程序在另一个系统上尝试时可能会崩溃。

#### 2.4.2.4. 有时候当我收到核心转储时，它会显示总线错误。我的 UNIX®书上说这意味着硬件问题，但计算机似乎仍在正常运行。这是真的吗？

不，幸运的是不会（除非当然你真的遇到了硬件问题...）。这通常是另一种说法，即您以不应该的方式访问了内存。

#### 2.4.2.5. 这个转储核心的过程听起来可能非常有用，如果我想要发生这种情况，我可以做到吗？还是我必须等到出现错误？

是的，只需转到另一个控制台或 xterm，执行

```
% ps
```

以查找您的程序的进程 ID，并执行

```
% kill -ABRT pid
```

其中 <em>pid</em> 是您查找的进程 ID。

如果您的程序陷入无限循环，这将非常有用。例如，如果您的程序恰好陷入 SIGABRT 陷阱，还有其他几个信号会产生类似的效果。

或者，您可以通过调用 abort() 函数从程序内部创建核心转储。查看 abort(3)的手册页以了解更多信息。

如果您想从程序外部创建核心转储，但又不希望进程终止，可以使用 gcore 程序。有关更多信息，请参阅 gcore(1)的手册页。

## 2.5. 制造

### 2.5.1. 什么是 make ?

当你在处理一个只有一个或两个源文件的简单程序时，键入

```
% cc file1.c file2.c
```

不是太糟糕，但当有多个文件时很快变得非常乏味，而且编译也可能需要一段时间。

一个解决方法是使用目标文件，只有在源代码发生更改时才重新编译源文件。所以我们可以有类似这样的东西：

```
% cc file1.o file2.o … file37.c …
```

如果我们已经更改了 file37.c，但没有更改其他任何文件，自上次编译以来。这可能会大大加快编译速度，但并不能解决输入问题。

或者我们可以编写一个shell脚本来解决输入问题，但这将需要重新编译所有内容，在大型项目上效率会很低。

如果我们有成百上千的源文件散落在周围会发生什么？如果我们和其他人合作，在使用其他源文件时他们忘记告诉我们已经更改了其中一个源文件，会怎样？

或许我们可以将两种解决方案结合起来，编写类似shell脚本的东西，其中包含一些神奇的规则，规定何时需要编译源文件。现在我们所需要的是一个能够理解这些规则的程序，因为这对shell来说有点太复杂了。

这个程序叫做 make 。它读取一个名为 makefile 的文件，该文件告诉它不同文件之间的依赖关系，并计算出哪些文件需要重新编译，哪些不需要。例如，一条规则可以说“如果 fromboz.o 比 fromboz.c 旧，这意味着有人修改了 fromboz.c，所以需要重新编译。” makefile 还包含告诉 make 如何重新编译源文件的规则，使其成为一个更强大的工具。

Makefile 通常与其所适用的源代码保存在同一个目录中，可以命名为 makefile、Makefile 或 MAKEFILE。大多数程序员使用名称 Makefile，因为这样可以将其放在目录列表的顶部，便于查看。^[ 5]^

### 2.5.2. 使用 make 的示例

这是一个非常简单的 make 文件：

```
foo: foo.c
	cc -o foo foo.c
```

它由两行组成，一个依赖行和一个创建行。

这里的依赖行由程序的名称（称为目标），后跟一个冒号，然后是空格，然后是源文件的名称组成。当 make 读取这行时，它会查看 foo 是否存在；如果存在，它会比较 foo 上次修改的时间和 foo.c 上次修改的时间。如果 foo 不存在，或者比 foo.c 旧，那么它会查看创建行以找出该执行什么操作。换句话说，这是确定何时需要重新编译 foo.c 的规则。

创建行以制表符开头（按制表符键），然后是您在命令提示符处创建 foo 时要键入的命令。如果 foo 已过时或不存在，则 make 会执行此命令以创建它。换句话说，这是告诉 make 如何重新编译 foo.c 的规则。

因此，当您键入 make 时，它将确保 foo 相对于您对 foo.c 的最新更改是最新的。这个原则可以扩展到具有数百个目标的 Makefile-事实上，在 FreeBSD 上，只需在适当的目录中键入 make world 就可以编译整个操作系统！

Makefile 的另一个有用属性是目标不必是程序。例如，我们可以有一个看起来像这样的 make 文件：

```
foo: foo.c
	cc -o foo foo.c

install:
	cp foo /home/me
```

我们可以通过输入来告诉 make 我们要制作哪个目标：

```
% make target
```

make 然后只会查看该目标，忽略其他任何目标。例如，如果我们在上述 makefile 中键入 make foo ，make 将忽略 install 目标。

如果我们只输入 make ，make 将始终查看第一个目标，然后停止而不查看其他任何目标。因此，如果我们在这里输入 make ，它将直接转到 foo 目标，如果需要的话重新编译 foo，然后停止而不继续到 install 目标。

注意， install 目标实际上并不依赖于任何东西！这意味着当我们尝试通过键入 make install 来生成该目标时，下一行上的命令总是被执行。在这种情况下，它将 foo 复制到用户的主目录中。这在应用程序的 makefile 中经常被使用，这样当应用程序被正确编译后，可以安装在正确的目录中。

这是一个稍微令人困惑的主题要去解释。如果你不太理解 make 的工作原理，最好的方法是编写一个简单的程序像“hello world”，以及一个像上面那样的 make 文件，并进行实验。然后逐渐使用多个源文件，或者将源文件包含一个头文件。 touch 在这里非常有用-它可以更改文件的日期而无需手动编辑。

### 2.5.3. Make 和 include 文件

C 代码通常以要包含的文件列表开头，例如 stdio.h。其中一些文件是系统包含文件，另一些来自您目前正在工作的项目：

```
#include <stdio.h>
#include "foo.h"

int main(....
```

为了确保在 foo.h 更改时重新编译此文件，您必须在 Makefile 中添加它：

```
foo: foo.c foo.h
```

当您的项目变得越来越庞大，您有越来越多自己维护的包含文件时，跟踪所有包含文件以及依赖于它的文件将是一种痛苦。如果更改一个包含文件但忘记重新编译所有依赖于它的文件，结果将是灾难性的。 clang 有一个选项来分析您的文件并生成包含文件及其依赖关系的列表： -MM 。

如果您将此添加到您的 Makefile 中：

```
depend:
	cc -E -MM *.c > .depend
```

并运行 make depend ，文件.depend 将显示一个对象文件列表，C 文件和包含文件：

```
foo.o: foo.c foo.h
```

如果您更改 foo.h，下次运行 make 时，所有依赖于 foo.h 的文件将被重新编译。

每次向文件中添加包含文件时，不要忘记运行 make depend 。

### 2.5.4. FreeBSD Makefiles

编写 Makefiles 可能会相当复杂。幸运的是，像 FreeBSD 这样的基于 BSD 的系统自带一些非常强大的 Makefiles 作为系统的一部分。一个很好的例子是 FreeBSD ports 系统。这是典型 ports Makefile 的基本部分。

```
MASTER_SITES=   ftp://freefall.cdrom.com/pub/FreeBSD/LOCAL_PORTS/
DISTFILES=      scheme-microcode+dist-7.3-freebsd.tgz

.include <bsd.port.mk>
```

现在，如果我们转到此port的目录并键入 make ，将会发生以下情况：

1. 将检查系统上是否已存在此port的源代码。
2. 如果不存在，则将建立到 MASTER_SITES 中 URL 的 FTP 连接以下载源代码。
3. 源代码的校验和将被计算并与已知的良好源代码的校验和进行比较。这是为了确保源代码在传输过程中没有损坏。
4. 适用于 FreeBSD 的源代码所需的任何更改都会被应用-这被称为打补丁。
5. 源代码所需的任何特殊配置都将完成。（许多 UNIX®程序发行版尝试确定它们正在编译的 UNIX®版本以及哪些可选的 UNIX®功能存在-这就是在 FreeBSD ports场景中提供信息的地方）。
6. 编译程序的源代码。实际上，我们转到解压源代码的目录，执行 make - 程序的 make 文件包含构建程序所需的必要信息。
7. 现在我们有了程序的已编译版本。如果希望，现在可以测试它；当对程序感到满意时，可以输入 make install 。这将导致程序及其所需的任何支持文件被复制到正确的位置；还会在 package database 中进行记录，以便稍后可以轻松卸载port，如果我们改变主意的话。

现在我认为您会同意，这对于一个四行脚本来说相当令人印象深刻！

秘密在最后一行，告诉 make 查找名为 bsd.port.mk 的系统 makefile。很容易忽略这一行，但这是所有聪明技巧的来源-有人编写了一个 makefile，告诉 make 执行上述所有操作（还有一些我没有提到的其他操作，包括处理可能发生的任何错误），任何人都可以通过在自己的 makefile 中加入一行来访问它！

如果您想查看这些系统 makefile，它们位于 /usr/share/mk，但最好等到您对 makefile 有了一些实践后再查看，因为它们非常复杂（如果您确实查看它们，请确保您随身携带一瓶浓咖啡！）

### 2.5.5. make 的更高级用法

Make 是一个非常强大的工具，可以做比上面简单示例展示的更多事情。不幸的是，有几个不同版本的 make ，它们之间有很大的区别。了解它们能做什么的最好方法可能是阅读文档-希望这个介绍能为您提供一个基础，让您可以做到这一点。make(1) 手册页面提供了关于变量、参数以及如何使用 make 的全面讨论。

在 ports 中，许多应用程序使用 GNU make，它有一套非常好的 "info" 页面。如果您安装了其中任何一个 ports，GNU make 将自动安装为 gmake 。它也作为一个 port 和独立软件包提供。

要查看 GNU make 的 info 页面，您需要编辑 /usr/local/info 目录中的 dir，以添加一个条目。这涉及添加一行，例如

```
 * Make: (make).                 The GNU Make utility.
```

到文件中。一旦完成此操作，您可以键入 info ，然后从菜单中选择 make（或者在 Emacs 中，执行 C-h i ）。

## 2.6. 调试

### 2.6.1. 可用调试器介绍

使用调试器允许在更受控制的情况下运行程序。通常，可以逐行执行程序，检查变量的值，更改它们，告诉调试器运行到某个特定点然后停止，等等。还可以附加到已经运行的程序，或加载核心文件以调查程序崩溃的原因。

本节旨在快速介绍使用调试器，并不涵盖诸如调试内核之类的专业主题。有关更多信息，请参阅内核调试。

FreeBSD 提供的标准调试器称为 lldb （LLVM 调试器）。由于它是该版本的标准安装的一部分，因此无需采取任何特殊操作即可使用它。它具有良好的命令帮助，可通过 help 命令访问，以及网页教程和文档。

|  | lldb 命令也可以从 ports 或软件包 devel/llvm 中获取。 |
| -- | ------------------------------------------------------ |

FreeBSD 中另一个可用的调试器称为 gdb （GNU 调试器）。与 lldb 不同，它不会默认安装在 FreeBSD 上；要使用它，请从 ports 或软件包中安装 devel/gdb。它具有出色的在线帮助，以及一组信息页面。

这两个调试器具有类似的功能集，因此使用哪一个在很大程度上取决于个人口味。如果只熟悉其中一个，请使用该调试器。对于既不熟悉也不熟悉但想要在 Emacs 中使用一个调试器的人，需要使用 gdb ，因为 Emacs 不支持 lldb 。否则，请尝试两者，看看您更喜欢哪一个。

### 2.6.2. 使用 lldb

#### 2.6.2.1. 启动 lldb

通过输入启动 lldb

```
% lldb -- progname
```

#### 运行带有 lldb 的程序

使用 -g 编译程序，以充分利用 lldb 的功能。没有也可以运行，但只会显示当前运行函数的名称，而不是源代码。如果显示类似以下行：

```
Breakpoint 1: where = temp`main, address = …
```

（在设置断点时没有显示源代码文件名和行号），这意味着程序没有使用 -g 编译。

|  | 大多数 lldb 命令都有可以替代使用的缩写形式。为了清晰起见，这里使用了较长的形式。 |
| -- | ---------------------------------------------------------------------------------- |

在 lldb 提示符下，输入 breakpoint set -n main 。这将告诉调试器不显示正在运行的程序的预备设置代码，并在程序代码的开头停止执行。现在输入 process launch 来实际启动程序- 它将从设置代码的开头开始运行，然后在调用 main() 时被调试器停止。

逐行执行程序，输入 thread step-over 。当程序执行到函数调用时，通过输入 thread step-in 进入该函数调用。在函数调用中，通过输入 thread step-out 返回，或使用 up 和 down 快速查看调用方。

这里是一个简单的示例，展示如何通过 lldb 来发现程序中的错误。 这是我们的程序（有一个故意的错误）：

```
#include <stdio.h>

int bazz(int anint);

main() {
	int i;

	printf("This is my program\n");
	bazz(i);
	return 0;
}

int bazz(int anint) {
	printf("You gave me %d\n", anint);
	return anint;
}
```

该程序将 i 设置为 5 ，并将其传递给一个函数 bazz() ，该函数打印出我们给定的数字。

编译并运行程序会显示

```
% cc -g -o temp temp.c
% ./temp
This is my program
anint = -5360
```

那不是预期的情况！是时候看看发生了什么！

```
% lldb -- temp
(lldb) target create "temp"
Current executable set to 'temp' (x86_64).
(lldb) breakpoint set -n main				Skip the set-up code
Breakpoint 1: where = temp`main + 15 at temp.c:8:2, address = 0x00000000002012ef	lldb puts breakpoint at main()
(lldb) process launch					Run as far as main()
Process 9992 launching
Process 9992 launched: '/home/pauamma/tmp/temp' (x86_64)	Program starts running

Process 9992 stopped
* thread #1, name = 'temp', stop reason = breakpoint 1.1	lldb stops at main()
    frame #0: 0x00000000002012ef temp`main at temp.c:8:2
   5	main() {
   6		int i;
   7
-> 8		printf("This is my program\n");			Indicates the line where it stopped
   9		bazz(i);
   10		return 0;
   11	}
(lldb) thread step-over			Go to next line
This is my program						Program prints out
Process 9992 stopped
* thread #1, name = 'temp', stop reason = step over
    frame #0: 0x0000000000201300 temp`main at temp.c:9:7
   6		int i;
   7
   8		printf("This is my program\n");
-> 9		bazz(i);
   10		return 0;
   11	}
   12
(lldb) thread step-in			step into bazz()
Process 9992 stopped
* thread #1, name = 'temp', stop reason = step in
    frame #0: 0x000000000020132b temp`bazz(anint=-5360) at temp.c:14:29	lldb displays stack frame
   11	}
   12
   13	int bazz(int anint) {
-> 14		printf("You gave me %d\n", anint);
   15		return anint;
   16	}
(lldb)
```

等一下！anint 怎么到 -5360 了？它不是在 main() 中设置为 5 的吗？让我们上到 main() 看看。

```
(lldb) up		Move up call stack
frame #1: 0x000000000020130b temp`main at temp.c:9:2		lldb displays stack frame
   6		int i;
   7
   8		printf("This is my program\n");
-> 9		bazz(i);
   10		return 0;
   11	}
   12
(lldb) frame variable i			Show us the value of i
(int) i = -5360							lldb displays -5360
```

噢！看代码，我们忘记初始化 i 了。我们本意是要放置

```
...
main() {
	int i;

	i = 5;
	printf("This is my program\n");
...
```

但我们遗漏了 i=5; 行。因为我们没有初始化 i，所以当程序运行时，它会使用内存区域中的任意数字，而在这种情况下恰好是 -5360 。

|  | lldb 命令在每次进入或退出函数时显示堆栈帧，即使我们正在使用 up 和 down 在调用堆栈中移动。这显示函数的名称和其参数的值，帮助我们跟踪我们所在的位置和发生的情况。（堆栈是程序存储有关传递给函数的参数以及从函数调用返回时要去哪里的信息的存储区域。） |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### 2.6.2.3. 使用 lldb 检查核心文件

核心文件基本上是一个文件，其中包含进程在崩溃时的完整状态。在“旧日里”，程序员们不得不打印核心文件的十六进制列表，并苦苦钻研机器码手册，但现在生活变得更容易了。顺便说一句，在 FreeBSD 和其他 4.4BSD 系统下，核心文件被称为 progname.core，而不仅仅是 core，以便更清楚地表明核心文件属于哪个程序。

要检查核心文件，请除了程序本身外，还指定核心文件的名称。不要像通常那样启动 lldb ，而是键入 lldb -c<span> </span><em>progname</em>.core --<span> </span><em>progname</em> 。

调试器将显示类似于这样的内容：

```
% lldb -c progname.core -- progname
(lldb) target create "progname" --core "progname.core"
Core file '/home/pauamma/tmp/progname.core' (x86_64) was loaded.
(lldb)
```

在这种情况下，程序被称为 progname，因此核心文件被称为 progname.core。调试器不会显示程序崩溃的原因或位置。为此，请使用 thread backtrace all 。这也会显示程序转储核心的函数是如何被调用的。

```
(lldb) thread backtrace all
 thread #1, name = 'progname', stop reason = signal SIGSEGV
   frame #0: 0x0000000000201347 progname`bazz(anint=5) at temp2.c:17:10
    frame #1: 0x0000000000201312 progname`main at temp2.c:10:2
    frame #2: 0x000000000020110f progname`_start(ap=<unavailable>, cleanup=<unavailable>) at crt1.c:76:7
(lldb)
```

SIGSEGV 表示程序尝试访问内存（通常运行代码或读/写数据）的位置不属于它，但不提供任何具体信息。为此，请查看文件 temp2.c 的第 10 行的源代码，在 bazz() 中。回溯还显示，在这种情况下， bazz() 是从 main() 调用的。

#### 2.6.2.4. 使用 lldb 附加到正在运行的程序

lldb 的一个最棒的功能是它可以附加到已经运行的程序。当然，这需要足够的权限才能这样做。一个常见的问题是在一个分叉的程序中单步执行，并希望跟踪子进程，但调试器只会跟踪父进程。

要做到这一点，启动另一个 lldb ，使用 ps 找到子进程的进程 ID，然后执行

```
(lldb) process attach -p pid
```

在 lldb 中，然后像往常一样进行调试。

要让这个功能正常工作，调用 fork 创建子元素的代码需要像以下这样做（感谢 gdb 信息页面）：

```
...
if ((pid = fork()) < 0)		/* _Always_ check this */
	error();
else if (pid == 0) {		/* child */
	int PauseMode = 1;

	while (PauseMode)
		sleep(10);	/* Wait until someone attaches to us */
	...
} else {			/* parent */
	...
```

现在需要做的只是附加到子元素，将 PauseMode 设置为 0 ，使用 expr PauseMode = 0 等待 sleep() 调用返回。

### 2.6.3. 使用 LLDB 进行远程调试

|  | 描述的功能从 LLDB 版本 12.0.0 开始可用。包含早期 LLDB 版本的 FreeBSD 发行版用户可能希望使用ports中提供的快照或软件包，如 devel/llvm-devel。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------- |

从 LLDB 12.0.0 开始，FreeBSD 支持远程调试。这意味着 lldb-server 可以在一个主机上启动以调试程序，而交互式 lldb 客户端可以从另一个主机连接到它。

要启动新的进程以进行远程调试，请在远程服务器上运行 lldb-server ，然后输入。

```
% lldb-server g host:port -- progname
```

启动后立即停止该进程，并 lldb-server 等待客户端连接。

在本地启动 lldb ，然后输入以下命令以连接到远程服务器：

```
(lldb) gdb-remote host:port
```

lldb-server 也可以连接到正在运行的进程。要执行此操作，请在远程服务器上输入以下内容：

```
% lldb-server g host:port --attach pid-or-name
```

### 2.6.4. 使用 gdb

#### 2.6.4.1. 启动 gdb

通过输入以下内容启动 gdb

```
% gdb progname
```

尽管许多人更喜欢在 Emacs 中运行它。要做到这一点，请输入：

```
 M-x gdb RET progname RET
```

最后，对于那些觉得文本命令提示风格令人反感的人，Ports集合中有一个图形化界面 (devel/xxgdb)可用。

#### 2.6.4.2. 使用 gdb 运行程序

使用 -g 编译程序，以充分利用 gdb 。没有也能运行，但只会显示当前运行函数的名称，而不是源代码。像这样的一行：

```
... (no debugging symbols found) ...
```

当 gdb 启动时，意味着程序未使用 -g 编译。

在 gdb 提示符下，输入 break main 。这将告诉调试器跳过正在运行程序中的预备设置代码，并在程序代码开头停止执行。现在输入 run 启动程序-它将从设置代码开头开始运行，然后在调用 main() 时被调试器停止。

逐行执行程序，按 n 。在函数调用时，按 s 进入函数。在函数调用中，按 f 返回，或使用 up 和 down 快速查看调用者。

这里是一个简单的示例，演示如何使用 gdb 在程序中发现错误。这是我们的程序（带有一个故意的错误）：

```
#include <stdio.h>

int bazz(int anint);

main() {
	int i;

	printf("This is my program\n");
	bazz(i);
	return 0;
}

int bazz(int anint) {
	printf("You gave me %d\n", anint);
	return anint;
}
```

该程序将 i 设置为 5 ，并将其传递给一个函数 bazz() ，该函数打印出我们给它的数字。

编译和运行程序显示

```
% cc -g -o temp temp.c
% ./temp
This is my program
anint = 4231
```

那不是我们所期望的！是时候看看发生了什么！

```
% gdb temp
GDB is free software and you are welcome to distribute copies of it
 under certain conditions; type "show copying" to see the conditions.
There is absolutely no warranty for GDB; type "show warranty" for details.
GDB 4.13 (i386-unknown-freebsd), Copyright 1994 Free Software Foundation, Inc.
(gdb) break main				Skip the set-up code
Breakpoint 1 at 0x160f: file temp.c, line 9.	gdb puts breakpoint at main()
(gdb) run					Run as far as main()
Starting program: /home/james/tmp/temp		Program starts running

Breakpoint 1, main () at temp.c:9		gdb stops at main()
(gdb) n						Go to next line
This is my program				Program prints out
(gdb) s						step into bazz()
bazz (anint=4231) at temp.c:17			gdb displays stack frame
(gdb)
```

等一下！anint 是怎么变成 4231 的？它不是应该设为 5 在 main() 中吗？让我们上到 main() 看一看。

```
(gdb) up					Move up call stack
#1  0x1625 in main () at temp.c:11		gdb displays stack frame
(gdb) p i					Show us the value of i
$1 = 4231					gdb displays 4231
```

哦亲爱的！看着这段代码，我们忘了初始化 i。我们本打算放

```
...
main() {
	int i;

	i = 5;
	printf("This is my program\n");
...
```

但我们遗漏了 i=5; 这一行。由于我们没有初始化 i，在程序运行时，它拥有那个区域内存中的任意数字，在这种情况下恰好是 4231 。

|  | gdb 命令会在每次进入或离开函数时显示栈帧，即使我们正在使用 up 和 down 在调用堆栈中移动。这会显示函数的名称及其参数的值，有助于我们跟踪自己所处的位置以及发生的情况。（栈是程序存储有关传递给函数的参数以及在从函数调用返回时要去哪里的信息的存储区域。） |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### 2.6.4.3. 使用 gdb 检查核心文件

核心文件基本上是一个包含进程在崩溃时完整状态的文件。在“往昔”，程序员必须打印出核心文件的十六进制列表，并苦思冥想于机器码手册，但现在生活变得容易一些。顺便说一句，在 FreeBSD 和其他 4.4BSD 系统下，核心文件称为 progname.core 而不仅仅是 core，以便清楚地表示核心文件属于哪个程序。

要检查一个核心文件，以通常的方式启动 gdb 。而不是输入 break 或 run ，输入

```
(gdb) core progname.core
```

如果核心文件不在当前目录中，请首先键入 dir /path/to/core/file 。

调试器应该显示类似以下内容：

```
% gdb progname
GDB is free software and you are welcome to distribute copies of it
 under certain conditions; type "show copying" to see the conditions.
There is absolutely no warranty for GDB; type "show warranty" for details.
GDB 4.13 (i386-unknown-freebsd), Copyright 1994 Free Software Foundation, Inc.
(gdb) core progname.core
Core was generated by `progname'.
Program terminated with signal 11, Segmentation fault.
Cannot access memory at address 0x7020796d.
#0  0x164a in bazz (anint=0x5) at temp.c:17
(gdb)
```

在这种情况下，程序被称为 progname，因此核心文件称为 progname.core。我们可以看到程序由于尝试访问一个在内存中对它不可用的区域而崩溃，此区域位于一个名为 bazz 的函数中。

有时候，能够查看函数的调用方式是很有用的，因为问题可能出现在复杂程序的调用堆栈中很长的地方。 bt 导致 gdb 打印出调用堆栈的回溯:

```
(gdb) bt
#0  0x164a in bazz (anint=0x5) at temp.c:17
#1  0xefbfd888 in end ()
#2  0x162c in main () at temp.c:11
(gdb)
```

当程序崩溃时，会调用 end() 函数；在这种情况下， bazz() 函数是从 main() 调用的。

#### 2.6.4.4. 通过 gdb 连接到正在运行的程序

gdb 最好的功能之一是它可以附加到已经运行的程序。当然，这需要足够的权限才能这样做。一个常见的问题是在一个分叉并希望跟踪子进程的程序中进行步进，但调试器只会跟踪父进程。

要做到这一点，启动另一个 gdb ，使用 ps 查找子进程的进程 ID，然后执行

```
(gdb) attach pid
```

在 gdb 中，然后像往常一样进行调试。

为了使其正常工作，调用 fork 创建子项的代码需要执行类似以下操作（感谢 gdb 信息页面）：

```
...
if ((pid = fork()) < 0)		/* _Always_ check this */
	error();
else if (pid == 0) {		/* child */
	int PauseMode = 1;

	while (PauseMode)
		sleep(10);	/* Wait until someone attaches to us */
	...
} else {			/* parent */
	...
```

现在所需的只是附加到子项，将 PauseMode 设置为 0 ，并等待 sleep() 调用返回！

## 2.7. 使用 Emacs 作为开发环境

### 2.7.1. Emacs

Emacs 是一个高度可定制的编辑器-事实上，它已经被定制到了一个程度，更像是一个操作系统而不是一个编辑器！许多开发人员和系统管理员实际上几乎所有的时间都在 Emacs 中工作，只是偶尔登出一下。

在这里是不可能甚至仅概括 Emacs 能做的一切的，但下面是一些对开发人员感兴趣的功能：

* 非常强大的编辑器，允许对字符串和正则表达式（模式）进行查找和替换，跳转到块表达式的开始/结束等。
* 下拉菜单和在线帮助。
* 语言相关的语法高亮和缩进。
* 完全定制化。
* 您可以在 Emacs 中编译和调试程序。
* 在编译错误时，您可以跳转到有问题的源代码行。
* 用于阅读 GNU 超文本文档的 info 程序的友好前端，包括 Emacs 本身的文档。
* 友好的 gdb 前端，允许您在逐步执行程序时查看源代码。

肯定还有许多被忽视的。

Emacs 可以使用 editors/emacs port在 FreeBSD 上进行安装。

安装完成后，启动它，按下 C-h t 以阅读 Emacs 教程-这意味着按住 Ctrl 键，按下 h 键，释放 Ctrl 键，然后按下 t 键。(或者，您也可以使用鼠标从帮助菜单中选择 Emacs 教程。)

尽管 Emacs 有菜单，但学习按键绑定是非常值得的，因为在编辑时按下几个键比试图找到鼠标然后点击正确位置要快得多。当您与经验丰富的 Emacs 用户交谈时，您会发现他们经常轻松地抛出像“M-x replace-s RET foo RET bar RET”这样的表达，所以知道它们是什么意思是有用的。而且无论如何，Emacs 有太多有用的功能，以至于它们无法全部适合菜单栏中。

幸运的是，很容易掌握组合键，因为它们显示在菜单项旁边。我的建议是，使用菜单项，比如打开一个文件，直到你理解它是如何工作的并且感到自信，然后尝试按下 C-x C-f。当你对此感到满意时，再转向另一个菜单命令。

如果你记不住特定组合键的作用，可以从帮助菜单中选择描述键，并输入它-Emacs 会告诉你它的作用。你也可以使用命令 Apropos 菜单项来查找所有包含特定单词的命令，以及其旁边的组合键。

顺便说一句，上面的表达意思是按住 Meta 键，按下 x，释放 Meta 键，输入 replace-s （缩写为 replace-string -Emacs 的另一个特性是你可以缩写命令），按下回车键，输入 foo （你想要替换的字符串），再次按下回车键，输入 bar（你想要用 foo 替换的字符串），然后再次按下回车。Emacs 将执行你刚刚请求的搜索和替换操作。

如果您想知道 Meta 究竟是什么，它是许多 UNIX®工作站具有的一个特殊键。不幸的是，PC 没有这样的键，因此通常是 alt 键（或者如果您不幸的话，是 escape 键）。

哦，要退出 Emacs，请执行 C-x C-c （这意味着按住控制键，按下 x 键，按下 c 键，然后释放控制键）。如果有任何未保存的文件打开，Emacs 会询问您是否要保存它们。（忽略文档中说 C-z 是离开 Emacs 的通常方法的部分-那会让 Emacs 在后台运行，并且只有在您使用没有虚拟终端的系统时才真正有用）。

### 2.7.2. 配置 Emacs

Emacs 做了许多美好的事情;其中一些是内置的，一些需要配置。

Emacs 不使用专有的宏语言进行配置，而是使用一种专门为编辑器调整的 Lisp 版本，称为 Emacs Lisp。如果您想继续学习类似 Common Lisp 之类的内容，使用 Emacs Lisp 可以帮助您很多。Emacs Lisp 具有许多 Common Lisp 的特性，尽管它相对较小（因此更容易掌握）。

学习 Emacs Lisp 的最佳途径是阅读在线的 Emacs 参考手册。

然而，不需要实际了解任何 Lisp 就可以开始配置 Emacs，因为我已经包含了一个示例.emacs，这应该足以让您开始。只需将其复制到您的主目录并重新启动 Emacs（如果它已经在运行）；它将从该文件中读取命令，并（希望）为您提供一个有用的基本设置。

### 2.7.3. 一个示例.emacs

不幸的是，在这里有太多内容要详细解释；然而，有一两点值得一提。

* 一切以 ; 开头的内容都是注释，Emacs 会忽略它。
* 在第一行， -<strong>- Emacs-Lisp -</strong>- 是为了让我们可以在 Emacs 中编辑.emacs 本身，并获得用于编辑 Emacs Lisp 的所有花哨功能。Emacs 通常会根据文件名来猜测这一点，对于.emacs 可能猜测不正确。
* 在某些模式下，Tab 键绑定到缩进功能，因此当您按 Tab 键时，它会缩进当前行的代码。如果您想在您正在编写的任何内容中放置一个制表符，请在按 Tab 键时按住 Control 键。
* 该文件通过从文件名猜测语言来支持 C、C++、Perl、Lisp 和 Scheme 的语法高亮显示。
* Emacs 已经有一个预定义的函数称为 next-error 。在编译输出窗口中，这允许您通过执行 M-n 从一个编译错误移动到下一个；我们定义了一个补充函数 previous-error ，它允许您通过执行 M-p 转到上一个错误。最好的功能是 C-c C-c 将打开发生错误的源文件并跳转到适当的行。
* 我们启用了 Emacs 作为服务器的功能，这样，如果您在 Emacs 之外做一些事情并且想要编辑一个文件，您只需输入

  ```
  % emacsclient filename
  ```

  然后您可以在 Emacs 中编辑文件！^[ 6]^

示例 1. 一个示例 .emacs

```
;; -*-Emacs-Lisp-*-

;; This file is designed to be re-evaled; use the variable first-time
;; to avoid any problems with this.
(defvar first-time t
  "Flag signifying this is the first time that .emacs has been evaled")

;; Meta
(global-set-key "\M- " 'set-mark-command)
(global-set-key "\M-\C-h" 'backward-kill-word)
(global-set-key "\M-\C-r" 'query-replace)
(global-set-key "\M-r" 'replace-string)
(global-set-key "\M-g" 'goto-line)
(global-set-key "\M-h" 'help-command)

;; Function keys
(global-set-key [f1] 'manual-entry)
(global-set-key [f2] 'info)
(global-set-key [f3] 'repeat-complex-command)
(global-set-key [f4] 'advertised-undo)
(global-set-key [f5] 'eval-current-buffer)
(global-set-key [f6] 'buffer-menu)
(global-set-key [f7] 'other-window)
(global-set-key [f8] 'find-file)
(global-set-key [f9] 'save-buffer)
(global-set-key [f10] 'next-error)
(global-set-key [f11] 'compile)
(global-set-key [f12] 'grep)
(global-set-key [C-f1] 'compile)
(global-set-key [C-f2] 'grep)
(global-set-key [C-f3] 'next-error)
(global-set-key [C-f4] 'previous-error)
(global-set-key [C-f5] 'display-faces)
(global-set-key [C-f8] 'dired)
(global-set-key [C-f10] 'kill-compilation)

;; Keypad bindings
(global-set-key [up] "\C-p")
(global-set-key [down] "\C-n")
(global-set-key [left] "\C-b")
(global-set-key [right] "\C-f")
(global-set-key [home] "\C-a")
(global-set-key [end] "\C-e")
(global-set-key [prior] "\M-v")
(global-set-key [next] "\C-v")
(global-set-key [C-up] "\M-\C-b")
(global-set-key [C-down] "\M-\C-f")
(global-set-key [C-left] "\M-b")
(global-set-key [C-right] "\M-f")
(global-set-key [C-home] "\M-<")
(global-set-key [C-end] "\M->")
(global-set-key [C-prior] "\M-<")
(global-set-key [C-next] "\M->")

;; Mouse
(global-set-key [mouse-3] 'imenu)

;; Misc
(global-set-key [C-tab] "\C-q\t")	; Control tab quotes a tab.
(setq backup-by-copying-when-mismatch t)

;; Treat 'y' or <CR> as yes, 'n' as no.
(fset 'yes-or-no-p 'y-or-n-p)
(define-key query-replace-map [return] 'act)
(define-key query-replace-map [?\C-m] 'act)

;; Load packages
(require 'desktop)
(require 'tar-mode)

;; Pretty diff mode
(autoload 'ediff-buffers "ediff" "Intelligent Emacs interface to diff" t)
(autoload 'ediff-files "ediff" "Intelligent Emacs interface to diff" t)
(autoload 'ediff-files-remote "ediff"
  "Intelligent Emacs interface to diff")

(if first-time
    (setq auto-mode-alist
	  (append '(("\\.cpp$" . c++-mode)
		    ("\\.hpp$" . c++-mode)
		    ("\\.lsp$" . lisp-mode)
		    ("\\.scm$" . scheme-mode)
		    ("\\.pl$" . perl-mode)
		    ) auto-mode-alist)))

;; Auto font lock mode
(defvar font-lock-auto-mode-list
  (list 'c-mode 'c++-mode 'c++-c-mode 'emacs-lisp-mode 'lisp-mode 'perl-mode 'scheme-mode)
  "List of modes to always start in font-lock-mode")

(defvar font-lock-mode-keyword-alist
  '((c++-c-mode . c-font-lock-keywords)
    (perl-mode . perl-font-lock-keywords))
  "Associations between modes and keywords")

(defun font-lock-auto-mode-select ()
  "Automatically select font-lock-mode if the current major mode is in font-lock-auto-mode-list"
  (if (memq major-mode font-lock-auto-mode-list)
      (progn
	(font-lock-mode t))
    )
  )

(global-set-key [M-f1] 'font-lock-fontify-buffer)

;; New dabbrev stuff
;(require 'new-dabbrev)
(setq dabbrev-always-check-other-buffers t)
(setq dabbrev-abbrev-char-regexp "\\sw\\|\\s_")
(add-hook 'emacs-lisp-mode-hook
	  '(lambda ()
	     (set (make-local-variable 'dabbrev-case-fold-search) nil)
	     (set (make-local-variable 'dabbrev-case-replace) nil)))
(add-hook 'c-mode-hook
	  '(lambda ()
	     (set (make-local-variable 'dabbrev-case-fold-search) nil)
	     (set (make-local-variable 'dabbrev-case-replace) nil)))
(add-hook 'text-mode-hook
	  '(lambda ()
	     (set (make-local-variable 'dabbrev-case-fold-search) t)
	     (set (make-local-variable 'dabbrev-case-replace) t)))

;; C++ and C mode...
(defun my-c++-mode-hook ()
  (setq tab-width 4)
  (define-key c++-mode-map "\C-m" 'reindent-then-newline-and-indent)
  (define-key c++-mode-map "\C-ce" 'c-comment-edit)
  (setq c++-auto-hungry-initial-state 'none)
  (setq c++-delete-function 'backward-delete-char)
  (setq c++-tab-always-indent t)
  (setq c-indent-level 4)
  (setq c-continued-statement-offset 4)
  (setq c++-empty-arglist-indent 4))

(defun my-c-mode-hook ()
  (setq tab-width 4)
  (define-key c-mode-map "\C-m" 'reindent-then-newline-and-indent)
  (define-key c-mode-map "\C-ce" 'c-comment-edit)
  (setq c-auto-hungry-initial-state 'none)
  (setq c-delete-function 'backward-delete-char)
  (setq c-tab-always-indent t)
;; BSD-ish indentation style
  (setq c-indent-level 4)
  (setq c-continued-statement-offset 4)
  (setq c-brace-offset -4)
  (setq c-argdecl-indent 0)
  (setq c-label-offset -4))

;; Perl mode
(defun my-perl-mode-hook ()
  (setq tab-width 4)
  (define-key c++-mode-map "\C-m" 'reindent-then-newline-and-indent)
  (setq perl-indent-level 4)
  (setq perl-continued-statement-offset 4))

;; Scheme mode...
(defun my-scheme-mode-hook ()
  (define-key scheme-mode-map "\C-m" 'reindent-then-newline-and-indent))

;; Emacs-Lisp mode...
(defun my-lisp-mode-hook ()
  (define-key lisp-mode-map "\C-m" 'reindent-then-newline-and-indent)
  (define-key lisp-mode-map "\C-i" 'lisp-indent-line)
  (define-key lisp-mode-map "\C-j" 'eval-print-last-sexp))

;; Add all of the hooks...
(add-hook 'c++-mode-hook 'my-c++-mode-hook)
(add-hook 'c-mode-hook 'my-c-mode-hook)
(add-hook 'scheme-mode-hook 'my-scheme-mode-hook)
(add-hook 'emacs-lisp-mode-hook 'my-lisp-mode-hook)
(add-hook 'lisp-mode-hook 'my-lisp-mode-hook)
(add-hook 'perl-mode-hook 'my-perl-mode-hook)

;; Complement to next-error
(defun previous-error (n)
  "Visit previous compilation error message and corresponding source code."
  (interactive "p")
  (next-error (- n)))

;; Misc...
(transient-mark-mode 1)
(setq mark-even-if-inactive t)
(setq visible-bell nil)
(setq next-line-add-newlines nil)
(setq compile-command "make")
(setq suggest-key-bindings nil)
(put 'eval-expression 'disabled nil)
(put 'narrow-to-region 'disabled nil)
(put 'set-goal-column 'disabled nil)
(if (>= emacs-major-version 21)
	(setq show-trailing-whitespace t))

;; Elisp archive searching
(autoload 'format-lisp-code-directory "lispdir" nil t)
(autoload 'lisp-dir-apropos "lispdir" nil t)
(autoload 'lisp-dir-retrieve "lispdir" nil t)
(autoload 'lisp-dir-verify "lispdir" nil t)

;; Font lock mode
(defun my-make-face (face color &optional bold)
  "Create a face from a color and optionally make it bold"
  (make-face face)
  (copy-face 'default face)
  (set-face-foreground face color)
  (if bold (make-face-bold face))
  )

(if (eq window-system 'x)
    (progn
      (my-make-face 'blue "blue")
      (my-make-face 'red "red")
      (my-make-face 'green "dark green")
      (setq font-lock-comment-face 'blue)
      (setq font-lock-string-face 'bold)
      (setq font-lock-type-face 'bold)
      (setq font-lock-keyword-face 'bold)
      (setq font-lock-function-name-face 'red)
      (setq font-lock-doc-string-face 'green)
      (add-hook 'find-file-hooks 'font-lock-auto-mode-select)

      (setq baud-rate 1000000)
      (global-set-key "\C-cmm" 'menu-bar-mode)
      (global-set-key "\C-cms" 'scroll-bar-mode)
      (global-set-key [backspace] 'backward-delete-char)
					;      (global-set-key [delete] 'delete-char)
      (standard-display-european t)
      (load-library "iso-transl")))

;; X11 or PC using direct screen writes
(if window-system
    (progn
      ;;      (global-set-key [M-f1] 'hilit-repaint-command)
      ;;      (global-set-key [M-f2] [?\C-u M-f1])
      (setq hilit-mode-enable-list
	    '(not text-mode c-mode c++-mode emacs-lisp-mode lisp-mode
		  scheme-mode)
	    hilit-auto-highlight nil
	    hilit-auto-rehighlight 'visible
	    hilit-inhibit-hooks nil
	    hilit-inhibit-rebinding t)
      (require 'hilit19)
      (require 'paren))
  (setq baud-rate 2400)			; For slow serial connections
  )

;; TTY type terminal
(if (and (not window-system)
	 (not (equal system-type 'ms-dos)))
    (progn
      (if first-time
	  (progn
	    (keyboard-translate ?\C-h ?\C-?)
	    (keyboard-translate ?\C-? ?\C-h)))))

;; Under UNIX
(if (not (equal system-type 'ms-dos))
    (progn
      (if first-time
	  (server-start))))

;; Add any face changes here
(add-hook 'term-setup-hook 'my-term-setup-hook)
(defun my-term-setup-hook ()
  (if (eq window-system 'pc)
      (progn
;;	(set-face-background 'default "red")
	)))

;; Restore the "desktop" - do this as late as possible
(if first-time
    (progn
      (desktop-load-default)
      (desktop-read)))

;; Indicate that this file has been read at least once
(setq first-time nil)

;; No need to debug anything now

(setq debug-on-error nil)

;; All done
(message "All done, %s%s" (user-login-name) ".")
```

### 2.7.4. 扩展 Emacs 理解的语言范围

现在，如果您只想在已经适应的语言（C、C++、Perl、Lisp 和 Scheme）中编程，那么这一切都很顺利，但是如果一个名为"whizbang"的新语言出现了，充满了令人兴奋的特性，会发生什么呢？

首先要做的是找出 whizbang 是否附带告诉 Emacs 有关该语言的任何文件。这些文件通常以.el 结尾，缩写为"Emacs Lisp"。例如，如果 whizbang 是一个 FreeBSD port，我们可以通过以下方式找到这些文件

```
% find /usr/ports/lang/whizbang -name "*.el" -print
```

并将它们复制到 Emacs 站点 Lisp 目录中进行安装。在 FreeBSD 上，这个目录是/usr/local/share/emacs/site-lisp。

所以例如，如果来自 find 命令的输出是

```
/usr/ports/lang/whizbang/work/misc/whizbang.el
```

 我们会执行

```
# cp /usr/ports/lang/whizbang/work/misc/whizbang.el /usr/local/share/emacs/site-lisp
```

接下来，我们需要决定 whizbang 源文件具有什么扩展名。假设为了论证的目的，它们都以 .wiz 结尾。我们需要向我们的 .emacs 添加一个条目，以确保 Emacs 能够使用 whizbang.el 中的信息。

在.emacs 中找到 auto-mode-alist 条目，并添加一行 whizbang，例如：

```
...
("\\.lsp$" . lisp-mode)
("\\.wiz$" . whizbang-mode)
("\\.scm$" . scheme-mode)
...
```

这意味着当您编辑以.wiz 结尾的文件时，Emacs 会自动进入 whizbang-mode 模式。

在此下方，您会找到 font-lock-auto-mode-list 条目。像这样将 whizbang-mode 添加到其中：

```
;; Auto font lock mode
(defvar font-lock-auto-mode-list
  (list 'c-mode 'c++-mode 'c++-c-mode 'emacs-lisp-mode 'whizbang-mode 'lisp-mode 'perl-mode 'scheme-mode)
  "List of modes to always start in font-lock-mode")
```

这意味着当编辑.wiz 文件时，Emacs 将始终启用 font-lock-mode （即语法高亮显示）。

这就是所需的全部。如果在打开.wiz 文件时还有其他自动完成的事情，您可以添加 whizbang-mode hook （请参阅 my-scheme-mode-hook 以查看一个添加 auto-indent 的简单示例）。

## 2.8. 更多阅读

有关设置开发环境以贡献 FreeBSD 本身修复的信息，请参阅 development(7)。

* Brian Harvey and Matthew Wright *Simply Scheme* MIT 1994. ISBN 0-262-08226-8
* Randall Schwartz *Learning Perl* O’Reilly 1993 ISBN 1-56592-042-2
* Patrick Henry Winston and Berthold Klaus Paul Horn *Lisp (3rd Edition)*  Addison-Wesley 1989 ISBN 0-201-08319-1
* Brian W. Kernighan and Rob Pike *The Unix Programming Environment* Prentice-Hall 1984 ISBN 0-13-937681-X
* Brian W. Kernighan and Dennis M. Ritchie *The C Programming Language (2nd Edition)*  Prentice-Hall 1988 ISBN 0-13-110362-8
* Bjarne Stroustrup *The C++ Programming Language* Addison-Wesley 1991 ISBN 0-201-53992-6
* W. Richard Stevens *Advanced Programming in the Unix Environment* Addison-Wesley 1992 ISBN 0-201-56317-7
* W. Richard Stevens *Unix Network Programming* Prentice-Hall 1990 ISBN 0-13-949876-1

---

1. 如果您在shell中运行它，可能会导致核心转储。
2. 如果您不知道，二进制排序是一种有效的排序方法，而冒泡排序则不是。
3. 其原因根植于历史的尘埃之中。
4. 注意，我们没有使用 -o 标志来指定可执行文件的名称，因此我们将得到一个名为 a.out 的可执行文件。生成一个名为 foobar 的调试版本留给读者作为练习！
5. 他们不使用 MAKEFILE 形式，因为大写字母通常用于 README 等文档文件。
6. 许多 Emacs 用户将他们的 EDITOR 环境设置为 emacsclient，因此每次需要编辑文件时都会发生这种情况。
