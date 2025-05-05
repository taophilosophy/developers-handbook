# 第 2 章 编程工具

## 2.1. 概要

本章是关于如何使用 FreeBSD 附带的一些编程工具的介绍，尽管其中大部分内容同样适用于许多其他版本的 UNIX®。本章**不会**尝试详细描述编码过程。大多数内容假设读者几乎没有或根本没有编程经验，尽管希望大多数程序员仍能从中找到一些有价值的内容。

## 2.2. 引言

FreeBSD 提供了一个优秀的开发环境。C 和 C++ 编译器以及汇编器随基本系统一同提供，更不用说像 `sed` 和 `awk` 这样的经典 UNIX® 工具了。如果这还不够，Ports 中还有许多其他编译器和解释器可供选择。下一节 [编程简介](https://docs.freebsd.org/en/books/developers-handbook/tools/#tools-programming) 列出了一些可用的选项。FreeBSD 与 POSIX®、ANSI C 等标准以及自身的 BSD 传统高度兼容，因此你可以编写在多种平台上几乎无需修改即可编译和运行的应用程序。

然而，如果你从未在 UNIX® 平台上编写过程序，这些强大工具在最初可能会令人感到困惑。本文档的目标是帮助你快速上手，而不深入涉及高级主题。我们希望本文档能为你提供足够的基础知识，使你能够理解相关文档的内容。

大多数内容几乎不需要任何编程知识，尽管它假设你对 UNIX® 的基本操作已有一定掌握，并且愿意学习！

## 2.3. 编程简介

程序是一组指令，告诉计算机执行各种操作；有时它要执行的指令还取决于之前执行某条指令时发生了什么。本节将概述你可以给出这些指令（通常称为“命令”）的两种主要方式：一种是使用 *解释器*，另一种是使用 *编译器*。由于人类语言对于计算机来说太复杂，难以实现明确理解，因此这些命令通常以专门为此设计的语言编写。

### 2.3.1. 解释器

使用解释器时，语言本身是作为一个环境存在的，你可以在提示符下输入命令，解释器会立即执行这些命令。对于更复杂的程序，你可以把命令写进一个文件，然后让解释器加载并执行这个文件中的命令。如果出错，许多解释器会将你带入调试器，以帮助你查找问题。

这种方式的优点在于你可以立即看到命令的执行结果，错误也可以快速修正。最大的缺点是在你想与他人分享程序时，对方必须拥有相同的解释器，或者你必须以某种方式提供给他们这个解释器，并且他们还需要知道如何使用它。此外，用户在按错键后直接进入调试器的情况，可能也会让人不太舒服。从性能角度来看，解释器通常消耗较多内存，生成的代码效率也不如编译器高。

我认为，如果你从未编程过，解释型语言是最好的入门方式。这类环境典型地见于 Lisp、Smalltalk、Perl 和 Basic 等语言。也有人认为 UNIX® 的 shell（如 `sh`、`csh`）本身就是一种解释器，事实上很多人确实会编写 shell “脚本”来辅助完成他们机器上的各种“维护”任务。实际上，UNIX® 最初的理念之一就是提供许多可以在 shell 脚本中组合使用的小型实用程序，以完成有用的任务。

### 2.3.2. FreeBSD 提供的解释器

下面是一些可以通过 FreeBSD Ports 获取的解释器列表，并简要介绍了一些较为流行的解释型语言。

关于如何获取和安装 Ports 中的应用程序，可以参考手册中的 [Ports 部分](https://docs.freebsd.org/en/books/handbook/#ports-using)。

**BASIC**
BASIC 是 “Beginner’s All-purpose Symbolic Instruction Code”（初学者通用符号指令代码） 的缩写。它在 20 世纪 50 年代被开发出来，用于教授大学生编程；而在 20 世纪 80 年代，每台自重的个人计算机都配备了 BASIC，使它成为许多程序员的第一门编程语言。它也是 Visual Basic 的基础。

Bywater Basic 解释器可以在 FreeBSD 的 Port 中找到，位置是 [lang/bwbasic](https://cgit.freebsd.org/ports/tree/lang/bwbasic/)，Phil Cockroft 编写的 Basic 解释器（原名 Rabbit Basic）则位于 [lang/pbasic](https://cgit.freebsd.org/ports/tree/lang/pbasic/)。

**Lisp**
Lisp 是在 20 世纪 50 年代末期开发的一种语言，用于替代当时流行的“数值计算”语言。它不是基于数字，而是基于列表；事实上，其名称就是 “List Processing”（列表处理） 的缩写。它在人工智能（AI）领域中非常流行。

Lisp 是一种极其强大而复杂的语言，但可能显得庞大且不易掌握。

在 FreeBSD 的 Port 中提供了多种可在 UNIX® 系统上运行的 Lisp 实现。Bruno Haible 和 Michael Stoll 编写的 CLISP 可在 [lang/clisp](https://cgit.freebsd.org/ports/tree/lang/clisp/) 找到；一个更为简化的 Lisp 实现 SLisp 可在 [lang/slisp](https://cgit.freebsd.org/ports/tree/lang/slisp/) 找到。

**Perl**
Perl 在系统管理员中非常流行，用于编写脚本；它也常用于 Web 服务器上编写 CGI 脚本。

Perl 可在 FreeBSD 的 Port 中找到，位置是 [lang/perl5.36](https://cgit.freebsd.org/ports/tree/lang/perl5.36/)，适用于所有 FreeBSD 发行版。

**Scheme**
Scheme 是 Lisp 的方言，它比 Common Lisp 更紧凑、更清晰。在大学中很受欢迎，因为它足够简单，可以作为初学者的第一门语言教学，同时抽象程度也高，适合用于科研工作。

可在 Port 中的 [lang/elk](https://cgit.freebsd.org/ports/tree/lang/elk/) 找到 Elk Scheme 解释器；MIT Scheme 解释器位于 [lang/mit-scheme](https://cgit.freebsd.org/ports/tree/lang/mit-scheme/)，SCM Scheme 解释器则位于 [lang/scm](https://cgit.freebsd.org/ports/tree/lang/scm/)。

**Lua**
Lua 是一种轻量级的可嵌入脚本语言。它具有良好的可移植性，结构也相对简单。Lua 可在 Port 中通过 [lang/lua54](https://cgit.freebsd.org/ports/tree/lang/lua54/) 获取。它也被包含在 base 系统中，路径为 **/usr/libexec/flua**，用于 base 系统组件。第三方软件不应依赖 **flua**。

**Python**
Python 是一种面向对象的解释型语言。它的支持者认为这是一门非常适合初学者的语言，因为它容易上手，但并不像其他用于开发大型复杂应用的解释型语言那样受到限制（Perl 和 Tcl 是另外两种常用于此类开发的语言）。

可在 Port 中通过 [lang/python](https://cgit.freebsd.org/ports/tree/lang/python/) 获取Python 的最新版本。

**Ruby**
Ruby 是一种解释型的纯面向对象编程语言。它因其易于理解的语法、编写灵活性强以及便于开发与维护大型复杂程序而广受欢迎。

可在 Port 中通过 [lang/ruby32](https://cgit.freebsd.org/ports/tree/lang/ruby32/) 获取 Ruby。

**Tcl 和 Tk**
Tcl 是一种可嵌入的解释型语言，因其良好的跨平台特性而得到广泛应用和普及。它既可用于快速编写小型原型应用，也可与 Tk（一个图形用户界面工具包）结合开发功能完备的正式程序。

多个版本的 Tcl 可作为 FreeBSD 的 Port 提供。最新版本 Tcl 8.7 可在 [lang/tcl87](https://cgit.freebsd.org/ports/tree/lang/tcl87/) 找到。

### 2.3.3. 编译器

编译器与解释器有很大不同。首先，你需要使用编辑器在文件中编写代码（一个或多个文件）。然后运行编译器，看看它是否接受你的程序。如果没有编译成功，咬紧牙关，返回编辑器进行修改；如果编译成功并生成了程序，你可以在 shell 命令提示符下运行它，或者在调试器中运行，以查看它是否正常工作。^\[[1](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnotedef_1 "查看脚注。")]^

显然，这种方式不像使用解释器那样直接。然而，它允许你做很多用解释器很难甚至不可能完成的事情，比如编写与操作系统密切交互的代码——甚至编写你自己的操作系统！如果你需要编写高效的代码，它也非常有用，因为编译器可以花时间优化代码，而解释器则无法接受这种优化。此外，为编译器编写的程序通常比为解释器编写的程序更易于分发——你只需要给他们一个可执行文件副本，前提是他们使用的操作系统与你相同。

由于使用单独程序进行编辑-编译-运行-调试的周期相当繁琐，许多商业编译器制造商已经开发了集成开发环境（简称 IDE）。FreeBSD 的基础系统中不包括 IDE，但 [devel/kdevelop](https://cgit.freebsd.org/ports/tree/devel/kdevelop/) 在 Ports Collection 中可以找到，许多人也使用 Emacs 来实现这个目的。关于如何使用 Emacs 作为 IDE，请参见 [Using Emacs as a Development Environment](https://docs.freebsd.org/en/books/developers-handbook/tools/#emacs)。

## 2.4. 使用 `cc` 编译

本节介绍了 FreeBSD 基础系统中安装的 clang 编译器用于 C 和 C++ 的情况。Clang 被安装为 `cc`；GNU 编译器 [gcc](https://cgit.freebsd.org/ports/tree/lang/gcc/) 在 Ports Collection 中也可以找到。使用解释器生成程序的详细过程因解释器而异，通常在解释器的文档和在线帮助中有很好的介绍。

一旦你编写完你的杰作，下一步就是将它转换成能够（希望！）在 FreeBSD 上运行的形式。这通常涉及几个步骤，每个步骤都由一个独立的程序完成。

1. 预处理你的源代码，移除注释并进行其他操作，如在 C 中展开宏。
2. 检查你的代码的语法，查看你是否遵循了语言的规则。如果没有，它会报错！
3. 将源代码转换为汇编语言——这非常接近机器代码，但仍然可以被人理解。据说。
4. 将汇编语言转换为机器代码——是的，我们在谈论的是比特和字节，1 和 0。
5. 检查你是否以一致的方式使用了诸如函数和全局变量之类的东西。例如，如果你调用了一个不存在的函数，它会报错。
6. 如果你试图从多个源代码文件生成可执行文件，计算如何将它们组合在一起。
7. 计算如何生成一个系统的运行时加载器能够加载到内存并运行的程序。
8. 最后，将可执行文件写入文件系统。

“编译”一词通常仅指步骤 1 到 4，其余步骤被称为 *链接*。有时步骤 1 被称为 *预处理*，步骤 3-4 被称为 *汇编*。

幸运的是，几乎所有的细节都被隐藏了，因为 `cc` 是一个前端，它为你管理调用所有这些程序并传递正确的参数；只需输入

```sh
% cc foobar.c
```

就会将 **foobar.c** 按照上述步骤进行编译。如果你有多个文件需要编译，只需像这样操作：

```sh
% cc foo.c bar.c
```

请注意，语法检查只是检查语法。它不会检查你可能犯的任何逻辑错误，比如将程序写入死循环，或者使用了冒泡排序而你本该使用二分排序。^\[[2](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnotedef_2 "查看脚注。")]^

`cc` 有很多选项，都可以在手册页中找到。以下是一些最重要的选项，并附有如何使用它们的示例。

`-o filename`
指定输出文件的名称。如果不使用此选项，`cc` 将生成一个名为 **a.out** 的可执行文件。^\[[3](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnotedef_3 "查看脚注。")]^

```sh
% cc foobar.c               可执行文件是 a.out
% cc -o foobar foobar.c     可执行文件是 foobar
```

`-c`
仅编译文件，不进行链接。对于只想检查语法的简单程序，或使用 **Makefile** 的情况非常有用。

```sh
% cc -c foobar.c
```

这将生成一个名为 **foobar.o** 的 *目标文件*（而不是可执行文件）。可以将该目标文件与其他目标文件一起链接，生成可执行文件。

`-g`
生成可调试版本的可执行文件。这会使编译器将有关源文件和函数调用行的信息添加到可执行文件中。调试器可以利用这些信息，在你单步调试程序时显示源代码，这 *非常* 有用；缺点是这些额外的信息会使程序变得更大。通常，在开发程序时使用 `-g` 编译，而在确认程序正常工作后，则不使用 `-g` 编译“发布版本”。

```sh
% cc -g foobar.c
```

这将生成程序的调试版本。^\[[4](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnotedef_4 "查看脚注。")]^

`-O`
生成优化版本的可执行文件。编译器执行各种巧妙的操作，尽力生成比普通版本运行更快的可执行文件。你可以在 `-O` 后添加一个数字，以指定更高等级的优化，但这往往会暴露编译器优化器中的 bug。

```sh
% cc -O -o foobar foobar.c
```

这将生成优化版的 **foobar**。

以下三个标志会强制 `cc` 检查你的代码是否符合相关的国际标准，通常称为 ANSI 标准，严格来说是 ISO 标准。

`-Wall`
启用 `cc` 作者认为值得启用的所有警告。尽管名称为 “Wall”，但它并不会启用 `cc` 能够生成的所有警告。

`-ansi`
关闭 `cc` 提供的大多数非 ANSI C 特性。尽管名称为 “ansi”，但它并不能严格保证你的代码符合标准。

`-pedantic`
关闭 `cc` 的 *所有* 非 ANSI C 特性。

没有这些标志，`cc` 将允许你使用一些其非标准的扩展功能。这些扩展虽然非常有用，但可能无法与其他编译器兼容——事实上，标准的主要目的之一就是允许人们编写能够在任何编译器和系统上运行的代码。这被称为 *可移植代码*。

通常，你应该尽量使代码具有可移植性，否则你可能需要在以后完全重写程序，以便它能够在其他地方工作——谁知道你几年后会使用什么呢？

```sh
% cc -Wall -ansi -pedantic -o foobar foobar.c
```

这将在检查 **foobar.c** 是否符合标准后生成一个名为 **foobar** 的可执行文件。

`-l <library>`
指定在链接时使用的函数库。

最常见的例子是在编译一个使用 C 中一些数学函数的程序时。与大多数其他平台不同，这些数学函数被放在一个与标准 C 库分开的库中，你需要告诉编译器将其添加进去。

规则是，如果库的名称是 **libsomething.a**，你需要给 `cc` 传递 `-l<something>` 参数。例如，数学库是 **libm.a**，因此你需要传递 `-lm` 给 `cc`。关于数学库的一个常见“陷阱”是，它必须是命令行中最后一个库。

```sh
% cc -o foobar foobar.c -lm
```

这将把数学库的函数链接到 **foobar** 中。

如果你正在编译 C++ 代码，使用 `c++`。在 FreeBSD 上，`c++` 也可以通过 `clang++` 调用。

```sh
% c++ -o foobar foobar.cc
```

这将从 C++ 源文件 **foobar.cc** 生成一个可执行文件 **foobar**。

### 2.4.1. 常见的 `cc` 查询和问题

#### 2.4.1.1. 我编译了一个名为 foobar.c 的文件，但找不到名为 foobar 的可执行文件。它去了哪里？

记住，除非你特别告诉它，否则 `cc` 会将可执行文件命名为 **a.out**。使用 `-o <filename>` 选项：

```sh
% cc -o foobar foobar.c
```

#### 2.4.1.2. 好的，我有一个名为 foobar 的可执行文件，在运行 `ls` 时能看到它，但当我在命令行中输入 foobar 时，告诉我没有这样的文件。为什么它找不到？

与 MS-DOS® 不同，UNIX® 在查找你要运行的可执行文件时，不会自动在当前目录中查找，除非你告诉它。输入 `./foobar`，意思是“运行当前目录下名为 **foobar** 的文件”。

### 2.4.2. 我叫我的可执行文件为 test，但运行时什么也没发生。怎么回事？

大多数 UNIX® 系统都有一个名为 `test` 的程序，它位于 **/usr/bin** 目录，Shell 在检查当前目录之前会先找到它。你可以输入：

```sh
% ./test
```

或者给你的程序取个更好的名字！

#### 2.4.2.1. 我编译了程序，刚开始似乎运行得很好，然后出现了一个错误，说什么“core dumped”。那是什么意思？

*core dump* 这个名字来源于 UNIX® 初期，当时计算机使用核心内存来存储数据。基本上，如果程序在某些条件下失败，系统会将核心内存的内容写入一个名为 **core** 的文件，程序员可以查看该文件以找出问题所在。

#### 2.4.2.2. 很有意思，但我现在该做什么？

使用调试器分析 core 文件（请参见 [调试](https://docs.freebsd.org/en/books/developers-handbook/tools/#debugging)）。

#### 2.4.2.3. 当我的程序发生 core dump 时，它提到一个“segmentation fault”。那是什么意思？

这基本上意味着你的程序尝试对内存执行某种非法操作；UNIX® 设计的目的是保护操作系统和其他程序免受恶意程序的影响。

常见的原因包括：

* 尝试写入 NULL 指针，例如：

  ```c
  char *foo = NULL;
  strcpy(foo, "bang!");
  ```

* 使用未初始化的指针，例如：

  ```c
  char *foo;
  strcpy(foo, "bang!");
  ```

  指针将具有一些随机值，运气好的话，它会指向一个程序无法访问的内存区域，内核会在程序产生任何损害之前终止它。如果运气不好，它可能会指向你程序内部的某个地方，破坏你的数据结构，导致程序神秘地失败。

* 尝试访问数组末尾之外的元素，例如：

  ```c
  int bar[20];
  bar[27] = 6;
  ```

* 尝试存储到只读内存中，例如：

  ```c
  char *foo = "My string";
  strcpy(foo, "bang!");
  ```

  UNIX® 编译器通常会将类似 `"My string"` 的字符串字面量放入只读内存区域。

* 对 `malloc()` 和 `free()` 做不当操作，例如：

  ```c
  char bar[80];
  free(bar);
  ```

  或者

  ```c
  char *foo = malloc(27);
  free(foo);
  free(foo);
  ```

做出这些错误并不总是会导致程序出错，但它们总是糟糕的实践。某些系统和编译器对这些错误的容忍度不同，这就是为什么在一个系统上运行良好的程序，在另一个系统上可能会崩溃的原因。

#### 2.4.2.4. 有时当我得到一个 core dump 时，它说是 bus error。我在 UNIX® 书上看到说这意味着硬件问题，但电脑似乎还在工作。这是真的吗？

不，幸运的是不是（当然，除非你真的遇到了硬件问题…）。这通常是指你以不应有的方式访问了内存。

#### 2.4.2.5. 这个 core dump 的过程看起来很有用，如果我能在需要时使其发生就好了。我可以这样做吗，还是只能等到出错？

是的，你可以这样做，只需去另一个控制台或 xterm，执行

```sh
% ps
```

找出你的程序的进程 ID，然后执行

```sh
% kill -ABRT pid
```

其中 `<pid>` 是你查找的进程 ID。

如果你的程序陷入了死循环，这会很有用。如果程序捕获了 SIGABRT 信号，还有其他一些信号也有类似的效果。

另外，你还可以通过调用 `abort()` 函数，在程序内部创建 core dump。有关更多信息，请参阅 [abort(3)](https://man.freebsd.org/cgi/man.cgi?query=abort&sektion=3&format=html) 的手册页。

如果你想从程序外部创建 core dump，但又不希望程序终止，可以使用 `gcore` 程序。有关更多信息，请参阅 [gcore(1)](https://man.freebsd.org/cgi/man.cgi?query=gcore&sektion=1&format=html) 的手册页。

## 2.5. Make

### 2.5.1. 什么是 `make`？

当你在处理一个简单的程序，只有一两个源文件时，输入

```sh
% cc file1.c file2.c
```

还算可以，但当有多个文件时，输入命令会变得非常繁琐——而且编译可能也会花费很长时间。

解决这个问题的一种方法是使用目标文件，并且只有在源代码发生变化时才重新编译源文件。所以我们可能会像这样：

```sh
% cc file1.o file2.o … file37.c …
```

如果我们只修改了 **file37.c**，而其他文件没有变化，则可以这样做。这样可以加快编译速度，但依然不能解决输入命令的问题。

或者我们可以写一个 shell 脚本来解决输入命令的问题，但它会重新编译所有文件，这在大型项目中非常低效。

如果我们有数百个源文件散布在不同地方呢？如果我们在一个团队中工作，而其他人没有告诉我们他们修改了我们使用的某个源文件怎么办？

也许我们可以将这两种方法结合起来，写一个 shell 脚本，其中包含某种规则，指示何时需要编译源文件。现在，我们需要一个可以理解这些规则的程序，因为这些规则对于 shell 来说有些复杂。

这个程序就是 `make`。它读取一个名为 *makefile* 的文件，文件中指定了不同文件之间的依赖关系，并根据这些规则计算哪些文件需要重新编译，哪些不需要。例如，某个规则可能会说：“如果 **fromboz.o** 比 **fromboz.c** 旧，说明 **fromboz.c** 可能被修改过，所以需要重新编译。”makefile 还会包含告诉 make 如何重新编译源文件的规则，这使它成为一个非常强大的工具。

makefile 通常保存在与其适用的源文件相同的目录中，并且可以命名为 **makefile**、**Makefile** 或 **MAKEFILE**。大多数程序员使用 **Makefile** 这个名字，因为它在目录列表的顶部，更容易被发现。

### 2.5.2. 使用 `make` 的示例

这是一个非常简单的 makefile：

```c
foo: foo.c
	cc -o foo foo.c
```

它由两行组成，一行是依赖关系行，另一行是创建行。

依赖关系行由程序的名称（即 *目标*）组成，后面跟着一个冒号，空格，再跟上源文件的名称。当 `make` 读取这一行时，它会查看 **foo** 是否存在；如果存在，它会比较 **foo** 的最后修改时间和 **foo.c** 的最后修改时间。如果 **foo** 不存在，或者比 **foo.c** 旧，它就会查看创建行，了解该做什么。换句话说，这就是判断 **foo.c** 是否需要重新编译的规则。

创建行以一个制表符开始（按下 tab 键），然后是你在命令行中输入的命令，来创建 **foo**。如果 **foo** 已过期，或者不存在，`make` 就会执行这个命令来创建它。换句话说，这就是告诉 make 如何重新编译 **foo.c** 的规则。

因此，当你输入 `make` 时，`make` 会确保 **foo** 与你对 **foo.c** 的最新更改保持同步。这个原理可以扩展到有数百个目标的 **Makefile**——实际上，在 FreeBSD 上，你只需在合适的目录中输入 `make world` 就可以编译整个操作系统！

makefile 的另一个有用特点是，目标不一定非得是程序。例如，我们可以有一个像这样的 makefile：

```c
foo: foo.c
	cc -o foo foo.c

install:
	cp foo /home/me
```

我们可以通过输入以下命令告诉 `make` 我们想要创建哪个目标：

```
% make target
```

`make` 会只查看该目标并忽略其他目标。例如，如果我们输入 `make foo`，`make` 会忽略 `install` 目标。

如果我们只输入 `make`，`make` 将始终查看第一个目标，并在查看完该目标后停止，而不会查看其他目标。所以如果我们输入 `make`，它会先处理 `foo` 目标，必要时重新编译 **foo**，然后停止，而不会继续处理 `install` 目标。

请注意，`install` 目标实际上并不依赖任何东西！这意味着，当我们输入 `make install` 来制作该目标时，接下来的命令始终会执行。在这种情况下，它会将 **foo** 复制到用户的家目录。这通常在应用程序的 makefile 中使用，以便在程序正确编译后，将应用程序安装到正确的目录中。

这个话题有些难以解释。如果你不完全理解 `make` 是如何工作的，最好的方法是编写一个简单的程序，如 `hello world`，以及像上面那样的 makefile，并进行实验。然后，逐步尝试使用多个源文件，或者让源文件包含一个头文件。`touch` 命令在这里非常有用——它可以更改文件的日期，而不需要编辑它。

### 2.5.3. `make` 和包含文件

C 代码通常以一系列要包含的文件开始，例如 `stdio.h`。其中一些是系统包含文件，有些则是当前项目中的文件：

```
#include <stdio.h>
#include "foo.h"

int main(....
```

为了确保一旦 **foo.h** 被修改，这个文件会立刻重新编译，你需要在 **Makefile** 中添加它：

```
foo: foo.c foo.h
```

当你的项目变大，有越来越多的自定义包含文件时，跟踪所有包含文件及其依赖的文件将变得非常麻烦。如果你修改了一个包含文件，却忘记重新编译所有依赖于它的文件，结果可能会非常糟糕。`clang` 提供了一个选项来分析你的文件并生成包含文件及其依赖关系的列表：`-MM`。

如果你在 **Makefile** 中添加以下内容：

```
depend:
	cc -E -MM *.c > .depend
```

并运行 `make depend`，那么会生成一个 **.depend** 文件，内容包含对象文件、C 文件和包含文件的依赖关系：

```
foo.o: foo.c foo.h
```

如果你修改了 **foo.h**，下次运行 `make` 时，所有依赖于 **foo.h** 的文件都会重新编译。

每次添加包含文件时，别忘了运行 `make depend`。

### 2.5.4. FreeBSD Makefile

编写 Makefile 可能相当复杂。幸运的是，基于 BSD 的系统，如 FreeBSD，提供了一些非常强大的 Makefile，这些文件是系统的一部分。一个很好的例子就是 FreeBSD 的 ports 系统。以下是一个典型的 ports **Makefile** 的核心部分：

```
MASTER_SITES=   ftp://freefall.cdrom.com/pub/FreeBSD/LOCAL_PORTS/
DISTFILES=      scheme-microcode+dist-7.3-freebsd.tgz

.include <bsd.port.mk>
```

现在，如果我们进入该端口的目录并输入 `make`，会发生以下几件事：

1. 系统检查此端口的源代码是否已经存在。
2. 如果不存在，将建立与 MASTER\_SITES 中指定的 URL 的 FTP 连接来下载源代码。
3. 系统计算源代码的校验和，并与已知的源代码校验和进行比较，确保源代码在传输过程中没有损坏。
4. 应用所需的任何更改，使源代码能够在 FreeBSD 上正常工作——这称为 *patching*。
5. 进行源代码所需的特殊配置。（许多 UNIX® 程序在编译时会试图找出它们运行的 UNIX® 版本和所支持的 UNIX® 特性——在 FreeBSD 的 ports 系统中，这些信息会被提供给源代码。）
6. 编译程序的源代码。实际上，我们进入源代码解压的目录并执行 `make`——程序自己的 makefile 已包含构建程序所需的信息。
7. 我们现在得到了编译好的程序。如果需要，可以进行测试；当我们确认程序正常工作时，可以输入 `make install`，这会将程序和任何需要的支持文件复制到正确的位置，并在 `package database` 中创建条目，以便以后如果改变主意，可以轻松卸载该端口。

现在你应该会同意，这个四行的脚本非常强大！

其中的秘密就在于最后一行，它告诉 `make` 查找系统的 makefile 文件 **bsd.port.mk**。这一行很容易被忽视，但正是它包含了所有的巧妙内容——有人编写了一个 makefile，告诉 `make` 执行上述所有操作（包括一些我没有提到的内容，如处理可能发生的错误），任何人只需要在自己的 makefile 中加上这一行，就可以使用这些功能！

如果你想查看这些系统的 makefile，它们位于 **/usr/share/mk**，但最好等你熟悉了 makefile 的使用后再去查看，因为它们非常复杂（如果查看时，记得准备好一瓶浓咖啡！）

### 2.5.5. `make` 的高级用法

`make` 是一个非常强大的工具，能做的事情远远超过上面简单示例所展示的内容。不幸的是，存在多种不同版本的 `make`，它们之间有很大差异。学习它们能做什么的最佳方式可能是阅读文档——希望本介绍已经为你提供了一个良好的基础。可以通过 [make(1)](https://man.freebsd.org/cgi/man.cgi?query=make&sektion=1&format=html) 手册页，了解更多关于变量、参数及如何使用 make 的全面讨论。

许多 ports 应用程序使用 GNU make，它提供了非常好的 "info" 页面。如果你已经安装了这些 ports，GNU make 会自动安装为 `gmake`。它也可以作为一个独立的端口或包安装。

要查看 GNU make 的 info 页面，你需要编辑 **/usr/local/info** 目录下的 **dir** 文件，添加一行：

```sh
* Make: (make).                 The GNU Make utility.
```

添加后，你可以输入 `info` 并从菜单中选择 **make**（或者在 Emacs 中，使用 `C-h i`）。

## 2.6. 调试

### 2.6.1. 可用调试器简介

使用调试器可以在更受控的环境下运行程序。通常，您可以逐行执行程序，检查变量的值，修改变量，指示调试器运行到某个特定位置后停止，等等。还可以附加到一个正在运行的程序，或加载核心文件以调查程序崩溃的原因。

本节旨在提供使用调试器的简要介绍，不涵盖诸如内核调试等专业话题。有关详细信息，请参阅 [内核调试](https://docs.freebsd.org/en/books/developers-handbook/kerneldebug/#kerneldebug)。

FreeBSD 提供的标准调试器是 `lldb`（LLVM 调试器）。由于它是该版本的标准安装的一部分，因此无需做任何特殊的操作即可使用它。它提供了很好的命令帮助，可以通过 `help` 命令访问，还有 [网络教程和文档](https://lldb.llvm.org/)。

>**注意**
>
> 也可以通过 [Ports 或 Packages](https://docs.freebsd.org/en/books/handbook/ports/#ports-using) 从 [devel/llvm](https://cgit.freebsd.org/ports/tree/devel/llvm/) 获取 `lldb` 命令。

FreeBSD 还提供了另一个调试器 `gdb`（GNU 调试器）。与 lldb 不同，`gdb` 并不是 FreeBSD 的默认安装，若要使用它，请从 Ports 或 Packages 中安装 [devel/gdb](https://cgit.freebsd.org/ports/tree/devel/gdb/)。它提供了很好的在线帮助和一套 info 页面。

这两个调试器具有相似的功能集，因此选择使用哪个调试器很大程度上取决于个人喜好。如果只熟悉其中一个，可以使用该调试器。如果对两者都不熟悉，或者都熟悉但希望在 Emacs 中使用其中一个，应该选择 `gdb`，因为 `lldb` 不支持 Emacs。否则，尝试两者并看看哪个更适合自己。

### 2.6.2. 使用 lldb

#### 2.6.2.1. 启动 lldb

通过输入以下命令启动 lldb：

```sh
% lldb -- progname
```

#### 2.6.2.2. 使用 lldb 运行程序

使用 `-g` 编译程序，以便充分利用 `lldb`。即使不加 `-g` 也可以使用，但它将只显示当前正在运行的函数的名称，而不是源代码。如果它显示类似以下的行：

```sh
Breakpoint 1: where = temp`main, address = …
```

（没有源代码文件名和行号的指示）表示程序没有使用 `-g` 编译。

>**注意**
>
> 大多数 `lldb` 命令都有可以替代的简短形式，这里使用了较长的形式以便更清晰。 

在 `lldb` 提示符下，输入 `breakpoint set -n main`。这将告诉调试器不要显示程序运行中的初始设置代码，并在程序的代码开始时停止执行。然后输入 `process launch` 以实际启动程序——它将从初始设置代码开始，然后在调用 `main()` 时被调试器停止。

要逐行执行程序，输入 `thread step-over`。当程序进入函数调用时，输入 `thread step-in` 进入函数。一旦进入函数调用，输入 `thread step-out` 退出函数，或使用 `up` 和 `down` 快速查看调用者。

以下是如何使用 `lldb` 查找程序错误的一个简单示例。我们有一个故意出错的程序：

```c
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

此程序将 `i` 设置为 `5` 并将其传递给函数 `bazz()`，然后打印出我们给它的数字。

编译并运行该程序将显示：

```sh
% cc -g -o temp temp.c
% ./temp
This is my program
anint = -5360
```

这不是预期的结果！该是时候看看发生了什么了！

```sh
% lldb -- temp
(lldb) target create "temp"
Current executable set to 'temp' (x86_64).
(lldb) breakpoint set -n main				跳过设置代码
Breakpoint 1: where = temp`main + 15 at temp.c:8:2, address = 0x00000000002012ef	lldb 设置断点在 main()
(lldb) process launch					运行到 main()
Process 9992 launching
Process 9992 launched: '/home/pauamma/tmp/temp' (x86_64)	程序开始运行

Process 9992 stopped
* thread #1, name = 'temp', stop reason = breakpoint 1.1	lldb 停在 main()
    frame #0: 0x00000000002012ef temp`main at temp.c:8:2
   5	main() {
   6		int i;
   7
-> 8		printf("This is my program\n");			指示停在的行
   9		bazz(i);
   10		return 0;
   11	}
(lldb) thread step-over			执行到下一行
This is my program						程序打印输出
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
(lldb) thread step-in			进入 bazz()
Process 9992 stopped
* thread #1, name = 'temp', stop reason = step in
    frame #0: 0x000000000020132b temp`bazz(anint=-5360) at temp.c:14:29	lldb 显示堆栈帧
   11	}
   12
   13	int bazz(int anint) {
-> 14		printf("You gave me %d\n", anint);
   15		return anint;
   16	}
(lldb)
```

等一下！`anint` 怎么变成了 `-5360`？它不是在 `main()` 中被设置为 `5` 吗？让我们回到 `main()` 看看。

```sh
(lldb) up		返回到调用栈
frame #1: 0x000000000020130b temp`main at temp.c:9:2		lldb 显示堆栈帧
   6		int i;
   7
   8		printf("This is my program\n");
-> 9		bazz(i);
   10		return 0;
   11	}
   12
(lldb) frame variable i			显示 i 的值
(int) i = -5360							lldb 显示 -5360
```

哦，糟糕！看看代码，我们忘记初始化 `i` 了。我们本应写成：

```c
...
main() {
	int i;

	i = 5;
	printf("This is my program\n");
...
```

但我们忘记了 `i=5;` 这一行。由于没有初始化 `i`，它就有了程序运行时该内存位置上的任意值，在这种情况下是 `-5360`。

>**技巧**
>
>每次我们进入或退出一个函数时，`lldb` 命令都会显示堆栈帧，即使我们使用 `up` 和 `down` 移动调用栈时也是如此。这会显示函数的名称和其参数的值，帮助我们跟踪程序的运行情况。（调用栈是程序存储传递给函数的参数和返回时要跳转的位置的存储区域。）


#### 2.6.2.3. 使用 lldb 检查核心文件

核心文件基本上是包含程序崩溃时完整状态的文件。在“好老的日子里”，程序员需要打印出核心文件的十六进制清单，并为机器代码手册而苦苦挣扎，但现在生活变得容易多了。顺便提一下，在 FreeBSD 和其他 4.4BSD 系统中，核心文件被称为 **progname.core**，而不是仅仅叫 **core**，以便更清楚地表明核心文件属于哪个程序。

要检查核心文件，需要在指定程序的同时指定核心文件的名称。不要像通常那样启动 `lldb`，而是输入 `lldb -c <progname>.core -- <progname>`。

调试器将显示如下内容：

```sh
% lldb -c progname.core -- progname
(lldb) target create "progname" --core "progname.core"
Core file '/home/pauamma/tmp/progname.core' (x86_64) was loaded.
(lldb)
```

在这个例子中，程序被命名为 **progname**，因此核心文件名为 **progname.core**。调试器不会显示程序崩溃的原因或位置。为此，可以使用 `thread backtrace all`。这也会显示导致程序崩溃的函数是如何被调用的。

```sh
(lldb) thread backtrace all
​ thread #1, name = 'progname', stop reason = signal SIGSEGV
   frame #0: 0x0000000000201347 progname`bazz(anint=5) at temp2.c:17:10
    frame #1: 0x0000000000201312 progname`main at temp2.c:10:2
    frame #2: 0x000000000020110f progname`_start(ap=<unavailable>, cleanup=<unavailable>) at crt1.c:76:7
(lldb)
```

`SIGSEGV` 表示程序试图访问其不属于自己的内存（通常是运行代码或读写数据），但没有给出具体细节。为此，可以查看 `temp2.c` 文件中第 10 行的源代码，查看 `bazz()` 中的代码。回溯还表明，在此情况下，`bazz()` 是从 `main()` 中调用的。

#### 2.6.2.4. 使用 lldb 附加到正在运行的程序

`lldb` 的一大亮点功能是它可以附加到一个已经在运行的程序上。当然，这要求有足够的权限才能执行此操作。一个常见问题是当程序进行 `fork` 操作并希望跟踪子进程时，调试器通常只会跟踪父进程。

为此，启动另一个 `lldb`，使用 `ps` 查找子进程的进程 ID，然后在 `lldb` 中执行

```sh
(lldb) process attach -p pid
```

然后像往常一样进行调试。

为了让这个过程顺利工作，调用 `fork` 创建子进程的代码需要做如下处理（摘自 `gdb` 的信息页面）：

```c
...
if ((pid = fork()) < 0)		/* _始终_ 检查这个 */
	error();
else if (pid == 0) {		/* 子进程 */
	int PauseMode = 1;

	while (PauseMode)
		sleep(10);	/* 等待直到有人附加到我们 */
	...
} else {			/* 父进程 */
	...
```

现在，只需要附加到子进程，在 `lldb` 中执行 `expr PauseMode = 0`，并等待 `sleep()` 调用返回。

### 2.6.3. 使用 LLDB 进行远程调试

>**注意**
>
> 从 LLDB 版本 12.0.0 开始，支持在 FreeBSD 上进行远程调试。使用早期 LLDB 版本的 FreeBSD 版本的用户可能希望使用 [ports 或 packages](https://docs.freebsd.org/en/books/handbook/#ports-using) 中提供的快照，如 [devel/llvm-devel](https://cgit.freebsd.org/ports/tree/devel/llvm-devel/)。 

从 LLDB 12.0.0 开始，FreeBSD 支持远程调试。这意味着可以在一台主机上启动 `lldb-server` 来调试程序，而交互式的 `lldb` 客户端则可以从另一台主机连接到它。

要启动一个需要远程调试的程序，请在远程服务器上运行 `lldb-server`，命令如下：

```sh
% lldb-server g host:port -- progname
```

程序启动后会立即停止，`lldb-server` 会等待客户端的连接。

然后，在本地启动 `lldb`，并输入以下命令来连接到远程服务器：

```sh
(lldb) gdb-remote host:port
```

`lldb-server` 也可以附加到一个正在运行的进程。要做到这一点，在远程服务器上输入以下命令：

```sh
% lldb-server g host:port --attach pid-or-name
```

### 2.6.4. 使用 gdb

#### 2.6.4.1. 启动 gdb

通过输入以下命令启动 gdb：

```sh
% gdb progname
```

不过许多人更喜欢在 Emacs 中运行它。要在 Emacs 中运行，输入：

```c
M-x gdb RET progname RET
```

最后，对于那些觉得文本命令提示风格不太友好的人，Ports 集合中有一个图形前端（[devel/xxgdb](https://cgit.freebsd.org/ports/tree/devel/xxgdb/)）可以使用。

#### 2.6.4.2. 使用 gdb 运行程序

使用 `-g` 选项编译程序，以便最大限度地发挥 gdb 的功能。即使不加 `-g` 选项，gdb 也能工作，但它只会显示当前运行的函数名称，而不是源代码。如果在启动 gdb 时看到类似以下内容：

```c
... (no debugging symbols found) ...
```

这意味着程序没有使用 `-g` 编译。

在 gdb 提示符下，输入 `break main`。这将告诉调试器跳过程序中的初步设置代码，并在程序代码开始时停止执行。接着输入 `run` 启动程序，程序会从设置代码开始执行，并在调用 `main()` 时被调试器停止。

要逐行调试程序，可以按 `n`。当遇到函数调用时，按 `s` 步入该函数。进入函数后，按 `f` 返回，或者使用 `up` 和 `down` 快速查看调用者。

以下是使用 gdb 找到程序错误的一个简单示例。我们有如下程序（包含一个故意的错误）：

```c
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

该程序将 `i` 设置为 `5`，并将其传递给函数 `bazz()`，该函数打印出我们给它的数字。

编译并运行该程序，输出为：

```sh
% cc -g -o temp temp.c
% ./temp
This is my program
anint = 4231
```

这不是我们期望的结果！是时候看看发生了什么！

```sh
% gdb temp
GDB is free software and you are welcome to distribute copies of it
 under certain conditions; type "show copying" to see the conditions.
There is absolutely no warranty for GDB; type "show warranty" for details.
GDB 4.13 (i386-unknown-freebsd), Copyright 1994 Free Software Foundation, Inc.
(gdb) break main				跳过设置代码
Breakpoint 1 at 0x160f: file temp.c, line 9.	gdb 在 main() 设置断点
(gdb) run					运行到 main()
Starting program: /home/james/tmp/temp		程序开始运行

Breakpoint 1, main () at temp.c:9		gdb 在 main() 停止
(gdb) n						执行下一行
This is my program				程序输出
(gdb) s						步入 bazz()
bazz (anint=4231) at temp.c:17			gdb 显示堆栈帧
(gdb)
```

等一下！`anint` 怎么成了 `4231`？它不是在 `main()` 中被设置为 `5` 吗？让我们回到 `main()`，看看。

```c
(gdb) up					回到调用栈
#1  0x1625 in main () at temp.c:11		gdb 显示堆栈帧
(gdb) p i					显示 i 的值
$1 = 4231					gdb 显示 i 的值是 4231
```

哦，天哪！查看代码，我们忘记初始化 `i` 了。我们本来应该写：

```c
...
main() {
	int i;

	i = 5;
	printf("This is my program\n");
...
```

但我们忘了写 `i=5;` 这一行。由于没有初始化 `i`，它包含了程序运行时该内存区域的任意值，而在这个情况下，恰好是 `4231`。

>**注意**
>
>每次进入或退出一个函数时，`gdb` 命令都会显示堆栈帧，即使我们使用 `up` 和 `down` 来在调用栈中移动。这显示了函数的名称和其参数的值，这有助于我们跟踪当前的位置和发生了什么。（堆栈是程序存储有关传递给函数的参数以及返回时应该去哪里的信息的区域。）


#### 2.6.4.3. 使用 gdb 检查 Core 文件

Core 文件基本上是一个包含程序崩溃时完整状态的文件。在“好久以前”，程序员们不得不打印出 core 文件的十六进制清单，并靠着机器代码手册来调试，但现在生活变得轻松一些。顺便提一下，在 FreeBSD 和其他 4.4BSD 系统中，core 文件被称为 **progname.core**，而不仅仅是 **core**，这样可以更清楚地标明哪个程序的 core 文件。

要检查一个 core 文件，像平常一样启动 `gdb`。不过，不需要输入 `break` 或 `run`，而是输入：

```sh
(gdb) core progname.core
```

如果 core 文件不在当前目录中，首先输入 `dir /path/to/core/file`。

调试器应该会显示如下信息：

```sh
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

在这个例子中，程序名为 **progname**，因此 core 文件名为 **progname.core**。我们可以看到程序因为尝试访问一个无法使用的内存区域而崩溃，崩溃发生在 `bazz` 函数中。

有时查看函数是如何被调用的很有用，因为问题可能出现在复杂程序中的调用栈的更高层。`bt` 命令会让 `gdb` 打印出调用栈的回溯信息：

```sh
(gdb) bt
#0  0x164a in bazz (anint=0x5) at temp.c:17
#1  0xefbfd888 in end ()
#2  0x162c in main () at temp.c:11
(gdb)
```

`end()` 函数在程序崩溃时被调用；在这种情况下，`bazz()` 函数是从 `main()` 被调用的。

#### 2.6.4.4. 附加到正在运行的程序

`gdb` 最酷的功能之一就是它可以附加到一个已经在运行的程序。当然，这需要足够的权限才能做到这一点。一个常见的问题是，在调试一个 fork 的程序时，想要追踪子进程，但调试器只会追踪父进程。

为此，可以启动另一个 `gdb`，使用 `ps` 查找子进程的 PID，然后在 `gdb` 中执行：

```
(gdb) attach pid
```

然后像平常一样调试。

为了让这个过程顺利工作，调用 `fork` 来创建子进程的代码需要像以下这样写（摘自 `gdb` 的信息页面）：

```c
...
if ((pid = fork()) < 0)		/* _Always_ check this */
	error();
else if (pid == 0) {		/* child */
	int PauseMode = 1;

	while (PauseMode)
		sleep(10);	/* 等待直到有人附加到我们 */
	...
} else {			/* parent */
	...
```

现在，只需附加到子进程，将 `PauseMode` 设置为 `0`，并等待 `sleep()` 调用返回即可！

## 2.7. 使用 Emacs 作为开发环境

### 2.7.1. Emacs

Emacs 是一个高度可定制的编辑器——事实上，它已经被定制到几乎像一个操作系统而不是编辑器的程度！许多开发者和系统管理员确实几乎把所有的时间都花在 Emacs 中，只有在注销时才会离开它。

在这里简要总结 Emacs 能做的所有事情几乎是不可能的，但以下是一些对开发者有用的功能：

* 非常强大的编辑器，支持对字符串和正则表达式（模式）进行搜索和替换，跳转到代码块的开始/结束等。
* 下拉菜单和在线帮助。
* 语言相关的语法高亮和缩进。
* 完全可定制。
* 你可以在 Emacs 中编译和调试程序。
* 当编译出错时，你可以跳转到源代码中的错误行。
* 提供一个友好的前端来使用 `info` 程序，阅读 GNU 超文本文档，包括 Emacs 本身的文档。
* 提供一个友好的前端来使用 `gdb`，允许你在程序调试时查看源代码。

当然，还有许多其他功能未被列出。

Emacs 可以通过 FreeBSD 的 [editors/emacs](https://cgit.freebsd.org/ports/tree/editors/emacs/) 端口进行安装。

安装完成后，启动 Emacs，输入 `C-h t` 阅读 Emacs 教程——这意味着按住控制键，按 h 键，松开控制键，然后按 t 键。（或者，你可以使用鼠标从 **Help** 菜单中选择 **Emacs Tutorial**。）

尽管 Emacs 有菜单，但学习键绑定非常值得，因为编辑时按几个键比寻找鼠标并点击正确的地方要快得多。而且，当你与经验丰富的 Emacs 用户交谈时，你会发现他们常常随意地说出像“`M-x replace-s RET foo RET bar RET`”这样的表达方式，所以了解它们的意思很有用。无论如何，Emacs 有太多有用的功能，菜单栏根本容不下所有功能。

幸运的是，学习键绑定非常容易，因为它们会显示在菜单项旁边。我的建议是，首先使用菜单项打开文件，直到你了解它是如何工作的并且对其有信心，然后尝试使用 `C-x C-f`。当你熟悉这个操作后，再尝试其他菜单命令。

如果你记不住某个特定的键组合，可以从 **Help** 菜单中选择 **Describe Key**，然后输入它——Emacs 会告诉你它的功能。你还可以使用 **Command Apropos** 菜单项，查找包含某个特定单词的所有命令，旁边会显示其键绑定。

顺便说一下，前面的表达式意味着按住 Meta 键，按下 x 键，松开 Meta 键，输入 `replace-s`（`replace-string` 的缩写——Emacs 的另一个特点是你可以缩写命令），按回车键，输入 `foo`（你要替换的字符串），按回车键，输入 `bar`（你希望用来替换 `foo` 的字符串），再次按回车。Emacs 会执行你刚刚请求的查找和替换操作。

如果你在想 Meta 键到底是什么，它是许多 UNIX® 工作站上都有的一个特殊键。不幸的是，PC 没有这个键，因此通常使用 alt 键（如果不幸的话，使用 escape 键）。

哦，要退出 Emacs，输入 `C-x C-c`（这意味着按住控制键，按 x 键，按 c 键，释放控制键）。如果你有任何未保存的文件，Emacs 会询问你是否保存它们。（忽略文档中提到的 `C-z` 是退出 Emacs 的常用方式——那样会让 Emacs 在后台挂着，只有在没有虚拟终端的系统上才有用。）

### 2.7.2. 配置 Emacs

Emacs 有许多奇妙的功能；其中一些是内建的，有些则需要配置。

Emacs 并没有使用专有的宏语言来进行配置，而是使用了一种特别为编辑器改编的 Lisp 版本，称为 Emacs Lisp。如果你想学习像 Common Lisp 这样的语言，学习 Emacs Lisp 是非常有帮助的。Emacs Lisp 具有许多 Common Lisp 的特性，尽管它要小得多（因此更容易掌握）。

学习 Emacs Lisp 的最佳方式是阅读在线的 [Emacs Reference](https://www.gnu.org/software/emacs/manual/elisp.html) 手册。

不过，实际上并不需要懂 Lisp 就可以开始配置 Emacs，因为我提供了一个示例 **.emacs** 文件，应该足以让你入门。只需将它复制到你的主目录中，并重新启动 Emacs（如果已经运行的话）；它会读取文件中的命令，并（希望）为你提供一个有用的基本设置。

### 2.7.3. 示例 **.emacs**

不幸的是，这里有太多内容需要详细解释；然而，有一两个值得一提的要点。

* 以 `;` 开头的所有内容都是注释，Emacs 会忽略它们。
* 在第一行，`-<strong>- Emacs-Lisp -</strong>-` 是为了使我们能够在 Emacs 内部编辑 **.emacs** 文件，并获得所有编辑 Emacs Lisp 的高级功能。Emacs 通常会根据文件名尝试猜测这一点，但可能不会为 **.emacs** 文件正确识别。
* Tab 键在某些模式下绑定到缩进功能，因此当你按下 Tab 键时，它会缩进当前的代码行。如果你想在写的内容中插入一个 Tab 字符，可以在按 Tab 键时按住控制键。
* 该文件支持 C、C++、Perl、Lisp 和 Scheme 的语法高亮，通过从文件名猜测语言来启用。
* Emacs 已经有一个预定义的函数 `next-error`。在编译输出窗口中，它允许你通过执行 `M-n` 从一个编译错误跳到下一个；我们定义了一个互补函数 `previous-error`，允许你通过执行 `M-p` 跳转到前一个错误。最好的功能是，`C-c C-c` 会打开发生错误的源文件并跳转到相应的行。
* 我们启用 Emacs 的服务器功能，这样，如果你在 Emacs 之外做一些事情，想要编辑一个文件，只需输入

  ```sh
  % emacsclient filename
  ```

  然后你就可以在 Emacs 中编辑该文件！^\[[6](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnotedef_6 "查看脚注。")]^

**示例 1 .emacs**


```ini
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

### 2.7.4. 扩展 Emacs 支持的语言范围

如果你只想在 **.emacs** 中使用已经支持的语言（C、C++、Perl、Lisp 和 Scheme），那是很好，但如果有一种新的语言叫做 "whizbang" 出现，充满了激动人心的功能，该怎么办呢？

首先要做的是查看 whizbang 是否附带了任何可以让 Emacs 了解该语言的文件。通常这些文件的扩展名是 **.el**，即 "Emacs Lisp" 的缩写。例如，如果 whizbang 是一个 FreeBSD port，我们可以通过以下命令来查找这些文件：

```sh
% find /usr/ports/lang/whizbang -name "*.el" -print
```

然后将这些文件复制到 Emacs 的 site-lisp 目录中进行安装。在 FreeBSD 中，site-lisp 目录是 **/usr/local/share/emacs/site-lisp**。

例如，如果 find 命令的输出是：

```sh
/usr/ports/lang/whizbang/work/misc/whizbang.el
```

那么我们应该执行：

```sh
# cp /usr/ports/lang/whizbang/work/misc/whizbang.el /usr/local/share/emacs/site-lisp
```

接下来，我们需要决定 whizbang 源文件的扩展名是什么。假设它们都以 **.wiz** 结尾。我们需要在 **.emacs** 中添加一条记录，确保 Emacs 能够使用 **whizbang.el** 中的信息。

找到 **.emacs** 中的 `auto-mode-alist` 条目，然后添加一行，如下所示：

```sh
...
("\\.lsp$" . lisp-mode)
("\\.wiz$" . whizbang-mode)
("\\.scm$" . scheme-mode)
...
```

这意味着当你编辑一个以 **.wiz** 结尾的文件时，Emacs 会自动进入 `whizbang-mode`。

接下来，在 **.emacs** 中找到 `font-lock-auto-mode-list` 条目。像这样将 `whizbang-mode` 添加到其中：

```
;; 自动字体锁定模式
(defvar font-lock-auto-mode-list
  (list 'c-mode 'c++-mode 'c++-c-mode 'emacs-lisp-mode 'whizbang-mode 'lisp-mode 'perl-mode 'scheme-mode)
  "始终启用字体锁定模式的模式列表")
```

这意味着当编辑一个 **.wiz** 文件时，Emacs 会始终启用 `font-lock-mode`（即语法高亮）。

就这样，完成了所有必要的设置。如果你希望在打开 **.wiz** 文件时自动执行其他操作，可以添加一个 `whizbang-mode hook`（参见 `my-scheme-mode-hook`，这是一个简单的例子，添加了 `auto-indent`）。

## 2.8. 进一步阅读

关于如何设置开发环境以便为 FreeBSD 本身贡献修复，请参阅 [development(7)](https://man.freebsd.org/cgi/man.cgi?query=development&sektion=7&format=html)。

* Brian Harvey 和 Matthew Wright *Simply Scheme* MIT 1994. ISBN 0-262-08226-8
* Randall Schwartz *Learning Perl* O’Reilly 1993 ISBN 1-56592-042-2
* Patrick Henry Winston 和 Berthold Klaus Paul Horn *Lisp (3rd Edition)* Addison-Wesley 1989 ISBN 0-201-08319-1
* Brian W. Kernighan 和 Rob Pike *The Unix Programming Environment* Prentice-Hall 1984 ISBN 0-13-937681-X
* Brian W. Kernighan 和 Dennis M. Ritchie *The C Programming Language (2nd Edition)* Prentice-Hall 1988 ISBN 0-13-110362-8
* Bjarne Stroustrup *The C++ Programming Language* Addison-Wesley 1991 ISBN 0-201-53992-6
* W. Richard Stevens *Advanced Programming in the Unix Environment* Addison-Wesley 1992 ISBN 0-201-56317-7
* W. Richard Stevens *Unix Network Programming* Prentice-Hall 1990 ISBN 0-13-949876-1

---

[1](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnoteref_1). 如果你在 shell 中运行它，可能会得到核心转储。

[2](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnoteref_2). 如果你不知道，二进制排序是一种高效的排序方式，而冒泡排序则不是。

[3](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnoteref_3). 这背后的原因深藏在历史的迷雾中。

[4](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnoteref_4). 请注意，我们没有使用 -o 标志来指定可执行文件名，所以我们将得到一个名为 a.out 的可执行文件。生成一个名为 foobar 的调试版本留给读者自己完成！

[5](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnoteref_5). 它们不使用 MAKEFILE 格式，因为大写字母通常用于文档文件，比如 README。

[6](https://docs.freebsd.org/en/books/developers-handbook/tools/#_footnoteref_6). 许多 Emacs 用户将他们的 EDITOR 环境设置为 emacsclient，这样每当他们需要编辑文件时，Emacs 就会启动。
