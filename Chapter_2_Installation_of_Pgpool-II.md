# 第2章 安装Pgpool II

[TOC]

## 2.1. 规划

由于Pgpool-II是管理PostgreSQL的工具，我们需要先决定如何部署它们。此外，可以安装多个Pgpool-II，以提高Pgpool-II本身的可用性。我们需要事先计划需要安装多少Pgpool-II。在本章中，我们首先讨论PostgreSQL的运行模式，然后讨论Pgpool-II的部署。

### 2.1.1. PostgreSQL的集群模式

PostgreSQL可以安装一个或多个，通常会安装两个以上，因为如果只有一个，当PostgreSQL不可用时，整个数据库系统都会崩溃。当我们使用两个或多个PostgreSQL服务器时，有必要以某种方式同步数据库。我们将同步数据库的方法称为“clustering running mode（集群运行模式）”。有史以来最流行的模式是“streaming replication mode（流式复制模式）”。除非需要特别考虑，否则建议使用流式复制模式。有关运行模式的更多详细信息，请参阅第3.3.2节。

接下来我们需要考虑的是我们想要安装多少个PostgreSQL。如果有两个，我们可以继续操作数据库系统。然而，如果你想通过在多个服务器上通过读取查询负载平衡来运行多个读取查询，使用两个以上的PostgreSQL并不罕见。Pgpool-II提供了丰富的选项来调整负载平衡。更多详细信息请参见第5.8节。

可以通过Pgpool-II中添加PostgreSQL服务器，因此两个PostgreSQL可能是一个很好的起点。

### 2.1.2. Pgpool-II的部署

虽然可以只使用一个Pgpool-II，但我们建议使用多个PgpoolII，以避免由于Pgpool-II关闭而导致整个数据库不可用。多个Pgpool-II协同工作并相互监控。其中一个被称为“leader”，它有一个虚拟IP。客户不需要知道有多个Pgpool-II，因为他们总是访问同一个VIP。（监视器见第4.1节）。如果Pgpool-II中的一个发生故障，另一个Pgpool-II将接管leader角色。

由于不允许有多个leader，监视器会投票决定新的leader。如果Pgpool-II的数量是偶数，则不可能通过投票决定新leader。因此，我们建议部署大于或等于3个奇数的Pgpool-II。

请注意，Pgpool-II和PostgreSQL可以放在同一台服务器上。例如，你可以只有三台服务器在每台服务器上运行Pgpool-II和PostgreSQL。

您可以在第8.2节中找到一个生产级的详细示例，该示例使用三个Pgpool-II和两个PostgreSQL在流式复制模式下，供那些今天想要安装生产级Pgpool-II的人使用。

## 2.2. Pgpool II的安装

本章介绍Pgpool-II的安装。首先介绍从源代码进行安装。然后介绍从RPM软件包安装。

## 2.3. 要求

一般来说，一个现代的Unix兼容平台应该能够运行Pgpool-II。不支持Windows。

构建Pgpool-II需要以下软件包：

- 需要GNU make 3.80或更高版本；其他make程序或较旧的GNU make版本将无法运行。（有的GNU make安装名字是gmake。）要测试GNU make，请输入：

```shell
make --version
```

- 你需要一个ISO/ANSI C编译器（至少兼容C89）。建议使用GCC的最新版本，但众所周知，Pgpool-II可以使用来自不同供应商的各种编译器进行构建。

- 除了gzip之外，还需要tar来解压缩源发行版。

- 安装Pgpool-II需要几个PostgreSQL包。您可以从rpm安装postgresql-libs和postgresql-devel包。

如果你是从Git树构建而不是使用已发布的源代码包，或者如果你想进行服务开发，你还需要以下包：

- Flex和Bison需要从Git签出构建，或者如果您更改了实际的扫描程序和解析器定义文件。如果你需要它们，一定要获取Flex 2.5.31或更高版本和Bison 1.875或更高型号。其他lex和yacc程序无法使用。


如果你需要一个GNU软件包，你可以在当地的GNU镜像站点找到它（参见http://www.gnu.org/order/ftp.html列表）或ftp://ftp.gnu.org/gnu/。

还要检查是否有足够的磁盘空间。编译期间，源代码树大约需要40 MB，安装目录大约需要20 MB。如果你要运行回归测试，你暂时需要额外的4 GB。使用df命令检查可用磁盘空间。

## 2.4. 获取来源

Pgpool II 4.5.4的源代码可以从官方网站下载：http://www.pgpool.net.您应该得到一个名为pgpool-II-4.5.4.tar.gz的文件。获取文件后，解压缩它：

```shell
tar xf pgpool-II-4.5.4.tar.gz
```

这将在当前目录下创建一个包含pgpool-II源代码的目录pgpool-II-4.5.4。在安装过程的其余部分切换到该目录。

## 2.5. 安装Pgpool-II

提取源代码tarball后，按照以下步骤编译源代码并安装Pgpool-II。

自Pgpool-II 4.5以来，由autoconf/autoreconf生成的configure等文件已从存储库中删除，因此请先运行autoreconf -fi以生成configure。

```shell
dnf install libtool

cd pgpool-II-4.5.4
autoreconf -fi
```

接下来，执行配置脚本。

```shell
./configure
```

您可以通过提供以下一个或多个命令行选项运行configure来自定义编译和安装过程：

--prefix=path

​	指定Pgpool-II二进制文件和相关文件（如文档）的安装位置。默认值为/usr/local。

--with-pgsql=path

​	指定安装PostgreSQL客户端库的顶级目录。默认值是pg_config命令提供的路径。

--with-openssl

​	Pgpool-II二进制文件将使用OpenSSL编译。如果您计划使用AES256加密对密码进行加密，您也需要此选项。更多详细信息请参见第6.4节。默认情况下关闭OpenSSL支持。

--enable-sequence-lock

​	使用与Pgpool-II 3.0系列兼容的insert_lock（3.0.4之前）。Pgpool-II锁定序列表中的一行。2011年6月之后发布的PostgreSQL 8.2或更高版本无法使用此锁方法。

--enable-table-lock

​	使用与Pgpool-II 2.2和2.3系列兼容的insert_lock。Pgpool-II锁定插入目标表。此锁方法已被弃用，因为它会导致与VACUUM的锁冲突。

--with-memcached=path

​	Pgpool-II二进制文件将使用memcached进行内存查询缓存。您必须安装libmemcached。

--with-pam

​	Pgpool-II二进制文件将使用PAM身份验证编译。默认情况下关闭PAM身份验证支持。

编译源文件。

```shell
make
```

安装Pgpool-II。

```shell
make install
```

这将安装Pgpool II。（如果您使用Solaris或FreeBSD，请将make替换为gmake）

## 2.6. 安装pgpool_recovery

当使用在线恢复时，Pgpool-II需要pgpool_recovery、pgpool_remote_start和pgpool_switch_xlog功能。还有管理工具的pgpoolAdmin，使用pgpool_pgctl在停止、重启或重新加载PostgreSQL。这些功能安装在template1中就足够了。并非所有数据库都需要安装这些功能。

这是所有Pgpool-II安装所必需的。

```shell
$ cd pgpool-II-4.5.4/src/sql/pgpool-recovery
$ make
$ make install
```

在此之后：

```shell
$ psql template1
=# CREATE EXTENSION pgpool_recovery;
```

或者

```shell
$ psql -f pgpool-recovery.sql template1
```

使用Pgpool-II 3.3或更高版本，您需要调整postgresql.conf。假设pg_ctl的路径是/usr/local/pgsql/bin/pg_ctl。然后将以下内容添加到postgresql.conf中。

```shell
pgpool.pg_ctl = '/usr/local/pgsql/bin/pg_ctl'
```

您可能希望在此之后执行以下操作：

```shell
$ pg_ctl reload -D /usr/local/pgsql/data
```

## 2.7. 安装pgpool-regclass

如果你使用的是PostgreSQL 9.4或更高版本，你可以跳过这一节。

如果你使用的是PostgreSQL 8.0到PostgreSQL 9.3，强烈建议在pgpool-II访问的所有PostgreSQL上安装pgpool_regclass函数，因为它是pgpool-II内部使用的。如果没有这个，处理不同模式中的重复表名可能会造成麻烦（临时表不是问题）。如果您使用的是PostgreSQL 9.4或更高版本，则不需要安装pgpool_regclass，因为PostgreSQL核心中包含了等效的（to_regclass）。

```shell
$ cd pgpool-II-4.5.4/src/sql/pgpool-regclass
$ make
$ make install
```

在此之后：

```shell
$ psql template1
=# CREATE EXTENSION pgpool_regclass;
```

或者

```shell
$ psql -f pgpool-regclass.sql template1
```

应在通过pgpool-II访问的每个数据库上执行CREATE EXTENSION或pgpool-regclass.sql。但是，对于在执行CREATE EXTENSION或psql-f pgpool-regclass.sql template1后创建的数据库，您不需要这样做，因为此模板数据库将被克隆以创建新数据库。

## 2.8. 创建insert_lock表

如果不打算使用本机复制模式或快照隔离模式，可以跳过本节。

如果您计划使用本机复制模式或快照隔离模式和insert_lock，强烈建议创建pgpool_catalog.insert_lock表进行互斥。如果没有这个，insert_lock到目前为止仍然有效。然而，在这种情况下，Pgpool-II会锁定插入目标表。这种行为与VACUUM的表锁冲突相同，因此INSERT处理可能会因此等待很长时间。

```shell
$ cd pgpool-II-4.5.4/src/sql
$ psql -f insert_lock.sql template1
```

应在通过Pgpool-II访问的每个数据库上执行insert_lock.sql。对于执行psql-f insert_lock.sql template1后创建的数据库，您不需要这样做，因为此模板数据库将被克隆以创建新数据库。

## 2.9. 编译和安装文档

### 2.9.1. 工具集

Pgpool-II文档是用SGML（更确切地说，DocBook，一种使用SGML实现的语言）编写的。要生成可读的HTML文档，您需要使用docbook工具对其进行编译。要在RHEL或类似系统上安装Docbook工具，请使用：

```shell
dnf install --enablerepo=powertools docbook-dtds docbook-style-dsssl docbook-style-xsl libxslt openjade
```

### 2.9.2. 编译文档

一旦工具集安装在系统上，您就可以编译文档：

```shell
$ cd doc
$ make
$ cd ..
$ cd doc.ja
$ make
```

您将在doc/src/sgml/HTML下看到英文HTML文档，在sgml/man[1-8]下看到在线文档。日语文档可以在doc.ja/src/sgml/html下找到，在线文档可以在sgml/man[1-8]下找到。

## 2.10. 从RPM安装

本章介绍如何从RPM安装Pgpool-II。如果要从源代码安装，请检查第2.2节。

Pgpool-II社区为RHEL9/8/7和与RHEL兼容的操作系统提供RPM包。您可以从官方Pgpool-II存储库下载包文件。

Pgpool-II官方存储库包含以下软件包：

表2-1 Pgpool II RPM软件包

| 包                                  | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| pgpool-II-pgXX                      | 运行Pgpool-II所需的库和二进制文件                            |
| pgpool-II-pgXX-extensions           | 此软件包必须安装在所有PostgreSQL服务器上才能使用在线恢复功能 |
| pgpool-II-pgXX-debuginfo            | 调试符号用于调试                                             |
| pgpool-II-pgXX-debugsource          | 仅适用于RHEL8/9。调试符号用于调试                            |
| pgpool-II-pgXX-extensions-debuginfo | 仅适用于RHEL8/9。调试符号用于调试                            |
| pgpool-II-pgXX-devel                | 开发人员的头文件                                             |

Pgpool-II需要PostgreSQL的库和扩展目录。由于特定PostgreSQL版本的目录路径不同，Pgpool-II为每个PostgreSQL版本提供了单独的包。上面包中的“XX”是一个代表PostgreSQL版本的两位数。选择与PostgreSQL版本相对应的Pgpool II RPM。（例如，如果你使用的是PostgreSQL 16，你需要安装pgpool-II-pg16）

### 2.10.1. 安装前

由于PostgreSQL YUM存储库中也包含了与Pgpool-II相关的包，如果已经安装了PostgreSQL存储库包，请将“exclude”设置添加到/etc/YUM.repos.d/pgdg-redhat-all.repo中，这样Pgpool-II就不会从PostgreSQL YUM仓库安装。如果Pgpool-II和PostgreSQL安装在不同的服务器上，您可以跳过本节。

```shell
vi /etc/yum.repos.d/pgdg-redhat-all.repo
```

以下是/etc/yum.repos.d/pgdg-redhat-all.repo的设置示例。

```shell
[pgdg-common]
...
exclude=pgpool*

[pgdg16]
...
exclude=pgpool*

[pgdg15]
...
exclude=pgpool*

[pgdg14]
...
exclude=pgpool*

[pgdg13]
...
exclude=pgpool*

[pgdg12]
...
exclude=pgpool*

[pgdg11]
...
exclude=pgpool*
```

### 2.10.2. 安装RPM

在这里，我们使用Pgpool-II官方YUM存储库安装Pgpool-II。

以下命令假设您在RHEL8上为PostgreSQL 16使用Pgpool-II 4.5.x。如果您使用的是其他版本，请将“pgXX”替换为您的PostgreSQL版本。

首先，安装与您的Pgpool-II版本和发行版相对应的存储库。有关REHL7/9，请参阅此处。

```shell
dnf install https://www.pgpool.net/yum/rpms/4.5/redhat/rhel-8-x86_64/pgpool-II-release-4.5-1.noarch.rpm
```

然后，安装Pgpool-II。

```shell
dnf install pgpool-II-pg16
```

要使用在线恢复功能，请在所有PostgreSQL服务器上安装pgpool-II-pg16-extensions。因为pgpool-II的pgXX扩展依赖于pgpool-II pgXX包，如果pgpool-II和PostgreSQL安装在单独的服务器上，pgpool-II pgXX也需要安装在PostgreSQL服务器上。

注意：pgpool-II pgXX扩展需要安装在PostgreSQL服务器上。如果Pgpool-II和PostgreSQL安装在单独的服务器上，则不需要将其安装在运行Pgpool-II的服务器上。

```shell
dnf install pgpool-II-pg16-extensions pgpool-II-pg16
```

如果需要，您可以为开发人员安装debuginfo和devel包。

```shell
dnf install pgpool-II-pg16-debuginfo pgpool-II-pg16-devel
```

### 2.10.3. 配置Pgpool-II

所有Pgpool-II配置文件都安装在/etc/Pgpool-II中。请参阅第3.3节，了解如何设置配置文件。

### 2.10.4. 启动/停止Pgpool-II

在RHEL9/8/7上，如果设置了Pgpool-II的自动启动，请执行一次此操作。

```shell
systemctl enable pgpool.service
```

在此之后，要启动Pgpool-II，请运行以下命令或重新启动整个系统。请注意，PostgreSQL服务器必须在此之前启动。

```shell
systemctl start pgpool.service
```

要停止Pgpool-II，请执行一次。请注意，在停止PostgreSQL之前，必须先停止Pgpool-II。

```shell
systemctl stop pgpool.service
```

在此之后，您可以停止PostgreSQL服务器。

## 2.11. 安装提示

本章收集了安装Pgpool-II的随机提示。

### 2.11.1. 防火墙

当Pgpool-II连接到其他Pgpool-II服务器或PostgreSQL服务器时，必须通过启用防火墙管理软件来访问目标端口。

首先，允许访问Pgpool-II使用的端口。在下面的示例中，设Pgpool-II侦听端口号为9999，PCP侦听端口号是9898，看门狗侦听端口号也是9000，心跳侦听端口号就是9694。请注意，只有心跳端口使用UDP，其他端口使用TCP。

```shell
firewall-cmd --permanent --zone=public --add-port=9999/tcp --add-port=9898/tcp --add-port=9000/tcp
firewall-cmd --permanent --zone=public --add-port=9694/udp
firewall-cmd --reload
```

以下是需要访问PostgreSQL时CentOS/RHEL7的示例。

```shell
firewall-cmd --permanent --zone=public --add-service=postgresql
firewall-cmd --reload
```

“postgresql”是分配给postgresql的服务名称。服务名称列表可以通过以下方式获得：

```shell
firewall-cmd --get-services
```

请注意，您可以在/usr/lib/firewalld/services/中定义自己的服务名称。

如果PostgreSQL正在监听11002端口，而不是标准的5432端口，你可以这样做：

```shell
firewall-cmd --zone=public --remove-service=postgresql --permanent
firewall-cmd --zone=public --add-port=11002/tcp --permanent
firewall-cmd --reload
```

