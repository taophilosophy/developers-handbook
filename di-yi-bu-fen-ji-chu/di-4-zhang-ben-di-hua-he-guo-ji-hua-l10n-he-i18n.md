# 第 4 章 本地化和国际化——L10N 和 I18N

## 4.1. 编写符合 I18N 标准的应用程序

为了使您的应用程序对其他语言的使用者更有用，我们希望您编写符合 I18N 的代码。GNU gcc 编译器和诸如 QT 和 GTK 之类的 GUI 库通过对字符串的特殊处理来支持 I18N。使程序符合 I18N 非常容易。它允许贡献者快速地将您的应用程序翻译成其他语言。有关更多详细信息，请参考特定库的 I18N 文档。

与普遍看法相反，符合 I18N 的代码很容易编写。通常，只需要使用特定库函数包装您的字符串即可。此外，请确保支持宽字符或多字节字符。

### 4.1.1. 统一 I18N 工作的呼吁

我们注意到，每个国家的个别 I18N/L10N 工作一直在重复彼此的努力。我们许多人一再且低效地重复造轮子。我们希望 I18N 中的各个主要团体能够聚集到一个类似核心团队责任的团体努力中。

目前，我们希望当您编写或 port I18N 程序时，您可以将其发送到每个国家相关的 FreeBSD 邮件列表进行测试。未来，我们希望创建能够在所有语言中即插即用而无需肮脏的黑客技巧的应用程序。

已建立 FreeBSD 国际化邮件列表。如果您是 I18N/L10N 开发人员，请发送您的评论、想法、问题以及您认为与之相关的任何内容。

### 4.1.2. Perl 和 Python

Perl 和 Python 都具有 I18N 和宽字符处理库。请使用它们来实现 I18N 合规性。

## 4.2. 具有 POSIX.1 本地化消息和本地语言支持（NLS）

超越基本的 I18N 功能，比如支持各种输入编码或支持国家约定，比如不同的十进制分隔符，在更高级别的 I18N 中，可以将各种程序写入输出的消息本地化。 做到这一点的常见方法是使用 POSIX.1 NLS 功能，这些功能作为 FreeBSD 基本系统的一部分提供。

### 4.2.1. 将本地化消息组织成目录文件

POSIX.1 NLS 基于目录文件，其中包含所需编码的本地化消息。 消息被组织成集合，每条消息在所包含集合中都用整数编号标识。 目录文件通常以包含本地化消息的区域设置命名，后跟 .msg 扩展名。 例如，ISO8859-2 编码的匈牙利消息应存储在名为 hu_HU.ISO8859-2 的文件中。

这些目录文件是常见的文本文件，包含编号的消息。可以通过在行首使用 $ 符号来编写注释。集合边界也由特殊注释分隔，其中关键字 set 必须直接跟在 $ 符号后面。然后 set 关键字跟着集合编号。例如：

```
$set 1
```

实际消息条目以消息编号开头，后跟本地化消息。来自 printf(3)的著名修饰符会被接受：

```
15 "File not found: %s\n"
```

语言目录文件在能够从程序中打开之前必须被编译成二进制形式。这种转换通过 gencat(1)实用程序完成。它的第一个参数是编译后目录的文件名，后续参数是输入目录。本地化消息也可以组织成多个目录文件，然后所有这些文件都可以通过 gencat(1)处理。

### 4.2.2. 使用源代码中的目录文件

使用目录文件很简单。要使用相关函数，必须包含 nl_types.h。在使用目录之前，必须使用 catopen(3) 打开它。该函数接受两个参数。第一个参数是已安装和编译的目录的名称。通常使用程序的名称，例如 grep。在查找已编译的目录文件时将使用此名称。catopen(3) 调用在 /usr/share/nls/locale/catname 和 /usr/local/share/nls/locale/catname 中查找此文件，其中 locale 是区域设置， catname 是正在讨论的目录名称。第二个参数是一个常量，可以有两个值：

* NL_CAT_LOCALE ，这意味着所使用的目录文件将基于 LC_MESSAGES 。
* 0 ，这意味着必须使用 LANG 打开正确的目录。

catopen(3) 调用返回一个 nl_catd 类型的目录标识符。请参阅手册页以获取可能返回的错误代码列表。

打开目录后，catgets(3) 可用于检索消息。第一个参数是 catopen(3) 返回的目录标识符，第二个是集编号，第三个是消息编号，第四个是回退消息，如果无法从目录文件中检索到请求的消息，将返回该回退消息。

使用目录文件后，必须通过调用 catclose(3)来关闭它，catclose(3)有一个参数，即目录 id。

### 4.2.3. 一个实际示例

以下示例将演示如何以灵活的方式使用 NLS 目录的简单解决方案。

下面的行需要放入程序的公共头文件中，该头文件包含所有需要本地化消息的源文件中：

```
#ifdef WITHOUT_NLS
#define getstr(n)	 nlsstr[n]
#else
#include nl_types.h

extern nl_catd		 catalog;
#define getstr(n)	 catgets(catalog, 1, n, nlsstr[n])
#endif

extern char		*nlsstr[];
```

接下来，将这些行放入主源文件的全局声明部分：

```
#ifndef WITHOUT_NLS
#include nl_types.h
nl_catd	 catalog;
#endif

/*
 * Default messages to use when NLS is disabled or no catalog
 * is found.
 */
char    *nlsstr[] = {
        "",
/* 1*/  "some random message",
/* 2*/  "some other message"
};
```

接下来是真正的代码片段，打开，读取和关闭目录：

```
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

#### 将字符串减少至本地化

通过使用 libc 错误消息，可以很好地减少需要本地化的字符串。这对于避免重复并为可能遇到的许多程序的常见错误提供一致的错误消息也非常有用。

首先，这里有一个不使用 libc 错误消息的示例：

```
#include err.h
...
if (!S_ISDIR(st.st_mode))
	errx(1, "argument is not a directory");
```

这可以通过阅读 errno 并相应地打印错误消息来转换为打印错误消息：

```
#include err.h
#include errno.h
...
if (!S_ISDIR(st.st_mode)) {
	errno = ENOTDIR;
	err(1, NULL);
}
```

在这个例子中，自定义字符串被消除，因此在本地化程序时，翻译人员的工作量会减少，当用户遇到此错误时，他们将看到通常的"不是目录"错误消息。这个消息对他们来说可能更为熟悉。请注意，为了直接访问 errno ，有必要包含 errno.h。

值得注意的是，有些情况下， errno 是由前面的调用自动设置的，因此不需要显式设置它：

```
#include err.h
...
if ((p = malloc(size)) == NULL)
	err(1, NULL);
```

### 利用 bsd.nls.mk

使用目录文件需要一些重复的步骤，比如编译目录文件并将其安装到适当的位置。为了进一步简化这个过程，bsd.nls.mk 引入了一些宏。不需要显式包含 bsd.nls.mk，它是从通用的 Makefiles 中引入的，比如 bsd.prog.mk 或 bsd.lib.mk。

通常只需要定义 NLSNAME ，其中应该将作为 catopen(3) 第一个参数提到的目录名称列出，并在 NLS 中列出目录文件，但不包括它们的 .msg 扩展名。这里有一个示例，使得在之前的代码示例中与 NLS 一起使用时可以禁用 NLS。必须定义 WITHOUT_NLS make(1) 变量，以便在没有 NLS 支持的情况下构建程序。

```
.if !defined(WITHOUT_NLS)
NLS=	es_ES.ISO8859-1
NLS+=	hu_HU.ISO8859-2
NLS+=	pt_BR.ISO8859-1
.else
CFLAGS+=	-DWITHOUT_NLS
.endif
```

传统上，目录文件放置在 nls 子目录下，这是 bsd.nls.mk 的默认行为。但是，可以通过 NLSSRCDIR make(1)变量覆盖目录的位置。预编译目录文件的默认名称也遵循之前提到的命名约定。可以通过设置 NLSNAME 变量来覆盖它。还有其他选项可以微调目录文件的处理，但通常不需要，因此这里不进行描述。有关 bsd.nls.mk 的更多信息，请参考文件本身，它简短易懂。
