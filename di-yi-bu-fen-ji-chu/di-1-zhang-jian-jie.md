# 第 1 章 简介


## 1.1. 在 FreeBSD 上进行开发

我们现在准备开始了。系统已经安装好，你也准备开始编程了。但是从哪里开始呢？FreeBSD 提供了什么？作为程序员，它能为我做什么？

本章试图回答这些问题。当然，像其他任何技能一样，编程也有不同的熟练程度。对一些人来说，这是业余爱好；对另一些人来说，这是职业。本章的信息可能更偏向初学者；事实上，它对那些不熟悉 FreeBSD 平台的程序员可能非常有帮助。

## 1.2. BSD 的愿景

打造最佳的类 UNIX® 操作系统套件，既尊重最初的软件工具理念，也兼顾可用性、性能与稳定性。

## 1.3. 架构指导方针

我们的理念可通过以下指导方针来描述：

* 除非实现者无法完成一个真实的应用，否则不要添加新功能。
* 明确一个系统不是什么，与定义它是什么同样重要。不要试图满足全世界的所有需求；相反，应使系统具有可扩展性，以便以向后兼容的方式满足额外需求。
* 唯一比根据一个例子泛化更糟的，是在没有任何例子的情况下泛化。
* 如果一个问题尚未完全理解，最好根本不要提供解决方案。
* 如果用 10% 的工作量就能实现 90% 的效果，那就选更简单的方案。
* 尽可能将复杂性隔离开来。
* 提供机制，而不是政策。特别是，将用户界面策略交给客户端掌控。

摘自 Scheifler 与 Gettys：《X Window System》

## 1.4. /usr/src 的结构

FreeBSD 的完整源代码可从我们的 [公共 Git 仓库](https://cgit.freebsd.org/src/) 获取。源代码通常安装在 **/usr/src**。源代码树的结构可通过顶层的 [README.md](https://cgit.freebsd.org/src/tree/README.md) 文件了解。
