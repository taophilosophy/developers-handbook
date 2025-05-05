# 第 6 章 回归测试与性能测试

回归测试用于针对系统中的特定部分进行测试，以确认其按预期运行，并确保旧的错误不会被重新引入。

FreeBSD 的回归测试工具可以在 FreeBSD 源码树中的 **src/tools/regression** 目录下找到。

## 6.1. 微基准测试检查清单

本节包含在 FreeBSD 上或对 FreeBSD 本身进行正确微基准测试的建议。

并非每次测试都能用到以下所有建议，但使用得越多，基准测试检测微小差异的能力就越强。

* 禁用 APM 以及任何其他形式的时钟干预（ACPI？）。
* 以单用户模式运行。例如，[cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html) 和其他守护进程只会增加干扰。[sshd(8)](https://man.freebsd.org/cgi/man.cgi?query=sshd&sektion=8&format=html) 守护进程也可能造成问题。如果测试时需要 ssh 访问，可以禁用 SSHv1 密钥重新生成，或者在测试过程中杀死 `sshd` 的父进程。
* 不要运行 [ntpd(8)](https://man.freebsd.org/cgi/man.cgi?query=ntpd&sektion=8&format=html)。
* 如果会产生 [syslog(3)](https://man.freebsd.org/cgi/man.cgi?query=syslog&sektion=3&format=html) 事件，可以让 [syslogd(8)](https://man.freebsd.org/cgi/man.cgi?query=syslogd&sektion=8&format=html) 读取一个空的 **/etc/syslogd.conf**，否则就不要运行它。
* 最小化磁盘 I/O，若可行则完全避免。
* 不要挂载不需要的文件系统。
* 如果可能，将 **/**、**/usr** 以及其他文件系统挂载为只读。这可以避免因访问时间（atime）更新而引起的磁盘 I/O 干扰。
* 每次运行前用 [newfs(8)](https://man.freebsd.org/cgi/man.cgi?query=newfs&sektion=8&format=html) 重新初始化读写测试使用的文件系统，并用 [tar(1)](https://man.freebsd.org/cgi/man.cgi?query=tar&sektion=1&format=html) 或 [dump(8)](https://man.freebsd.org/cgi/man.cgi?query=dump&sektion=8&format=html) 文件进行填充。在测试开始前卸载并重新挂载，这样可以获得一致的文件系统布局。对于 worldstone 测试，这适用于 **/usr/obj**（只需用 `newfs` 重新初始化并挂载即可）。若想实现 100% 可重现性，可使用 [dd(1)](https://man.freebsd.org/cgi/man.cgi?query=dd&sektion=1&format=html) 命令从映像填充文件系统（例如：`dd if=myimage of=/dev/ad0s1h bs=1m`）。
* 使用由 malloc 支持或预加载的 [md(4)](https://man.freebsd.org/cgi/man.cgi?query=md&sektion=4&format=html) 分区。
* 在每次测试迭代之间重启系统，以保证状态一致。
* 从内核中移除所有非必要的设备驱动程序。例如，如果测试中不需要 USB，就不要在内核中加入 USB。驱动程序往往有计时器在运行。
* 卸载不使用的硬件。使用 [atacontrol(8)](https://man.freebsd.org/cgi/man.cgi?query=atacontrol&sektion=8&format=html) 和 [camcontrol(8)](https://man.freebsd.org/cgi/man.cgi?query=camcontrol&sektion=8&format=html) 卸载不参与测试的磁盘。
* 如果不是测试网络，测试前不要配置网络，或者测试结束后再将结果发出。
* 禁用“Turbo 模式”，因为它会根据环境调整时钟频率。这意味着即使代码完全相同，基准测试结果也可能因时间、饮品甚至办公室里其他人的存在而不同。

如果系统必须连接公共网络，要注意广播流量的突发峰值。虽然几乎察觉不到，但它仍然会占用 CPU 周期。多播（Multicast）也存在类似问题。
* 将每个文件系统放在单独的磁盘上，减少因磁头寻道优化带来的抖动。
* 尽量减少串口或 VGA 控制台的输出。将输出写入文件可减少抖动。（串口控制台容易成为瓶颈。）测试过程中不要触碰键盘，哪怕是空格或退格键也会影响结果。
* 保证测试时间足够长，但不要太长。若测试太短，时间戳精度不足；若太长，温度变化和晶体频率漂移会影响结果。经验法则：不少于 1 分钟，不超过 1 小时。
* 保持测试环境温度尽量稳定。这会影响晶体振荡器和硬盘算法。如需稳定时钟，可考虑注入稳定时钟信号。例如，使用 OCXO + PLL，并将输出注入主板时钟电路以替代晶振。联系 Poul-Henning Kamp（[phk@FreeBSD.org](mailto:phk@FreeBSD.org)）获取更多信息。
* 每个测试至少运行 3 次，最好是“修改前”和“修改后”都运行超过 20 次。如有可能，交叉运行（例如：不要先运行 20 次“前”，再运行 20 次“后”），这样有助于识别环境因素。不要按 1:1 交替运行，而是 3:3，有助于发现交互效应。

推荐模式为：`bababa{bbbaaa}*`。
这样，前 1+1 次就能初步判断趋势（若完全跑偏可及时终止测试），3+3 次可以估算标准差（决定是否值得长时间运行），后续数据可用于趋势和交互分析。
* 使用 [ministat(1)](https://man.freebsd.org/cgi/man.cgi?query=ministat&sektion=1&format=html) 判断数值是否有统计意义。如果你忘记或从未学过标准差和 Student's T 分布，强烈推荐《Cartoon guide to statistics》，ISBN: 0062731025。
* 不要使用后台 [fsck(8)](https://man.freebsd.org/cgi/man.cgi?query=fsck&sektion=8&format=html)，除非正在基准测试该功能。同时，在 **/etc/rc.conf** 中禁用 `background_fsck`，除非基准测试是在启动后至少延迟了 60 秒加 `fsck` 运行时间之后进行的，因为 [rc(8)](https://man.freebsd.org/cgi/man.cgi?query=rc&sektion=8&format=html) 会在启动时检查是否需要运行后台 `fsck`。
同样，除非是测试快照功能，否则确保没有残留快照存在。
* 如果基准测试表现异常差，检查是否有异常的中断源导致的中断量暴增。有报告指出部分 ACPI 版本存在“异常行为”，会产生过多中断。为诊断异常测试结果，可以用 `vmstat -i` 拍几张快照，看看有没有异常。
* 注意内核和用户空间的优化参数，以及调试选项。很容易不小心遗漏某项，最后发现测试内容不一致。
* 除非专门测试这些功能，不要在启用了 `WITNESS` 和 `INVARIANTS` 的内核下进行基准测试。`WITNESS` 可能导致性能下降 400% 以上。类似地，-CURRENT 中用户空间的 [malloc(3)](https://man.freebsd.org/cgi/man.cgi?query=malloc&sektion=3&format=html) 参数默认与生产版不同。

## 6.2. FreeBSD 源码 Tinderbox

Tinderbox 包括以下部分：

* 一个构建脚本 **tinderbox**，自动签出指定版本的 FreeBSD 源码树并构建。
* 一个监督脚本 **tbmaster**，监视各个 Tinderbox 实例、记录输出，并发送失败通知邮件。
* 一个名为 **index.cgi** 的 CGI 脚本，用于读取 tbmaster 日志并以 HTML 格式生成简洁易读的摘要。
* 一组持续构建 FreeBSD 各主要代码分支最新状态的构建服务器。
* 一个网页服务器，用于保存完整的 Tinderbox 日志，并显示最新摘要信息。

这些脚本由 Dag-Erling Smørgrav（[des@FreeBSD.org](mailto:des@FreeBSD.org)）开发并维护，现已从最初的 shell 脚本迁移为 Perl 编写。所有脚本与配置文件存放在 [/projects/tinderbox/](https://www.freebsd.org/cgi/cvsweb.cgi/projects/tinderbox/)。

关于 tinderbox 和 tbmaster 脚本的更多信息，请参见它们的手册页：tinderbox(1) 和 tbmaster(1)。

## 6.3. index.cgi 脚本

**index.cgi** 脚本会生成 Tinderbox 和 tbmaster 日志的 HTML 摘要。虽然名字显示这是一个 CGI 脚本，但它也可以从命令行或 [cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html) 作业中运行，此时会在脚本所在目录查找日志。它可自动检测运行上下文，在作为 CGI 脚本运行时会生成 HTTP 头信息。它符合 XHTML 标准并使用 CSS 样式。

脚本从 `main()` 开始，首先验证是否在官方 Tinderbox 网站上运行。如果不是，则会显示一页提示，并给出官方网站的链接。

接着，它扫描日志目录，获取存在日志文件的配置项、分支和架构清单，避免在脚本中硬编码这些列表，造成空行或空列。该信息由日志文件名提取，文件名需匹配以下格式：

```sh
tinderbox-$config-$branch-$arch-$machine.{brief,full}
```

官方 Tinderbox 构建服务器使用的配置名称与其构建的分支一致。例如，`releng_8` 配置用于构建 `RELENG_8` 以及所有仍受支持的 8.x 发布分支。

完成启动过程后，会为每个配置调用 `do_config()`。

`do_config()` 会为单个 Tinderbox 配置生成 HTML 内容。

它先生成表头行，然后遍历指定配置下的每个分支构建，每个分支输出一行，流程如下：

* 对每个项目：

  * 对该架构下的每台机器：

    * 若存在简略日志文件：

      * 调用 `success()` 判断构建是否成功。
      * 输出修改大小。
      * 输出简略日志文件大小并附带链接。
      * 如果存在完整日志文件：

        * 输出完整日志文件大小并附带链接。
    * 否则：

      * 不输出内容。

上述 `success()` 函数会扫描简略日志文件中是否含有 “tinderbox run completed” 字样，以判断构建是否成功。

配置和分支按其“分支等级”排序，排序规则如下：

* `HEAD` 和 `CURRENT` 的等级为 9999。
* `RELENG_x` 的等级为 *`xx`*99。
* `RELENG_x_y` 的等级为 *xxyy*。

这意味着 `HEAD` 总是最高等级，`RELENG` 分支按数字升序排列，每个 `STABLE` 分支高于其派生的 release 分支。例如，FreeBSD 8 中的顺序为：

* `RELENG_8`（等级 899）
* `RELENG_8_3`（等级 803）
* `RELENG_8_2`（等级 802）
* `RELENG_8_1`（等级 801）
* `RELENG_8_0`（等级 800）

Tinderbox 用 CSS 定义表格中每个单元格的颜色。构建成功显示绿色文字，构建失败显示红色文字。颜色会随时间推移逐渐变灰，每过半小时颜色就更趋向灰色。

## 6.4. 官方构建服务器

官方 Tinderbox 构建服务器由 [Sentex Data Communications](http://www.sentex.ca/) 托管，该公司同时也托管 FreeBSD Netperf 集群。

目前有三个构建服务器：

**freebsd-current.sentex.ca** 构建：

* `HEAD`，适用于 amd64、arm、i386、i386/pc98、ia64、mips、powerpc、powerpc64 和 sparc64。
* `RELENG_9` 以及受支持的 9.*X* 分支，适用于 amd64、arm、i386、i386/pc98、ia64、mips、powerpc、powerpc64 和 sparc64。

**freebsd-stable.sentex.ca** 构建：

* `RELENG_8` 以及受支持的 8.*X* 分支，适用于 amd64、i386、i386/pc98、ia64、mips、powerpc 和 sparc64。

**freebsd-legacy.sentex.ca** 构建：

* `RELENG_7` 以及受支持的 7.*X* 分支，适用于 amd64、i386、i386/pc98、ia64、powerpc 和 sparc64。

## 6.5. 官方摘要站点

来自官方构建服务器的摘要与日志可通过 [http://tinderbox.FreeBSD.org](http://tinderbox.freebsd.org/) 在线访问，由 Dag-Erling Smørgrav（[des@FreeBSD.org](mailto:des@FreeBSD.org)）托管并按如下方式设置：

* 一个 [cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html) 任务定期检查构建服务器，并使用 [rsync(1)](https://man.freebsd.org/cgi/man.cgi?query=rsync&sektion=1&format=html) 下载任何新的日志文件。
* Apache 被设置为使用 **index.cgi** 作为 `DirectoryIndex`。
