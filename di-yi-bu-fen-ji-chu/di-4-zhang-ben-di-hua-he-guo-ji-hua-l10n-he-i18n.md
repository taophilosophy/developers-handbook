# 第 4 章 本地化和国际化——L10N 和 I18N

## 4.1. 编写符合 I18N 标准的程序

为了让您的程序对其他语言用户更加实用，我们希望您能够按照 I18N 标准进行编程。GNU 的 gcc 编译器和像 QT、GTK 这样的图形界面库通过对字符串的特殊处理来支持 I18N。编写符合 I18N 的程序非常简单，这也使得其他人可以迅速将您的程序移植到其他语言。请参考特定库的 I18N 文档以获取更多细节。

与普遍认知相反，编写符合 I18N 的代码其实很容易。通常这只是将您的字符串包装在特定库的函数中。此外，请确保支持宽字符或多字节字符。

### 4.1.1. 统一 I18N 努力的呼声

我们注意到，各个国家的 I18N/L10N 工作经常在重复彼此的劳动。我们中很多人一次又一次地在低效地重复造轮子。我们希望 I18N 领域的主要群体能聚集起来，形成类似 Core Team 所负责的那种统一协作。

当前，我们希望您在编写或移植 I18N 程序时，能将其发送到每个国家相关的 FreeBSD 邮件列表中进行测试。未来，我们希望创建出无需任何脏补丁即可在所有语言中工作的应用程序。

已经建立 [FreeBSD 国际化邮件列表](https://lists.freebsd.org/subscription/freebsd-i18n)。如果您是 I18N/L10N 的开发者，请将您的评论、想法、问题或您认为相关的一切内容发送到该邮件列表。

### 4.1.2. Perl 与 Python

Perl 和 Python 拥有 I18N 和宽字符处理库。请在进行 I18N 编程时使用它们。

## 4.2. 使用 POSIX.1 本地语言支持（NLS）实现消息本地化

在支持各种输入编码和国家惯例（如不同的小数点符号）这些基础 I18N 功能之上，更高级的 I18N 还可以对程序输出的消息进行本地化。一种常见的做法是使用 POSIX.1 NLS 函数，这些函数作为 FreeBSD 基本系统的一部分提供。

### 4.2.1. 将本地化消息组织成目录文件

POSIX.1 NLS 基于目录文件（catalog file），这些文件使用所需编码格式包含本地化的消息。消息被组织成若干集合，每条消息在其所属集合中通过一个整数编号标识。目录文件的命名惯例是使用包含的语言环境名，加上 `.msg` 扩展名。例如，ISO8859-2 编码的匈牙利语消息应该存储在名为 **hu\_HU.ISO8859-2** 的文件中。

这些目录文件是普通文本文件，包含带编号的消息。可以通过在行首添加 `$` 符号写注释。集合边界也通过特殊注释分隔，其中 `set` 关键字必须紧跟在 `$` 符号之后，然后是集合编号。例如：

```sh
$set 1
```

实际的消息条目以消息编号开头，后跟本地化的消息。支持 [printf(3)](https://man.freebsd.org/cgi/man.cgi?query=printf&sektion=3&format=html) 中常见的格式修饰符：

```sh
15 "File not found: %s\n"
```

语言目录文件在程序使用之前必须编译成二进制格式。这个转换通过 [gencat(1)](https://man.freebsd.org/cgi/man.cgi?query=gencat&sektion=1&format=html) 工具完成。它的第一个参数是生成的目录文件名，后续参数为输入的目录文件。也可以将本地化消息组织成多个目录文件，然后通过 [gencat(1)](https://man.freebsd.org/cgi/man.cgi?query=gencat&sektion=1&format=html) 一起处理。

### 4.2.2. 在源代码中使用目录文件

使用这些目录文件很简单。为了使用相关函数，必须包含头文件 **nl\_types.h**。在使用目录前，需使用 [catopen(3)](https://man.freebsd.org/cgi/man.cgi?query=catopen&sektion=3&format=html) 打开目录文件。该函数接受两个参数，第一个是安装并编译后的目录文件名，通常使用程序名（如 grep），此名将用于查找编译目录文件。[catopen(3)](https://man.freebsd.org/cgi/man.cgi?query=catopen&sektion=3&format=html) 会在 **/usr/share/nls/locale/catname** 和 **/usr/local/share/nls/locale/catname** 中查找此文件，其中 `locale` 是设置的语言环境，`catname` 是目录名。

第二个参数是一个常量，有两个取值：

* `NL_CAT_LOCALE`，表示目录文件基于 `LC_MESSAGES`。
* `0`，表示使用 `LANG` 环境变量查找目录文件。

[catopen(3)](https://man.freebsd.org/cgi/man.cgi?query=catopen&sektion=3&format=html) 返回一个类型为 `nl_catd` 的目录标识符。可能的返回错误代码请参阅其手册页。

打开目录文件后，可以使用 [catgets(3)](https://man.freebsd.org/cgi/man.cgi?query=catgets&sektion=3&format=html) 来检索消息。第一个参数是 `catopen()` 返回的目录标识符，第二个参数是集合编号，第三个是消息编号，第四个是备用消息，当无法从目录文件中取出所请求消息时将返回此备用消息。

使用完目录文件后，必须调用 [catclose(3)](https://man.freebsd.org/cgi/man.cgi?query=catclose&sektion=3&format=html) 关闭该文件，此函数有一个参数，即目录标识符。

### 4.2.3. 一个实际示例

以下示例展示了如何以灵活方式使用 NLS 目录。

首先，将以下代码行放入程序的公共头文件中，并在所有需要本地化消息的源文件中包含该头文件：

```c
#ifdef WITHOUT_NLS
#define getstr(n)	 nlsstr[n]
#else
#include nl_types.h

extern nl_catd		 catalog;
#define getstr(n)	 catgets(catalog, 1, n, nlsstr[n])
#endif

extern char		*nlsstr[];
```

接着，将以下代码放入主源文件的全局声明部分：

```c
#ifndef WITHOUT_NLS
#include nl_types.h
nl_catd	 catalog;
#endif

/*
 * 当禁用 NLS 或找不到目录文件时，使用的默认消息。
 */
char    *nlsstr[] = {
        "",
/* 1*/  "some random message",
/* 2*/  "some other message"
};
```

最后是打开、读取和关闭目录文件的实际代码片段：

```c
#ifndef WITHOUT_NLS
	catalog = catopen("myapp", NL_CAT_LOCALE);
#endif

...

printf(getstr(1));

...

#ifndef WITHOUT_NLS
	catclose(catalog);
#endif
```

#### 4.2.3.1. 减少需要本地化的字符串数量

有一个很好的方法可以减少需要本地化的字符串数量，就是使用 libc 的错误信息。这也有助于避免重复，并为许多程序可能遇到的常见错误提供一致的错误信息。

首先，下面是一个没有使用 libc 错误信息的例子：

```c
#include err.h
...
if (!S_ISDIR(st.st_mode))
	errx(1, "argument is not a directory");
```

这个例子可以改写为通过读取 `errno` 并据此打印错误信息来输出错误：

```c
#include err.h
#include errno.h
...
if (!S_ISDIR(st.st_mode)) {
	errno = ENOTDIR;
	err(1, NULL);
}
```

在这个例子中，自定义字符串被省略了，因此翻译人员在本地化程序时的工作量会更少，用户在遇到这个错误时将看到熟悉的 “Not a directory” 错误信息。这条信息对他们来说可能更为熟悉。请注意，为了直接访问 `errno`，必须包含 **errno.h**。

值得注意的是，有些情况下 `errno` 会由前面的调用自动设置，因此不需要显式设置：

```c
#include err.h
...
if ((p = malloc(size)) == NULL)
	err(1, NULL);
```

### 4.2.4. 使用 **bsd.nls.mk**

使用目录文件需要一些重复的步骤，例如编译目录文件并将其安装到正确的位置。为了进一步简化这一过程，**bsd.nls.mk** 引入了一些宏。无需显式包含 **bsd.nls.mk**，它会由常见的 Makefile（例如 **bsd.prog.mk** 或 **bsd.lib.mk**）自动包含进来。

通常，只需定义 `NLSNAME`（应与 [catopen(3)](https://man.freebsd.org/cgi/man.cgi?query=catopen&sektion=3&format=html) 第一个参数中提到的目录名一致），并在 `NLS` 中列出目录文件名（不带 `.msg` 扩展名）即可。下面是一个示例，它使得可以在结合前面的代码示例时禁用 NLS。若要构建不支持 NLS 的程序，只需定义 `WITHOUT_NLS` [make(1)](https://man.freebsd.org/cgi/man.cgi?query=make&sektion=1&format=html) 变量即可。

```c
.if !defined(WITHOUT_NLS)
NLS=	es_ES.ISO8859-1
NLS+=	hu_HU.ISO8859-2
NLS+=	pt_BR.ISO8859-1
.else
CFLAGS+=	-DWITHOUT_NLS
.endif
```

通常，目录文件放在 **nls** 子目录下，这是 **bsd.nls.mk** 的默认行为。不过，也可以通过设置 `NLSSRCDIR` [make(1)](https://man.freebsd.org/cgi/man.cgi?query=make&sektion=1&format=html) 变量来覆盖目录文件的位置。预编译的目录文件的默认命名方式也遵循前面提到的命名规范，但也可以通过设置 `NLSNAME` 变量来覆盖。还有一些其它选项可以微调目录文件的处理过程，不过通常不需要，因此此处未作介绍。要了解 **bsd.nls.mk** 的更多信息，请直接查阅该文件，它很简短，也很容易理解。
