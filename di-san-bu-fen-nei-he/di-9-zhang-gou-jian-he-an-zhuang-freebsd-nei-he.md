# 第 9 章 构建和安装 FreeBSD 内核


作为内核开发人员，了解内核构建过程是必要的。调试 FreeBSD 内核时，必须能够构建一个内核。已知有两种构建内核的方法：

支持的构建和安装内核的过程已在 FreeBSD 手册的 [构建和安装自定义内核](https://docs.freebsd.org/en/books/handbook/kernelconfig/#kernelconfig-building) 章节中进行了详细说明。

>**注意**
>
>本章节假设读者已经熟悉 FreeBSD 手册中的 [构建和安装自定义内核](https://docs.freebsd.org/en/books/handbook/kernelconfig/#kernelconfig-building) 章节中描述的信息。如果尚不熟悉，请阅读上述章节，了解构建过程的工作原理。

## 9.1. 更快但脆弱的构建方法

这种方法在进行内核代码开发时可能会更有用，并且在仅调整内核配置文件中的一个或两个选项时，实际可能比文档中描述的过程更快。另一方面，它可能会导致内核构建出现意外的中断。

1. 运行 [config(8)](https://man.freebsd.org/cgi/man.cgi?query=config&sektion=8&format=html) 来生成内核源代码：

   ```
   # /usr/sbin/config MYKERNEL
   ```
2. 进入构建目录。运行 [config(8)](https://man.freebsd.org/cgi/man.cgi?query=config&sektion=8&format=html) 后，会打印出此目录的名称。

   ```
   # cd ../compile/MYKERNEL
   ```
3. 编译内核：

   ```
   # make depend
   # make
   ```
4. 安装新内核：

   ```
   # make install
   ```
