# 第3章 服务器设置和操作

[TOC]

本章讨论如何设置和运行Pgpool-II服务器及其与操作系统的交互。

## 3.1. Pgpool II用户帐户

与外部世界可访问的任何服务器守护进程一样，建议在单独的用户帐户下运行Pgpool-II。此用户帐户应仅拥有服务器管理的数据，不应与其他守护进程共享。（例如，使用用户nobody是一个坏主意。）不建议安装此用户拥有的可执行文件，因为受感染的系统可能会修改自己的二进制文件。

要在系统中添加Unix用户帐户，请查找命令useradd或adduser。用户名pgpool经常被使用，并且在本书中一直被使用，但如果你愿意，你可以使用另一个名字。

## 3.2. 配置pcp.conf

Pgpool-II为管理员提供了一个界面来执行管理操作，例如远程获取Pgpool-II状态或终止Pgpool-II进程。pcp.conf是此接口用于身份验证的用户/密码文件。所有操作模式都需要设置pcp.conf文件。在Pgpool-II的安装过程中，会创建一个\$/etc/pcp.conf.sample文件。将文件复制为$prefix/etc/pcp.conf，并在其中添加您的用户名和密码。

```shell
$ cp $prefix/etc/pcp.conf.sample $prefix/etc/pcp.conf
```

空行或以#开头的行被视为注释，将被忽略。用户名及其关联密码必须使用以下格式写为一行：

```shell
username:[md5 encrypted password]
```

可以使用$prefix/bin/pg_md5命令生成[md5 encrypted password]。

```shell
$ pg_md5 your_password
1060b7b46a3bd36b3a0d66e0127d0517
```

如果您不想将密码作为参数传递，请执行pg_md5 -p。

```shell
$ pg_md5 -p
password: your_password
```

执行Pgpool-II的用户必须能够读取pcp.conf文件。

## 3.3. 配置Pgpool-II

### 3.3.1. 配置pgpool.conf

pgpool.conf是pgpool-II的主要配置文件。使用-f选项启动Pgpool-II时，需要指定文件的路径。如果从源代码安装，默认情况下pgpool.conf位于$prefix/etc/pgpool.conf。

要指定Pgpool-II群集模式，请将backend_clustering_mode参数设置为下面解释的值。

表3-1 pgpool.conf中的backend_clustering_mode值

| 集群模式          | 值                  |
| -------------------------- | ----------------------- |
| Streaming replication mode | streaming_replication |
| Replication mode           | native_replication    |
| Logical replication mode   | logical_replication   |
| Slony mode                 | slony                 |
| Snapshot isolation mode    | snapshot_isolation    |
| Raw mode                   | raw                   |

这些配置文件位于/usr/local/etc，默认从源代码安装。您可以将其中一个复制为pgpool.conf。（可能需要root权限）

```shell
# cd /usr/local/etc
# cp pgpool.conf.sample pgpool.conf
```

### 3.3.2. Pgpool-II的集群模式

Pgpool-II中有六种不同的集群模式：流式复制模式、逻辑复制模式、主副本模式（slony模式）、本机复制模式、原始模式和快照隔离模式。在任何模式下，Pgpool-II都提供连接池和自动故障转移。在线恢复只能用于流式复制模式、快照隔离模式和本机复制模式。有关在线恢复的更多详细信息，请参阅第5.11节。

这些模式是相互排斥的，启动服务器后无法更改。您应该在设计系统的早期阶段决定使用哪个。如果您不确定，建议使用流式复制模式或快照隔离模式。

流式复制模式可用于运行流式复制的PostgreSQL服务器。在这种模式下，PostgreSQL负责同步数据库。这种模式被广泛使用，也是使用Pgpool-II的最推荐方式。在该模式下可以进行负载平衡。无法保证节点之间的可见性一致性。

在快照隔离模式下，Pgpool-II负责同步数据库。该模式的优点是同步是以同步方式完成的：在所有PostgreSQL服务器完成写入操作之前，不会返回对数据库的写入。此外，它还保证了节点之间的可见性一致性。简单地说，这意味着单个服务器上事务的可见性规则也适用于由多个服务器组成的集群。这是Pgpool-II中快照隔离模式的一个显著特征。事实上，Pgpool-II中的快照隔离模式是目前唯一一个在不修改PostgreSQL的情况下保证节点之间可见性一致性的系统。因此，应用程序不需要认识到它们使用的是由PostgreSQL服务器组成的集群，而不是单个PostgreSQL系统。但是，在此模式下，事务隔离级别必须为可重复读取。您需要按如下方式设置postgresql.conf：

```shell
default_transaction_isolation = 'repeatable read'
```

此外，您还需要意识到，由于保持事务一致性的开销，该模式下的性能可能比流式复制模式和本机复制模式差。

在本机复制模式下，Pgpool-II负责同步数据库。该模式的优点是同步是以同步方式完成的：在所有PostgreSQL服务器完成写入操作之前，不会返回对数据库的写入。由于无法保证节点之间的可见性一致性，建议使用快照隔离模式，除非您想使用REPEATABLE READ以外的隔离模式。在该模式下可以进行负载平衡。

逻辑复制模式可用于运行逻辑复制的PostgreSQL服务器。在这种模式下，PostgreSQL负责同步表。在该模式下可以进行负载平衡。由于逻辑复制不会复制所有表，因此用户有责任复制可以负载平衡的表。Pgpool-II对所有表进行负载平衡。这意味着，如果不复制表，Pgpool-II可能会在用户端查找过时的表。

主副本模式（slony模式）可用于运行slony的PostgreSQL服务器。在这种模式下，Slony/PPostgreSQL负责同步数据库。由于流式复制正在淘汰Slony-I，我们不建议使用此模式，除非您有使用Slony的具体原因。在该模式下可以进行负载平衡。

在原始模式下，Pgpool-II不关心数据库同步。让整个系统做一件有意义的事情是用户的责任。在该模式下无法进行负载平衡。

### 3.3.3. 流程管理模式

Pgpool-II实现了一个多进程架构，其中每个子进程在任何时候都可以处理一个客户端连接。Pgpool-II可以处理的并发客户端连接总数由num_init_children配置参数配置。Pgpool-II支持两个子进程管理模式。动态和静态。在静态进程管理模式下，Pgpool-II在启动时预分叉子进程的num_init_children号，每个子进程都在监听传入的客户端连接。在动态进程管理模式下，Pgpool-II会跟踪空闲进程，并分叉或杀死进程，以将此数字保持在指定的边界内。

process_management_mode在Pgpool II V4.4之前不可用。

## 3.4. 配置后端信息

要让Pgpool-II识别PostgreSQL后端服务器，您需要在Pgpool.conf中配置backend*。对于初学者，至少需要设置backend_hostname和backend_port参数才能启动Pgpool-II服务器。

### 3.4.1. 后端设置

Pgpool-II使用的后端PostgreSQL必须在Pgpool.conf中指定。请参阅第5.5节

## 3.5. 启动Pgpool-II和PostgreSQL

要启动Pgpool II，请执行：

```shell
$ pgpool -f /usr/local/etc/pgpool.conf -F /usr/local/etc/pcp.conf
```

这将启动服务器在后台运行。“-f”指定主pgpool配置文件的路径，“-f”指定pcp服务器的配置文件路径，pcp服务器是pgpool-II的控制服务器。有关该命令的其他选项，请参阅pgpool手册。

在启动Pgpool-II之前，您必须启动PostgreSQL，因为如果PostgreSQL尚未启动，Pgpool-II会触发故障转移过程，使PostgreSQL处于关闭状态。

如果你在控制PostgreSQL的启动顺序方面有困难，例如Pgpool-II和PostgreSQL安装在不同的服务器上，你可以延长search_primary_node_timeout（默认值为5分钟），这样Pgpool-II就会等待PostgreSQL启动，直到search_primari_node_timeut到期。如果PostgreSQL在search_primary_node_timeout到期之前启动，Pgpool-II应该会顺利启动。如果search_primary_node_timeout在PostgreSQL启动之前到期，则不会检测到主节点，这意味着您无法执行DML/DDL。在这种情况下，您需要重新启动Pgpool-II。要确认主节点存在，可以使用SHOW POOL NODES命令。

请注意，search_primary_node_timeout只能在流式复制模式下使用，因为该参数仅在该模式下有效。有关流式复制模式的更多详细信息，请参阅第3.3.2节。对于其他模式，调整健康检查（见第5.9节）参数，以便在PostgreSQL可用之前有足够的时间。

如果健康检查检测到PostgreSQL在Pgpool-II启动之前不可用，则部分或全部PostgreSQL将被识别为“关闭”状态。在这种情况下，您需要使用pcp_tach_node命令手动将PostgreSQL服务器置于“up”状态。如果客户端在PostgreSQL可用之前尝试连接到Pgpool-II，则可能会触发故障转移。在这种情况下，您还需要执行pcp_tach_node命令，将PostgreSQL服务器置于“up”状态。

## 3.6. 停止Pgpool-II和PostgreSQL

要停止Pgpool II，请执行：

```shell
$ pgpool -f /usr/local/etc/pgpool.conf -F /usr/local/etc/pcp.conf -m fast stop
```

“-m”选项指定如何缓慢停止Pgpool-II。“快速”意味着即使有来自客户端的现有连接，也要立即关闭Pgpool-II。您可以为该选项指定“智能”，这将强制Pgpool-II等待，直到所有客户端都与Pgpool-II断开连接。但这可能会使Pgpool-II永远等待，这可能会导致从操作系统发送SIGKILL信号并留下垃圾，这将在下次启动Pgpool-II时带来麻烦。

关闭Pgpool-II后，您可以关闭PostgreSQL。

## 3.7. 暂时关闭PostgreSQL

有时你想暂时停止或重启PostgreSQL来维护或升级它。在本节中，我们将介绍如何在最短的停机时间内执行任务。

### 3.7.1. 使用pcp_deach_node命令

如果你使用pg_ctl停止PostgreSQL，故障转移将不会发生，直到Pgpool-II根据健康检查设置通过健康检查检测到它，并且需要一些时间来分离PostgreSQL。特别是如果启用了看门狗并且启用了failover_require_sensus，Pgpool-II将不会启动故障转移，直到超过一半的看门狗节点同意PostgreSQL停止。如果使用pcp_deach_node分离节点，则无论健康检查的设置如何，故障转移都将立即开始。请注意，分离的PostgreSQL节点实际上并没有停止，如有必要，您需要手动停止它。

### 3.7.2. 使用backend_flag

停止或重新启动PostgreSQL会导致故障转移。如果运行模式不是流复制模式，或者服务器是流复制模式下的备用服务器，这可能没什么大不了的，因为客户端总是可以使用群集中的其他服务器。但是，如果服务器是主服务器，则会通过升级其中一个备用服务器来更改主服务器。此外，如果集群中只剩下一台服务器，则没有可以升级的替代服务器或备用服务器。

在这种情况下，您可以使用backend_flag来避免故障转移。通过在pgpool.conf中进行以下设置，可以避免backend0的故障转移。

```shell
backend_flag0 = DISALLOW_TO_FAILOVER
```

这将通过重新加载或重新启动Pgpool-II来生效。如果设置了此标志，则如果后端不可用，则不会发生故障转移。当后端不可用时，客户端将收到错误消息：

```shell
psql: error: could not connect to server: FATAL:  failed to create a backend connection
DETAIL:  executing failover on backend
```

重启后端后，客户端可以像往常一样连接。要再次允许后端故障转移，您可以设置：

```shell
backend_flag0 = ALLOW_TO_FAILOVER
```

并重新加载或重启Pgpool-II。

## 3.8. 备份PostgreSQL数据库

如果您计划使用pg_dump、pg_basebackup或任何其他工具备份PostgreSQL数据库，我们强烈建议您直接对PostgreSQL运行这些命令。由于Pgpool-II是一个代理软件，它为中继消息包提供了开销。由于获取备份往往会产生大量数据包，因此与直接连接PostgreSQL相比，通过Pgpool-II执行备份将很慢，除非数据库非常小。

此外，如果通过Pgpool-II执行并行pg_dump，则会引发错误，因为该命令处理快照id，这是一个依赖于数据库的对象。

在大多数情况下，您希望选择主服务器作为备份目标。如果你想要备份备用服务器，你必须非常小心地选择合适的PostgreSQL服务器来获取备份，因为如果数据过时，你可能会有过时的数据库备份。您可以使用SHOW POOL NODES或pcp_node_info来了解备用服务器如何赶上主服务器。