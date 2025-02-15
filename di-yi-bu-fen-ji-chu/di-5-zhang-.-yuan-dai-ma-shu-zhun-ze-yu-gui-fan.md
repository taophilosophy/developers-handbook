# 第 5 章. 源代码树准则与规范

本章记录了适用于 FreeBSD 源代码树的各种准则和政策。

## 5.1. 风格指南

一致的编码风格非常重要，特别是对于像 FreeBSD 这样的大型项目。代码应该遵循在 style(9)和 style.Makefile(5)中描述的 FreeBSD 编码风格。

## 5.2. MAINTAINER 关于 Makefiles

如果 FreeBSD src/发行版的特定部分由一个或一组人维护，通过在 src/MAINTAINERS 中的条目进行通信传达。在Ports集合中维护者通过向有问题的port的 Makefile 添加 MAINTAINER 行向世界表达他们的维护权：

```
MAINTAINER= email-addresses
```

|  | 对于存储库的其他部分，或未列为具有维护者的部分，或当您不确定活跃维护者是谁时，请尝试查看相关源代码树最近提交历史。通常情况下，维护者没有明确命名，但是在源代码树的某个部分积极工作，比如过去几年来一直在那里工作的人，对于审查更改是感兴趣的。即使在文档或源文件中没有明确提到这一点，请求审查作为一种礼貌的形式是非常合理的事情。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

维护者的作用如下：

* 维护者拥有并负责该代码。这意味着他或她负责修复有关该代码部分的错误并回答问题报告，在涉及贡献的软件的情况下，需适时追踪新版本。
* 应该在提交变更到已定义维护者的目录之前将其发送给维护者进行审核。只有在维护者在不可接受的时间内未做出回应，即多封电子邮件后才允许在没有经过维护者审查的情况下提交更改。然而，建议尽可能让其他人审查这些更改。
* 当然，未经同意不可将个人或组添加为维护者来承担这一责任。另一方面，它不必是一个提交者，可以很容易地是一组人。

## 5.3. 贡献软件

FreeBSD 发行版的某些部分由 FreeBSD 项目之外积极维护的软件组成。出于历史原因，我们称之为贡献软件。一些例子包括 LLVM、zlib(3)和 awk(1)。

管理贡献软件的接受流程涉及创建供应商分支，可以在其中干净地导入软件（无需修改），并可以以版本化的方式跟踪更新。然后，将供应商分支的内容应用于源代码树，可能会进行本地修改。FreeBSD 特定的构建粘合剂在源代码树中维护，而不是在供应商分支中。

根据其需求和复杂性，个别软件项目可能会根据维护者的判断偏离此过程。更新特定贡献软件的确切步骤应记录在一个名为 FREEBSD-upgrade 的文件中；例如，libarchive 的 FREEBSD-upgrade 文件。

贡献的软件通常放置在源代码树的 contrib/子目录中，但也有一些例外情况。仅由内核使用的贡献软件位于 sys/contrib/下。

|  | 因为这样做会使未来版本的导入变得更加困难，强烈不建议对仍在跟踪供应商分支的文件进行次要、琐碎和/或表面的更改。 |
| -- | --------------------------------------------------------------------------------------------------------------- |

### 5.3.1. 供应商导入

管理贡献软件和供应商分支的标准流程在 Committer's Guide 中有详细描述。

## 5.4. 受限文件

偶尔可能需要在 FreeBSD 源代码树中包含一个受限制的文件。例如，如果设备在操作之前需要加载一小段二进制代码，并且我们没有该代码的源代码，那么该二进制文件就被称为受限制的。以下政策适用于在 FreeBSD 源代码树中包含受限制文件。

1. 任何由系统 CPU 解释或执行且不以源代码格式存在的文件都是受限制的。
2. 任何许可证比 BSD 或 GNU 更具限制性的文件都是受限制的。
3. 一个包含可下载二进制数据供硬件使用的文件不受限制，除非（1）或（2）适用于它。
4. 任何受限制的文件在添加到存储库之前都需要核心团队的特定批准。
5. 受限的文件放在 src/contrib 或 src/sys/contrib 中。
6. 整个模块应该保持在一起。除非存在与非受限制代码的代码共享，否则没有拆分的必要。
7. 在过去，二进制文件通常会被 uuencode，然后命名为 arch/filename.o.uu。这已不再必要，二进制文件可以无修改地添加到代码库中。
8. 内核文件：

    1. 在构建简易性时，应始终引用 conf/files.*中的内容。
    2. 应始终包含在 LINT 中，但核心团队根据情况决定是否应将其注释掉。核心团队当然可以随后改变他们的想法。
    3. 发布工程师决定是否将其包含在发布中。
9. 用户空间文件：

    1. 核心团队决定代码是否应该成为 make world 的一部分。
    2. 发行工程团队决定是否将其纳入发布版。

## 5.5. 共享库

如果您正在为port或其他没有共享库支持的软件添加支持，则版本号应遵循以下规则。通常，生成的版本号与软件的发布版本无关。

 对于ports:

* 优先使用上游已选择的数字
* 如果上游提供符号版本控制，请确保我们使用他们的脚本

对于基本系统：

* 从 1 开始启动库版本
* 强烈建议为新库添加符号版本
* 如果有不兼容的更改，请使用符号版本处理，保持向后 ABI 兼容
* 如果这是不可能的，或者库不使用符号版本控制，请提升库版本
* 即使考虑为符号版本化库提升库版本之前，请与发布工程团队协商，提供改变如此重要以至于应该允许破坏 ABI 的原因

例如，添加函数和修复不改变接口是可以的，而删除函数，更改函数调用语法等应提供向后兼容符号，否则将强制更改主版本号。

进行更改的提交者有责任处理库版本控制。

ELF 动态链接器会严格匹配库名称。一个流行的约定是将库版本写成 libexample.so.x.y 的形式，其中 x 是主版本，y 是次版本。常见的做法是将库的 soname ( DT_SONAME ELF 标签) 设置为 libexample.so.x ，并在安装库时设置 libexample.so.x→libexample.so.x.y 和 libexample.so→libexample.so.x 的符号链接以对应最新的次版本 y。因为静态链接器在指定 -lexample 命令行选项时会搜索 libexample.so ，所以与 libexample 链接的对象会依赖于正确的库。几乎所有流行的构建系统都会自动使用这个方案。
