# I. 服务器命令

此部分包含服务器命令的参考信息。目前只有pgpool属于这一类。

**目录**
pgpool -- pgpool-II主服务器

# pgpool

## 名字

pgpool --  Pgpool-II main server

## 简介

`pgpool` [`*option*`...]

`pgpool` [`*option*`...] `*stop*`

`pgpool` [`*option*`...] `*reload*`



## 描述

the Pgpool-II main server



## 使用

pgpool以3种模式运行：start, stop 和 reload。如果没有给出停止或重新加载，则假定指定了“启动”。

## 常用选项

这些是3种模式的常见选项。

- -a hba_config_file
- --hba-file=hba_config_file

	设置pool_hba.conf配置文件的路径。如果文件放置在标准位置以外，则为必填项。

- -f config_file
- --config-file=config_file

	设置pgpool.conf配置文件的路径。如果文件放置在标准位置以外，则为必填项。

- -F pc_config_file
- --pcp-file=pcp_config_file

	设置pcp.conf配置文件的路径。如果文件放置在标准位置以外，则为必填项。

- -k key_file
- --key-file=key_file

	设置.pgpoolkey文件的路径。如果您使用AES256加密密码，并且文件放置在标准位置之外并被使用，则必须使用。

- -h
- --help

	打印帮助信息。

## 启动Pgpool-II主服务器

以下是启动模式的选项。

- -d
- --debug

	在调试模式下运行Pgpool-II。产生了大量的调试消息。

- -n
- --dont-detach

	不在守护进程模式下运行，不分离控制ttys。

- -x
- --debug-assertions

	打开各种断言检查，这是一个调试辅助工具。

- -C
- --clear-oidmaps

	当[memqcache_method](https://www.pgpool.net/docs/latest/en/html/runtime-in-memory-query-cache.html#GUC-MEMQCACHE-METHOD)是“memcached”时清除查询缓存oidmap。

	如果memqcache_method为“shmem”，Pgpool-II在启动时总是丢弃oidmap。因此，这个选项是不必要的。

- -D
- --discard-status

	丢弃“pgpool_status”文件，不恢复以前的状态。

	> [!CAUTION]
	> 此选项仅用于开发人员的测试目的，您不应将其用于其他目的。如果pgpool_status被意外删除，pgpool-II可能会进入分裂大脑（存在多个主服务器）。

## 停止Pgpool-II主服务器

以下是停止模式的选项。

- -m shutdown_mode
- --mode=shutdown_mode

	停止Pgpool-II。shutdown_mode是智能、快速或即时的。如果指定了智能，Pgpool-II将等待所有客户端断开连接。如果指定了快速或立即，Pgpool-II会立即停止运行，而无需等待所有客户端断开连接。在当前的实施中，快速和即时之间没有区别。

## 重新加载Pgpool-II的配置文件

重新加载Pgpool-II的配置文件。重载模式没有特定的选项。通用选项适用。
