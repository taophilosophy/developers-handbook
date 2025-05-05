# 第 5 章. 源代码树准则与规范


该章节记录了 FreeBSD 源码树中施行的各种指南和政策。

## 5.1. 编码风格指南

一致的编码风格极为重要，尤其是在 FreeBSD 这样的大型项目中。代码应遵循 FreeBSD 编码风格，如 [style(9)](https://man.freebsd.org/cgi/man.cgi?query=style&sektion=9&format=html) 和 [style.Makefile(5)](https://man.freebsd.org/cgi/man.cgi?query=style.Makefile&sektion=5&format=html) 所述。

## 5.2. Makefile 中的 `MAINTAINER`

如果 FreeBSD **src/** 分发中的某个部分由某个人或一组人维护，这会通过 **src/MAINTAINERS** 文件中的一条记录来表达。Ports Collection 中某个 Port 的维护者通过在该 Port 的 **Makefile** 中添加一行 `MAINTAINER` 来向外界表明其维护权：

```sh
MAINTAINER= email-addresses
```

>**技巧**
>
> 对于仓库中的其他部分，或没有指定维护者的部分，或当你不确定谁是活跃的维护者时，可以尝试查看源码树相关部分的最近提交历史。很多时候，并没有明确指定某个维护者，但过去几年中持续活跃于某个源码树部分的人通常也愿意审阅更改。即使文档或源码中没有特别说明，请求审阅作为一种礼貌行为也是非常合理的。 

维护者的职责如下：

* 维护者拥有该代码的所有权，并对其负责。这意味着他/她需要修复与该部分代码相关的 bug 并回应问题报告；如果是引入的第三方软件，还需按需跟踪新版本。
* 如果某个目录指定了维护者，那么对该目录的更改在提交前应送审给维护者。只有在多次发送邮件仍长时间未收到回复时，才可在未经审阅的情况下提交。但建议尽可能还是请其他人进行审阅。
* 当然，未经同意不能将某个人或团队添加为维护者。另一方面，维护者不必是 committer，也可以是一个团队。

## 5.3. 引入的软件（Contributed Software）

FreeBSD 的一部分分发内容是由 FreeBSD 项目之外的团队维护的。出于历史原因，我们称这类软件为 *引入的软件（contributed software）*。典型例子有 LLVM、[zlib(3)](https://man.freebsd.org/cgi/man.cgi?query=zlib&sektion=3&format=html) 和 [awk(1)](https://man.freebsd.org/cgi/man.cgi?query=awk&sektion=1&format=html)。

管理引入软件的标准做法是创建一个 *vendor 分支*，在该分支中可以以“干净”方式导入软件（即不加修改），并以版本控制的方式跟踪更新。然后将 vendor 分支中的内容应用到源码树中，并可进行本地修改。FreeBSD 专属的构建集成代码应保存在源码树中，而非 vendor 分支中。

根据具体需求与复杂程度，个别软件项目可以在维护者酌情判断下偏离此流程。更新特定引入软件所需的具体步骤应记录在名为 `FREEBSD-upgrade` 的文件中，例如 [libarchive 的 FREEBSD-upgrade 文件](https://cgit.freebsd.org/src/tree/contrib/libarchive/FREEBSD-upgrade)。

引入软件通常放置在源码树的 **contrib/** 子目录中，也有一些例外。仅由内核使用的引入软件位于 **sys/contrib/** 之下。

>**注意**
>
>由于会增加后续版本导入的难度，因此在仍然跟踪 vendor 分支的文件上，*强烈不建议* 进行次要的、无关紧要的或纯粹为美观的修改。

### 5.3.1. Vendor 导入

关于引入软件与 vendor 分支的标准管理流程详见 [提交者指南（Committer’s Guide）](https://docs.freebsd.org/en/articles/committers-guide/#vendor-import-git)。

## 5.4. 受限文件（Encumbered Files）

有时可能需要将受限文件添加到 FreeBSD 源码树中。例如，如果某个设备在运行前需要加载一段我们没有源代码的小型二进制代码，那么这个二进制文件就被视为受限文件。下面是将受限文件纳入 FreeBSD 源码树所需遵循的政策：

1. 任何由系统 CPU 执行或解释、且不是源代码格式的文件都属于受限文件。
2. 任何授权比 BSD 或 GNU 更严格的文件都属于受限文件。
3. 包含供硬件使用的可下载二进制数据的文件不被视为受限文件，除非第 (1) 或 (2) 条适用于它。
4. 添加任何受限文件必须获得 [核心团队](https://www.freebsd.org/administration/#t-core) 的特别批准。
5. 受限文件应放在 **src/contrib** 或 **src/sys/contrib**。
6. 整个模块应保持完整。除非与非受限代码有代码共享，否则无需拆分。
7. 过去二进制文件通常是 uuencode 编码的，并命名为 **arch/filename.o.uu**。这已不再必要，现在可以直接将二进制文件原样加入仓库。
8. 内核文件：

   1. 应始终在 **conf/files.\*** 中列出（为简化构建）。
   2. 应始终包含在 **LINT** 中，但是否注释由 [核心团队](https://www.freebsd.org/administration/#t-core) 个案决定。该团队以后可以更改决定。
   3. 是否纳入发行版由 *Release Engineer* 决定。
9. 用户态文件：

   1. 是否加入 `make world` 由 [核心团队](https://www.freebsd.org/administration/#t-core) 决定。
   2. 是否进入发行版由 [Release Engineering](https://www.freebsd.org/administration/#t-re) 决定。

## 5.5. 共享库

如果你要为某个 Port 或其他软件添加共享库支持，而该软件原本并不使用共享库，那么其版本号应遵循以下规则。通常，这些版本号与软件的发行版本无关。

对于 Port：

* 优先使用上游已经指定的版本号。
* 如果上游提供了符号版本控制（symbol versioning），应确保我们也使用其脚本。

对于 base 系统：

* 库版本号从 1 开始。
* 强烈建议为新库添加符号版本控制。
* 若存在不兼容的更改，应使用符号版本控制并保持向后 ABI 兼容性。
* 如果无法做到这一点，或者库没有使用符号版本控制，则需要提升库的版本号。
* 若需提升 symbol-versioned 库的版本号，必须事先与 Release Engineering 团队协商，说明为何这一更改如此重要，以致必须突破 ABI 兼容限制。

例如，添加函数或修复接口不变的 bug 都可以接受；但删除函数、改变函数调用语法等行为要么需要提供向后兼容的符号版本，要么需要提升主版本号。

更改库版本是提交者的责任。

ELF 动态链接器会精确匹配库名。目前的流行做法是将库名设为 `libexample.so.x.y`，其中 x 为主版本号，y 为次版本号。惯例是将库的 soname（ELF 中的 `DT_SONAME` 标签）设为 `libexample.so.x`，并在安装时建立符号链接：`libexample.so.x → libexample.so.x.y`，`libexample.so → libexample.so.x`。这样，静态链接器在使用 `-lexample` 选项时，会自动链接到正确的库。几乎所有流行的构建系统都会自动采用这种方式。
