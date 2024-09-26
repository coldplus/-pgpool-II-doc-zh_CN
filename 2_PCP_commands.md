# PCP 命令

本部分包含PCP命令的参考信息。PCP命令是通过网络操纵pgpool-II的UNIX命令。请注意，自pgpool-II 3.5以来，所有PCP命令的参数格式都已更改。

# 1. PCP 连接认证

PCP user names and passwords must be declared in `pcp.conf` in `$prefix/etc` directory (查看 [Section 3.2](https://www.pgpool.net/docs/latest/en/html/configuring-pcp-conf.html) 了解如何创建文件). `-F` option can be used when starting Pgpool-II if `pcp.conf` is placed somewhere else.

# 2. PCP password 文件

如果没有为pcp连接指定密码，则用户主目录中的文件“.pcppass”或环境变量PCPPASSFILE引用的文件可以包含要使用的密码（在Pgpool II 4.4或之前，需要“-w/--no password”选项）。请参阅[pcp_com_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html)了解更多详情。

此文件应包含以下格式的行：

```
hostname:port:username:password
```

(You can add a reminder comment to the file by copying the line above and preceding it with #.) Each of the first three fields can be a literal value, or *, which matches anything. The password field from the first line that matches the current connection parameters will be used. (Therefore, put more-specific entries first when you are using wildcards.) If an entry needs to contain : or \, escape this character with \. The hostname field is matched to the host connection parameter if that is specified, if the host parameter is not given then the host name `localhost` is searched for. The host name `localhost` is also searched for when the connection is a Unix domain socket connection and the host parameter matches the default pcp socket directory path.

`.pcppass`上的权限必须禁止对世界或组的任何访问；通过命令`chmod 0600 ~/.pcppass`来实现这一点。如果权限没有这么严格，文件将被忽略。

- **目录**
- [pcp_common_options](#pcp_common_options) -- common options used in PCP commands
- [pcp_node_count](#pcp_node_count) -- displays the total number of database nodes
- [pcp_node_info](#pcp_node_info) -- displays the information on the given node ID
- [pcp_health_check_stats](#pcp_health_check_stats) -- displays health check statistics data on given node ID
- [pcp_watchdog_info](#pcp_watchdog_info) -- displays the watchdog status of the Pgpool-II
- [pcp_proc_count](#pcp_proc_count) -- displays the list of Pgpool-II children process IDs
- [pcp_proc_info](#pcp_proc_info) -- displays the information on the given Pgpool-II child process ID
- [pcp_pool_status](#pcp_pool_status) -- displays the parameter values as defined in `pgpool.conf`
- [pcp_detach_node](#pcp_detach_node) -- detaches the given node from Pgpool-II. Existing connections to Pgpool-II are forced to be disconnected.
- [pcp_attach_node](#pcp_attach_node) -- attaches the given node to Pgpool-II.
- [pcp_promote_node](#pcp_promote_node) -- promotes the given node as new main to Pgpool-II
- [pcp_stop_pgpool](#pcp_stop_pgpool) -- terminates the Pgpool-II process
- [pcp_reload_config](#pcp_reload_config) -- 重新加载 pgpool-II 配置文件
- [pcp_recovery_node](#pcp_recovery_node) -- 使用恢复功能附加给定的后端节点

# pcp_common_options

## Name

pcp_common_options -- common options used in PCP commands

## Synopsis

`pcp_command` [`*option*`...]

## Description

所有PCP命令都有一些共同的参数。其中大部分是用于身份验证的，其余的是关于详细模式、调试消息等。

## Options

- `-h *hostname*` `--host=*hostname*`

  运行服务器的计算机的主机名。如果该值以斜线开头，则将其用作Unix域套接字的目录。

- `-p *port*` `--port=*port*`

  PCP 端口号 (default:"9898").

- `-U *username*` `--username=*username*`

  PCP 认证用户名 (default: OS user name).

- `-w` `--no-password`

  不提示输入密码（自动的）。如果“.pcppass”文件没有密码，则连接尝试将失败。此选项在没有用户输入密码的批处理作业和脚本中非常有用。

- `-W` `--password`

  强制密码提示。

- `-d` `--debug`

  启用调试消息。

- `-v` `--verbose`

  启用详细输出。

- `-V` `--version`

  打印命令版本，然后退出。

- `-?` `--help`

  显示命令行参数的帮助，然后退出。

## Environment

- `PCPPASSFILE`

  指定pcp密码文件的路径。

# pcp_node_count

## Name

pcp_node_count -- displays the total number of database nodes

## Synopsis

`pcp_node_count` [`*option*`...]

## Description

`pcp_node_count` displays the total number of database nodes defined in `pgpool.conf`. It does not distinguish between nodes status, ie attached/detached. ALL nodes are counted.

## Options

See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

## Example

Here is an example output:

```shell
$ pcp_node_count -p 11001
Password:
2
```

# pcp_node_info

## Name

pcp_node_info -- displays the information on the given node ID

## Synopsis

`pcp_node_info` [`*option*`...] [`*node_id*`]

## Description

`pcp_node_info` displays the information on the given node ID.

## Options

- `-n *node_id*` `--node-id=*node_id*`

  The index of backend node to get information of.

- `-a` `--all`

  Display all backend nodes information.

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

## Example

Here is an example output:

```sql
$ pcp_node_info -w -p 11001 -n 1
/tmp 11003 1 0.500000 waiting up standby standby 0 streaming async 2021-02-27 14:51:30
```

The result is in the following order:

```
1. hostname
2. port number
3. status
4. load balance weight
5. status name
6. actual backend status (obtained using PQpingParams. Pgpool-II 4.3 or later)
7. backend role
8. actual backend role (obtained using pg_is_in_recovery. Pgpool-II 4.3 or later)
9. replication delay
10. replication state (taken from pg_stat_replication. Pgpool-II 4.1 or later)
11. sync replication state (taken from pg_stat_replication. Pgpool-II 4.1 or later)
12. last status change time
```

3 (status) is represented by a digit from [0 to 3].

- 0 - This state is only used during the initialization. PCP will never display it.
- 1 - Node is up. No connections yet.
- 2 - Node is up. Connections are pooled.
- 3 - Node is down.

4 (load balance weight) is displayed in normalized format (0 - 1).

6 shows the backend status in real time. The info is obtained by calling `PQpingParams` at the time when the command is invoked. `PQpingParams` is only available in PostgreSQL 9.1 or later. If Pgpool-II was built with PostgreSQL 9.0 or earlier, the column shows "unknown". Also if [health check](https://www.pgpool.net/docs/latest/en/html/runtime-config-health-check.html) is disabled, it shows "unknown". When a backend node is detached by [pcp_detach_node](https://www.pgpool.net/docs/latest/en/html/pcp-detach-node.html), the status managed by Pgpool-II will be "down", while the actual backend status is "up". Thus it is possible that 5 does not match with 6. However it should not happen that 5 is "up" while 6 is "down".

8 shows the backend status in real time. The result will be either "primary" or "standby", and possibly "unknown" if information retrieval failed. Since Pgpool-IIsearches backend nodes in the node id order and assumes the last found node is primary, it is possible that 7 does not match 8 when there are multiple nodes that are not standby by erroneous operations (this command is useful to find such that situation). In other than streaming replication mode, the status will be either "main" or "replica". Unlike streaming replication mode `pg_is_in_recovery` is not called and value for 7 and 8 will be always the same.

To correctly 9, 10, 11 are displayed, [sr_check_period](https://www.pgpool.net/docs/latest/en/html/runtime-streaming-replication-check.html#GUC-SR-CHECK-PERIOD) must not be 0. 10, 11 will not be displayed if [sr_check_user](https://www.pgpool.net/docs/latest/en/html/runtime-streaming-replication-check.html#GUC-SR-CHECK-USER) is not PostgreSQL super user nor it's not in "pg_monitor" group.

> **Note:** To make [sr_check_user](https://www.pgpool.net/docs/latest/en/html/runtime-streaming-replication-check.html#GUC-SR-CHECK-USER) in pg_monitor group, execute following SQL command by PostgreSQL super user (replace "sr_check_user" with the setting of [sr_check_user](https://www.pgpool.net/docs/latest/en/html/runtime-streaming-replication-check.html#GUC-SR-CHECK-USER)):
>
> ```sql
> GRANT pg_monitor TO sr_check_user;
> ```
>
> For PostgreSQL 9.6, there's no pg_monitor group and [sr_check_user](https://www.pgpool.net/docs/latest/en/html/runtime-streaming-replication-check.html#GUC-SR-CHECK-USER) must be PostgreSQL super user.

From Pgpool-II 4.4, 9 (replication delay) is displayed in either bytes or seconds. See [delay_threshold_by_time](https://www.pgpool.net/docs/latest/en/html/runtime-streaming-replication-check.html#GUC-DELAY-THRESHOLD-BY-TIME) for more details.

The `-a` or `--all` option lists all backend nodes information.

```shell
$ pcp_node_info -w -p 11001 -a
/tmp 11002 1 0.500000 waiting up primary primary 0 none none 2021-02-27 14:51:30
/tmp 11003 1 0.500000 waiting up standby standby 0 streaming async 2021-02-27 14:51:30
```

The `--verbose` option can help understand the output. For example:

```shell
$ pcp_node_info -w -p 11001 --verbose 1
Hostname               : /tmp
Port                   : 11003
Status                 : 1
Weight                 : 0.500000
Status Name            : waiting
Backend Status Name    : up
Role                   : standby
Backend Role           : standby
Replication Delay      : 0
Replication State      : streaming
Replication Sync State : async
Last Status Change     : 2021-02-27 14:51:30
```

# pcp_health_check_stats

## Name

pcp_health_check_stats -- displays health check statistics data on given node ID

## Synopsis

`pcp_health_check_stats` [`*option*`...] [`*node_id*`]

## Description

`pcp_health_check_stats` displays health check statistics data on given node ID.

## Options

- `-n *node_id*` `--node-id=*node_id*`

  The index of backend node to get information of.

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

## Example

Here is an example output:

```shell
$ pcp_health_check_stats -h localhost -p 11001 -w 0
0 /tmp 11002 up primary 2020-02-24 22:02:42 3 3 0 0 0 0.000000 0 5 1 3.666667 2020-02-24 22:02:47 2020-02-24 22:02:47
$ pcp_health_check_stats -h localhost -p 11001 -w -v 0
Node Id                       : 0
Host Name                     : /tmp
Port                          : 11002
Status                        : up
Role                          : primary
Last Status Change            : 2020-02-24 22:02:42
Total Count                   : 5
Success Count                 : 5
Fail Count                    : 0
Skip Count                    : 0
Retry Count                   : 0
Average Retry Count           : 0.000000
Max Retry Count               : 0
Max Health Check Duration     : 5
Minimum Health Check Duration : 1
Average Health Check Duration : 4.200000
Last Health Check             : 2020-02-24 22:03:07
Last Successful Health Check  : 2020-02-24 22:03:07
Last Skip Health Check        :
Last Failed Health Check      :
```

See [Table 1](https://www.pgpool.net/docs/latest/en/html/sql-show-pool-health-check-stats.html#HEALTH-CHECK-STATS-DATA-TABLE) for details of data.

# pcp_watchdog_info

## Name

pcp_watchdog_info -- displays the watchdog status of the Pgpool-II

## Synopsis

`pcp_watchdog_info` [`*options*`...] [`*watchdog_id*`]

## Description

`pcp_watchdog_info` displays the information on the given node ID.

## Options

- `-n *watchdog_id*` `--node-id=*watchdog_id*`

  The index of other Pgpool-II to get information for.Index 0 gets one's self watchdog information.If omitted then gets information of all watchdog nodes.

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

## Example

Here is an example output:

```shell
$ pcp_watchdog_info -h localhost -p 9898 -U postgres
Password:
3 3 YES server1:9999 Linux server1.localdomain server1

server1:9999 Linux server1.localdomain server1 9999 9000 4 LEADER 0 MEMBER
server2:9999 Linux server2.localdomain server2 9999 9000 7 STANDBY 0 MEMBER
server3:9999 Linux server3.localdomain server3 9999 9000 7 STANDBY 0 MEMBER
```

The result is in the following order:

```
The first output line describes the watchdog cluster information:

1. Total watchdog nodes in the cluster
2. Total watchdog nodes in the cluster with active membership
3. Local node's escalation status
4. Leader node name
5. Leader node host
```

```
Next is the list of watchdog nodes:

1. node name
2. hostname
3. pgpool port
4. watchdog port
5. current node state
6. current node state name
7. current cluster membership status
8. current cluster membership status name
```

The `--verbose` option can help understand the output. For example:

```shell
$ pcp_watchdog_info -h localhost -p 9898 -U pgpool -v
Password:
Watchdog Cluster Information
Total Nodes              : 3
Remote Nodes             : 2
Member Remote Nodes      : 2
Alive Remote Nodes       : 2
Nodes required for quorum: 2
Quorum state             : QUORUM EXIST
Local node escalation    : YES
Leader Node Name         : server1:9999 Linux server1.localdomain
Leader Host Name         : server1

Watchdog Node Information
Node Name         : server1:9999 Linux server1.localdomain
Host Name         : server1
Delegate IP       : 192.168.56.150
Pgpool port       : 9999
Watchdog port     : 9000
Node priority     : 1
Status            : 4
Status Name       : LEADER
Membership Status : MEMBER

Node Name         : server2:9999 Linux server2.localdomain
Host Name         : server2
Delegate IP       : 192.168.56.150
Pgpool port       : 9999
Watchdog port     : 9000
Node priority     : 1
Status            : 7
Status Name       : STANDBY
Membership Status : MEMBER

Node Name         : server3:9999 Linux server3.localdomain
Host Name         : server3
Delegate IP       : 192.168.56.150
Pgpool port       : 9999
Watchdog port     : 9000
Node priority     : 1
Status            : 7
Status Name       : STANDBY
Membership Status : MEMBER
```

# pcp_proc_count

## Name

pcp_proc_count -- displays the list of Pgpool-II children process IDs

## Synopsis

`pcp_proc_count` [`*options*`...]

## Description

`pcp_proc_count` displays the list of Pgpool-II children process IDs. If there is more than one process, IDs will be delimited by a white space.

## Options

See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

# pcp_proc_info

## Name

pcp_proc_info -- displays the information on the given Pgpool-II child process ID

## Synopsis

`pcp_proc_info` [`*options*`...] [`*pgpool_child_processid*`]

## Description

`pcp_proc_info` displays the information on the given Pgpool-II child process ID.

## Options

- `-a` `--all`

  Display all child processes and their available connection slots.

- `-P *PID*` `--process-id=*PID*`

  PID of Pgpool-II child process.

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

If -a nor -P is not specified, process information of all connected Pgpool-II child process will be printed. In this case if there's no connected Pgpool-II child process, nothing but "No process information available" message will be printed.

## Example

Here is an example output:

```shell
$ pcp_proc_info -p 1100
test postgres 2021-09-28 04:16:00 (4:56 before process restarting) 1 3 0 2021-09-28 04:16:16 2021-09-28 04:16:16 0 2021-09-28 04:16:33 1 30795 0 30750 0 Wait for connection
test postgres 2021-09-28 04:16:00 (4:56 before process restarting) 1 3 0 2021-09-28 04:16:16 2021-09-28 04:16:16 0 2021-09-28 04:16:33 1 30796 0 30750 1 Wait for connection
test kawamoto 2021-09-28 04:16:00 0 3 0 2021-09-28 04:16:03 2021-09-28 04:16:03 34 (4:26 before client disconnected)  1 30763 1 30751 0 Idle
test kawamoto 2021-09-28 04:16:00 0 3 0 2021-09-28 04:16:03 2021-09-28 04:16:03 34 (4:26 before client disconnected)  1 30764 1 30751 1 Idle
$ pcp_proc_info -p 11001 30751
test kawamoto 2021-09-28 04:16:00 0 3 0 2021-09-28 04:16:03 2021-09-28 04:16:03 58 (4:02 before client disconnected)  1 30763 1 30751 0 Idle
test kawamoto 2021-09-28 04:16:00 0 3 0 2021-09-28 04:16:03 2021-09-28 04:16:03 58 (4:02 before client disconnected)  1 30764 1 30751 1 Idle
```

The result is in the following order:

```
1. connected database name
2. connected user name
3. process start-up timestamp (If child_life_time is set not 0, the time before process restarting is displayed.)
4. process-reuse counter for child_max_connections
5. protocol major version
6. protocol minor version
7. connection created timestamp
8. last client connected timestamp
9. client idle duration (sec) (If client_idle_limit is set not 0, the time before client disconnected is displayed.)
10. last client disconnected timestamp
11. connection-reuse counter
12. PostgreSQL backend process id
13. 1 if frontend connected 0 if not
14. pgpool child process id
15. PostgreSQL backend id
16. process status
17. 1 if backend is load balance node and frontend connected, 0 otherwise
```

If `-a` or `--all` option is not specified and there is no connection to the backends, nothing will be displayed. If there are multiple connections, one connection's information will be displayed on each line multiple times. Timestamps are displayed in EPOCH format.

The `--verbose` option can help understand the output. For example:

```shell
$ pcp_proc_info -p 11001 --verbose
Database                  : test
Username                  : postgres
Start time                : 2021-09-28 04:16:00 (2:52 before process restarting)
Client connection count   : 1
Major                     : 3
Minor                     : 0
Backend connection time   : 2021-09-28 04:16:16
Client connection time    : 2021-09-28 04:16:16
Client idle duration      : 0
Client disconnection time : 2021-09-28 04:16:33
Pool Counter              : 1
Backend PID               : 30795
Connected                 : 0
PID                       : 30750
Backend ID                : 0
Status                    : Wait for connection
Load balance node         : 0

Database                  : test
Username                  : postgres
Start time                : 2021-09-28 04:16:00 (2:52 before process restarting)
Client connection count   : 1
Major                     : 3
Minor                     : 0
Backend connection time   : 2021-09-28 04:16:16
Client connection time    : 2021-09-28 04:16:16
Client idle duration      : 0
Client disconnection time : 2021-09-28 04:16:33
Pool Counter              : 1
Backend PID               : 30796
Connected                 : 0
PID                       : 30750
Backend ID                : 1
Status                    : Wait for connection
Load balance node         : 0

Database                  : test
Username                  : kawamoto
Start time                : 2021-09-28 04:16:00
Client connection count   : 0
Major                     : 3
Minor                     : 0
Backend connection time   : 2021-09-28 04:16:03
Client connection time    : 2021-09-28 04:16:03
Client idle duration      : 158 (2:22 before client disconnected)
Client disconnection time :
Pool Counter              : 1
Backend PID               : 30763
Connected                 : 1
PID                       : 30751
Backend ID                : 0
Status                    : Idle
Load balance node         : 1

Database                  : test
Username                  : kawamoto
Start time                : 2021-09-28 04:16:00
Client connection count   : 0
Major                     : 3
Minor                     : 0
Backend connection time   : 2021-09-28 04:16:03
Client connection time    : 2021-09-28 04:16:03
Client idle duration      : 158 (2:22 before client disconnected)
Client disconnection time :
Pool Counter              : 1
Backend PID               : 30764
Connected                 : 1
PID                       : 30751
Backend ID                : 1
Status                    : Idle
Load balance node         : 1
```

# pcp_pool_status

## Name

pcp_pool_status -- displays the parameter values as defined in `pgpool.conf`

## Synopsis

`pcp_pool_status` [`*options*`...]

## Description

`pcp_pool_status` displays the parameter values as defined in `pgpool.conf`.

## Options

See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

## Example

Here is an example output:

```shell
$ pcp_pool_status -h localhost -U postgres
name : listen_addresses
value: localhost
desc : host name(s) or IP address(es) to listen to

name : port
value: 9999
desc : pgpool accepting port number

name : socket_dir
value: /tmp
desc : pgpool socket directory

name : pcp_port
value: 9898
desc : PCP port # to bind
```

# pcp_detach_node

## Name

pcp_detach_node -- detaches the given node from Pgpool-II. Existing connections to Pgpool-II are forced to be disconnected.

## Synopsis

`pcp_detach_node` [`*options*`...] [`*node_id*`] [`*gracefully*`]

## Description

`pcp_detach_node` detaches the given node from Pgpool-II. If [failover_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FAILOVER-COMMAND) and/or [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) are specified, they are executed too. Existing connections to Pgpool-II are forced to be disconnected.

`pcp_detach_node` just detaches the node, and does not touch running backend behind the node. This command is useful when admin needs to maintain the PostgreSQLnode. He/she can shutdown or stop the backend as many times as he/she wants.

The safest way to re-attach the detached node is, stopping the backend and apply [pcp_recovery_node](https://www.pgpool.net/docs/latest/en/html/pcp-recovery-node.html). However if you are sure that there's no replication delay (or the delay will be recovered later on) and the role of the node (primary/standby) will not be changed, you can use [pcp_attach_node](https://www.pgpool.net/docs/latest/en/html/pcp-attach-node.html).

## Options

- `-n *node_id*` `--node_id=*node_id*`

  The index of backend node to detach.

- `-g` `--gracefully`

  wait until all clients are disconnected (unless client_idle_limit_in_recovery is -1 or recovery_timeout is expired).

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

# pcp_attach_node

## Name

pcp_attach_node -- attaches the given node to Pgpool-II.

## Synopsis

`pcp_attach_node` [`*options*`...] [`*node_id*`]

## Description

`pcp_attach_node` attaches the given node to Pgpool-II.

## Options

- `-n *node_id*` `--node_id=*node_id*`

  The index of backend node to attach.

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

# pcp_promote_node

## Name

pcp_promote_node -- promotes the given node as new main to Pgpool-II

## Synopsis

`pcp_promote_node` [`*options*`...] [`*node_id*`] [`*gracefully*`] [`*switchover*`]

## Description

`pcp_promote_node` promotes the given node as new primary to Pgpool-II. In streaming replication mode only. Please note that this command does not actually promote standby PostgreSQL backend unless `switchover` option is specified: it just changes the internal status of Pgpool-II and trigger failover and users have to promote standby PostgreSQL outside Pgpool-II.

If `switchover` is specified, Pgpool-II detaches current primary (changes the internal status to down) and execute the [failover_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FAILOVER-COMMAND), with the new main node argument to be set to the specified node id. Because most failover scripts promote the new main node, the specified node will be the new primary node. The [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) is necessary to be set properly to turn the former primary into standby.

`pcp_promote_node` executes followings if `switchover` is not specified. Please be warned that if [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) is set, the command will be executed. It is a standard advice that you disable [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) before executing this command.

1. Change the status of standby PostgreSQL from standby to primary. It just changes the internal status of Pgpool-II and it does not actually promote PostgreSQLstandby server.
2. Change the status of PostgreSQL node which is not specified by this command's argument to down. It just changes the internal status of Pgpool-II and it does not actually make PostgreSQL standby server down.
3. If [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) is set, execute [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) against PostgreSQL.

`pcp_promote_node` executes followings if `switchover` is specified. If [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) is set, the command will be executed. You need to set [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) before executing this command because failover script will create the new primary and other nodes need to be turned into standbys

1. Change the status of primary PostgreSQL from up to down. This triggers [failover_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FAILOVER-COMMAND) execution, with the new main node argument to be set to the specified node id. Because most failover scripts promote the new main node, the specified node will become the new primary node.
2. Change the status of standby PostgreSQL node which is not specified by this command's argument to down. It just changes the internal status of Pgpool-II and it does not actually make PostgreSQL standby server down.
3. If [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) is set, execute [follow_primary_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FOLLOW-PRIMARY-COMMAND) against PostgreSQL.

## Options

- `-n *node_id*` `--node-id=*node_id*`

  The index of backend node to promote as new main. The specified node must be in "up" or "waiting" status.

- `-g ` `--gracefully`

  Wait until all clients are disconnected (unless client_idle_limit_in_recovery is -1 or recovery_timeout is expired).

- `-s ` `--switchover`

  Let the specified node to be actually promoted by triggering the [failover_command](https://www.pgpool.net/docs/latest/en/html/runtime-config-failover.html#GUC-FAILOVER-COMMAND). Also change the current primary node status to down.

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

# pcp_stop_pgpool

## Name

pcp_stop_pgpool -- terminates the Pgpool-II process

## Synopsis

`pcp_stop_pgpool` [`*options*`...] [`*mode*`]

## Description

`pcp_stop_pgpool` terminates the Pgpool-II process.

## Options

- `-m *mode*` `--mode=*mode*`

  Shutdown mode for terminating the Pgpool-II process.The available modes are as follows (The default is "smart"):s, smart : smart mode f, fast : fast mode i, immediate : immediate mode Regarding the meaning of each mode, please refer to [pgpool](https://www.pgpool.net/docs/latest/en/html/pgpool.html) manual.

- `-s *scope*` `--scope=*scope*`

  Specifies the breadth of a command's impact.The supported command scopes are as follows (The default is "local"):c, cluster : terminates all Pgpool-II nodes part of the cluster l, local : terminates local Pgpool-II node only

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

# pcp_reload_config

## Name

pcp_reload_config -- 重新加载 pgpool-II 配置文件

## Synopsis

`pcp_reload_config` [`*options*`...]

## Description

`pcp_reload_config` reload Pgpool-II config file.

## Options

- `-s *scope*` `--scope=*scope*`

  Specifies the breadth of a command's impact.The supported command scopes are as follows (The default is "local"):c, cluster : reload config files of all Pgpool-II nodes part of the cluster l, local : reload config file of local Pgpool-II node only

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).

# pcp_recovery_node

## Name

pcp_recovery_node -- 使用恢复功能附加给定的后端节点

## Synopsis

`pcp_recovery_node` [`*options*`...] [`*node_id*`]

## Description

`pcp_recovery_node` 使用恢复功能附加给定的后端节点。 See [Section 5.11](https://www.pgpool.net/docs/latest/en/html/runtime-online-recovery.html) to set up necessary parameters of pgpool.conf.

## Options

- `-n *node_id*` `--node-id=*node_id*`

  The index of backend node.

- `Other options `

  See [pcp_common_options](https://www.pgpool.net/docs/latest/en/html/pcp-common-options.html).