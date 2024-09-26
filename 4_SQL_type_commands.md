# SQL类型命令

本部分包含各种SQL类型Pgpool-II命令的参考信息。这些命令可以在SQL会话中使用标准的PostgreSQL客户端（如psql）发出。它们不会被转发到后端数据库：而是由Pgpool-II服务器处理。请注意，SQL类型的命令不能在扩展查询模式下使用。您将收到PostgreSQL的解析错误。

**目录**

- [PGPOOL SHOW](#PGPOOL SHOW) -- 显示配置参数的值
- [PGPOOL SET](#PGPOOL SET) -- 更改配置参数
- [PGPOOL RESET](#PGPOOL RESET) --将配置参数的值还原为默认值
- [SHOW POOL STATUS](#SHOW POOL STATUS) -- 显示配置参数列表及其名称、值和描述
- [SHOW POOL NODES](#SHOW POOL NODES) -- 显示所有已配置节点的列表
- [SHOW POOL_PROCESSES](#SHOW POOL_PROCESSES) -- 显示所有等待连接和处理连接的Pgpool-II进程的列表
- [SHOW POOL_POOLS](#SHOW POOL_POOLS) -- 返回由Pgpool-II处理的池列表
- [SHOW POOL_VERSION](#SHOW POOL_VERSION) -- 显示一个包含Pgpool-II版本号的字符串
- [SHOW POOL_CACHE](#SHOW POOL_CACHE) -- 显示缓存存储统计信息
- [SHOW POOL_HEALTH_CHECK_STATS](#SHOW POOL_HEALTH_CHECK_STATS) -- 显示健康检查统计数据
- [SHOW POOL_BACKEND_STATS](#SHOW POOL_BACKEND_STATS) -- 显示后端SQL命令统计信息

# PGPOOL SHOW

## 名字

PGPOOL SHOW -- 显示配置参数的值

## 简介

```shell
PGPOOL SHOW configuration_parameter
PGPOOL SHOW configuration_parameter_group
PGPOOL SHOW ALL
```

## 描述

PGPOOL SHOW将显示PGPOOL-II配置参数的当前值。此命令类似于PostgreSQL中的show命令，但添加了PGPOOL关键字以将其与PostgreSQL SHOW命令区分开来。

## 参数

- configuration_parameter

	Pgpool-II配置参数的名称。可用参数记录在第5章中。

	> [!CAUTION]
	> 注意：如果参数包含大写字母（例如delegate_ip），则参数名称必须括在双引号中。

- configuration_parameter_group

	Pgpool-II配置参数组的名称。目前有四个参数组。

	- backend

		所有后端配置参数的配置组。

	- watchdog

		所有监视器节点配置参数的配置组。

	- heartbeat

		所有监视器心跳配置参数的配置组。

	- health_check

		所有健康检查参数的配置组。

- ALL

	显示所有配置参数的值，并附上说明。

## 示例

显示参数端口的当前设置：

```sql
PGPOOL SHOW port;
port
------
9999
(1 row)
```

显示参数write_function_list的当前设置：

```sql
PGPOOL SHOW write_function_list;
write_function_list
---------------------
nextval,setval
(1 row)
```

显示属于后端组的所有配置参数的当前设置：

```sql
PGPOOL SHOW backend;
item                     |          value          |              description
-------------------------+-------------------------+-----------------------------------------------------------
backend_hostname0        | 127.0.0.1               | hostname or IP address of PostgreSQL backend.
backend_port0            | 5434                    | port number of PostgreSQL backend.
backend_weight0          | 0                       | load balance weight of backend.
backend_data_directory0  | /var/lib/pgsql/data     | data directory of the backend.
backend_flag0            | ALLOW_TO_FAILOVER       | Controls various backend behavior.
backend_hostname1        | 127.0.0.1               | hostname or IP address of PostgreSQL backend.
backend_port1            | 5432                    | port number of PostgreSQL backend.
backend_weight1          | 1                       | load balance weight of backend.
backend_data_directory1  | /home/work/installed/pg | data directory of the backend.
backend_flag1            | ALLOW_TO_FAILOVER       | Controls various backend behavior.
(10 rows)
```

显示所有设置：

```sql
PGPOOL SHOW ALL;
item                     |          value          |              description
-------------------------+-------------------------+-----------------------------------------------------------
backend_hostname0        | 127.0.0.1               | hostname or IP address of PostgreSQL backend.
backend_port0            | 5434                    | port number of PostgreSQL backend.
backend_weight0          | 0                       | load balance weight of backend.
backend_data_directory0  | /var/lib/pgsql/data     | data directory of the backend.
backend_flag0            | ALLOW_TO_FAILOVER       | Controls various backend behavior.
backend_hostname1        | 127.0.0.1               | hostname or IP address of PostgreSQL backend.
backend_port1            | 5432                    | port number of PostgreSQL backend.
backend_weight1          | 1                       | load balance weight of backend.
backend_data_directory1  | /home/work/installed/pg | data directory of the backend.
backend_flag1            | ALLOW_TO_FAILOVER       | Controls various backend behavior.
hostname0                | localhost               | Hostname of pgpool node for watchdog connection.
.
.
.
ssl                      | off                     | Enables SSL support for frontend and backend connections
(138 rows)
```

## See Also

[PGPOOL SET](#PGPOOL SET)

# PGPOOL SET

## 名字

PGPOOL SET -- 更改配置参数

## 简介

```sql
PGPOOL SET  configuration_parameter { TO | = } { value | 'value' | DEFAULT }
```

## 描述

`PGPOOL SET`命令更改当前会话的PGPOOL-II配置参数的值。此命令类似于PostgreSQL中的`SET`命令，但添加了PGPOOL关键字以将其与PostgreSQL SET命令区分开来。第5章中列出的许多配置参数都可以使用`PGPOOL SET`动态更改，并且只影响当前会话使用的值。

## 示例

更改client_idle_limit参数的值：

```sql
PGPOOL SET client_idle_limit = 350;
```

将client_idle_limit参数的值重置为默认值：

```sql
PGPOOL SET client_idle_limit TO DEFAULT;
```

更改log_min_messages参数的值：

```sql
PGPOOL SET log_min_messages TO INFO;
```

## See Also

[PGPOOL RESET](#PGPOOL RESET), [PGPOOL SHOW](#PGPOOL SHOW)

# PGPOOL RESET

## 名字

PGPOOL RESET -- 将配置参数的值还原为默认值

## 简介

```sql
PGPOOL RESET configuration_parameter
PGPOOL RESET ALL
```

## 描述

`PGPOOL RESET`命令将PGPOOL-II配置参数的值恢复为默认值。默认值被定义为如果在当前会话中没有为参数发出`PGPOOL SET`，则参数将具有的值。此命令类似于[`RESET`](https://www.postgresql.org/docs/current/static/sql-reset.html)PostgreSQL中的命令，并添加`PGPOOL`关键字，以将其与PostgreSQL RESET命令区分开来。	

## 参数

- configuration_parameter

  可设置的Pgpool-II配置参数的名称。可用参数记录在[第5章](https://www.pgpool.net/docs/latest/en/html/runtime-config.html)中.

- ALL

  将所有可设置的Pgpool-II配置参数重置为默认值。

## 示例

重置[client_idle_limit](https://www.pgpool.net/docs/latest/en/html/runtime-config-connection-pooling.html#GUC-CLIENT-IDLE-LIMIT)值的参数：

```sql
PGPOOL RESET client_idle_limit;
```

将所有参数的值重置为默认值：

```sql
PGPOOL RESET ALL;
```

## See Also

[PGPOOL SET](#PGPOOL SET), [PGPOOL SHOW](#PGPOOL SHOW)

# SHOW POOL STATUS

## 名字

SHOW POOL STATUS -- 显示配置参数列表及其名称、值和描述

## 简介

```sql
SHOW POOL_STATUS
```

## 描述

`SHOW POOL_STATU`显示Pgpool-II配置参数的当前值。

此命令类似于[PGPOOL SHOW](https://www.pgpool.net/docs/latest/en/html/sql-pgpool-show.html)命令，但这是它的旧版本。建议使用[PGPOOL SHOW](https://www.pgpool.net/docs/latest/en/html/sql-pgpool-show.html)代替。

## See Also

[PGPOOL SHOW](#PGPOOL SHOW)

# SHOW POOL NODES

## 名字

SHOW POOL_NODES -- 显示所有已配置节点的列表

## 简介

```sql
SHOW POOL_NODES  
```

## 描述

`SHOW POOL_NODES`显示节点id、主机名、端口、状态、权重（仅在使用负载平衡模式时有意义）、角色、向每个后端发出的SELECT查询计数、每个节点是否是负载平衡节点、复制延迟（仅在流式复制模式下）和最后一次状态更改时间。除此之外，Pgpool-II 4.1或更高版本中还显示了备用节点的复制状态和同步状态。Pgpool-II 4.3或更高版本中还显示了实际节点状态和节点角色。

状态列中的值在[pcp_node_info](https://www.pgpool.net/docs/latest/en/html/pcp-node-info.html)中进行了说明。

如果主机名类似于“/tmp”，则意味着Pgpool-II正在使用UNIX域套接字连接到后端。SELECT计数不包括Pgpool-II使用的内部查询。启动Pgpool-II后，计数器也重置为零。最后一次状态更改时间最初设置为Pgpool-II启动的时间。之后，每当“status”或“role”发生变化时，它都会更新。

以下是一个示例会话：

```sql
test=# show pool_nodes;
 node_id | hostname | port  | status | pg_status | lb_weight |  role   | pg_role | select_cnt | load_balance_node | replication_delay | replication_state | replication_sync_state | last_status_change  
---------+----------+-------+--------+-----------+-----------+---------+---------+------------+-------------------+-------------------+-------------------+------------------------+---------------------
 0       | /tmp     | 11002 | up     | up        | 0.500000  | primary | primary | 0          | false             | 0                 |                   |                        | 2021-02-27 15:10:19
 1       | /tmp     | 11003 | up     | up        | 0.500000  | standby | standby | 0          | true              | 0                 | streaming         | async                  | 2021-02-27 15:10:19
(2 rows)
```

# SHOW POOL_PROCESSES

## 名字

SHOW POOL_PROCESSES -- 显示所有等待连接和处理连接的Pgpool-II进程的列表

## 简介

```sql
SHOW POOL_PROCESSES
```

## 描述

`SHOW POOL_PROCESSES`返回所有等待连接和处理连接的Pgpool II进程的列表。

它有8列：

- `pool_pid`是显示的Pgpool-II进程的pid。
- `start_time`是启动此进程的时间戳。
  - 如果[child_life_time](https://www.pgpool.net/docs/latest/en/html/runtime-config-connection-pooling.html#GUC-CHILD-LIFE-TIME)设置为非0，显示流程重新启动前的时间。
- `client_connection_count`统计客户端使用此进程的次数。
- `database`是此进程当前活动后端的数据库名称。
- `username`是此进程当前活动后端连接中使用的用户名。
- `backend_connection_time`是连接的创建时间和日期。
- `pool_counter`统计客户端使用此连接池（进程）的次数。
- `status`是此进程的当前状态。可能的值有：
  - `Execute command`:执行命令。
  - `Idle`:进程正在等待新的客户端命令。
  - `Idle`:进程正在等待事务中的新客户端命令。
  - `Wait for connection`: 进程正在等待新的客户端连接。

以下是一个示例会话：

```sql
test=# show pool_processes;
 pool_pid |                      start_time                      | client_connection_count | database | username | backend_connection_time | pool_counter |       status
----------+------------------------------------------------------+-------------------------+----------+----------+-------------------------+--------------+---------------------
 32641    | 2021-09-28 04:40:45                                  | 0                       |          |          |                         |              | Wait for connection
 32642    | 2021-09-28 04:40:45                                  | 0                       |          |          |                         |              | Wait for connection
 32643    | 2021-09-28 04:40:45                                  | 0                       | test     | kawamoto | 2021-09-28 04:40:48     | 1            | Idle
 32644    | 2021-09-28 04:40:45                                  | 0                       | test     | kawamoto | 2021-09-28 04:43:15     | 1            | Execute command
 32645    | 2021-09-28 04:40:45                                  | 0                       |          |          |                         |              | Wait for connection
 32646    | 2021-09-28 04:40:45                                  | 0                       |          |          |                         |              | Wait for connection
 32647    | 2021-09-28 04:40:45                                  | 0                       |          |          |                         |              | Wait for connection
 32648    | 2021-09-28 04:40:45 (3:15 before process restarting) | 2                       |          |          |                         |              | Wait for connection
(8 rows)
```

# SHOW POOL_POOLS

## Name

SHOW POOL_POOLS -- 返回由Pgpool-II处理的池列表。

## 简介

```sql
SHOW POOL_POOLS
```

## 描述

`SHOW POOL_POOLS`返回由Pgpool-II处理的池列表

它有17列：

- `pool_pid`是显示的Pgpool-II进程的pid。
- `start_time`是启动此进程的时间戳。
  - 如果[child_life_time](https://www.pgpool.net/docs/latest/en/html/runtime-config-connection-pooling.html#GUC-CHILD-LIFE-TIME)设置为非0，显示流程重新启动前的时间。
- `client_connection_count`统计客户端使用此进程的次数。
- `pool_id`是池标识符（应介于0和[max_pool](https://www.pgpool.net/docs/latest/en/html/runtime-config-connection-pooling.html#GUC-最大酚)之间-1）
- `backend_id`是后端标识符（应介于0和配置的后端数量减1之间）
- `database`是此进程池id连接的数据库名称。
- `username`是此进程池id连接的用户名。
- `backend_connection_time`是连接的创建时间和日期。
- `client_connection_time`是客户端上次使用此连接的日期。
- `client_disconnection_time`是客户端最后一次断开此连接的日期。
- `client_idle_duration`是客户端处于空闲状态的时间（秒）。
  - 如果[client_idle_limit](https://www.pgpool.net/docs/latest/en/html/runtime-config-connection-pooling.html#GUC-CLIENT-IDLE-LIMIT)设置为非0，则显示客户端断开连接之前的时间。
- `majorversion`和`minorversion`是用于此连接的协议版本号。
- `pool_counter`统计客户端使用此连接池（进程）的次数。
- `pool_backendpid`是PostgreSQL进程的PID。
- `pool_connected` is true (1) if a frontend is currently using this backend.
- `status`是此进程的当前状态。可能的值有：
  - `Execute command`: 执行命令。
  - `Idle`: 进程正在等待新的客户端命令。
  - `Idle in transaction`: 进程正在等待事务中的新客户端命令。
  - `Wait for connection`: 进程正在等待新的客户端连接。
- `load_balance_node`如果前端当前正在使用此后端并且后端是负载平衡节点，则为true（1）。

它总是返回[num_init_children](https://www.pgpool.net/docs/latest/en/html/runtime-config-connection.html#GUC-NUM-INIT-CHILDREN) * [max_pool](https://www.pgpool.net/docs/latest/en/html/runtime-config-connection-pooling.html#GUC-MAX-POOL) * number_of_backends行。以下是一个示例会话：

```sql
test=# show pool_pools;
 pool_pid |     start_time      | client_connection_count | pool_id | backend_id | database | username | backend_connection_time | client_connection_time | client_disconnection_time | client_idle_duration | majorversion | minorversion | pool_counter | pool_backendpid | pool_connected |       status        | load_balance_node 
----------+---------------------+-------------------------+---------+------------+----------+----------+-------------------------+------------------------+---------------------------+----------------------+--------------+--------------+--------------+-----------------+----------------+---------------------+-------------------
 641408   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641408   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641408   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641408   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641408   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641408   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641408   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641408   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641409   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641410   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641411   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641412   | 2023-07-10 13:20:51 | 0                       | 0       | 0          | test     | t-ishii  | 2023-07-10 13:20:56     | 2023-07-10 13:20:56    |                           | 0                    | 3            | 0            | 1            | 641448          | 1              | Idle                | 1
 641412   | 2023-07-10 13:20:51 | 0                       | 0       | 1          | test     | t-ishii  | 2023-07-10 13:20:56     | 2023-07-10 13:20:56    |                           | 0                    | 3            | 0            | 1            | 641449          | 1              | Idle                | 0
 641412   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Idle                | 0
 641412   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Idle                | 0
 641412   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Idle                | 0
 641412   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Idle                | 0
 641412   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Idle                | 0
 641412   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Idle                | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641413   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641414   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641415   | 2023-07-10 13:20:51 | 0                       | 0       | 0          | test     | t-ishii  | 2023-07-10 13:21:04     | 2023-07-10 13:21:04    |                           | 0                    | 3            | 0            | 1            | 641455          | 1              | Execute command     | 0
 641415   | 2023-07-10 13:20:51 | 0                       | 0       | 1          | test     | t-ishii  | 2023-07-10 13:21:04     | 2023-07-10 13:21:04    |                           | 0                    | 3            | 0            | 1            | 641456          | 1              | Execute command     | 1
 641415   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Execute command     | 0
 641415   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Execute command     | 0
 641415   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Execute command     | 0
 641415   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Execute command     | 0
 641415   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Execute command     | 0
 641415   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Execute command     | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641416   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641417   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641418   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641419   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641420   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641421   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641422   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 0       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 0       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 1       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 1       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 2       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 2       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 3       | 0          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
 641423   | 2023-07-10 13:20:51 | 0                       | 3       | 1          |          |          |                         |                        |                           | 0                    | 0            | 0            | 0            | 0               | 0              | Wait for connection | 0
(128 rows)
```

# SHOW POOL_VERSION

## 名字

SHOW POOL_VERSION -- 显示一个包含Pgpool-II版本号的字符串。

## 简介

```sql
SHOW POOL_VERSION
```

## 描述

`SHOW POOL_VERSION` displays a string containing the Pgpool-II release number. Here is an example session:

```sql
test=# show pool_version;
pool_version
--------------------------
3.6.0 (subaruboshi)
(1 row)
```

# SHOW POOL_CACHE

## 名字

SHOW POOL_CACHE -- 显示缓存存储统计信息

## 简介

```sql
SHOW POOL_CACHE
```

## 描述

`SHOW POOL_CACHE` displays [in memory query cache ](https://www.pgpool.net/docs/latest/en/html/runtime-in-memory-query-cache.html)statistics if in memory query cache is enabled. Here is an example session: See [Table 1](https://www.pgpool.net/docs/latest/en/html/sql-show-pool-cache.html#SHOW-POOL-CACHE-TABLE) for a description of each item.

```sql
test=# \x
\x
Expanded display is on.
test=# show pool_cache;
show pool_cache;
-[ RECORD 1 ]---------------+---------
num_cache_hits              | 891703
num_selects                 | 99995
cache_hit_ratio             | 0.90
num_hash_entries            | 131072
used_hash_entries           | 99992
num_cache_entries           | 99992
used_cache_entries_size     | 12482600
free_cache_entries_size     | 54626264
fragment_cache_entries_size | 0
```

> [!CAUTION]
> 注意：如果缓存存储是memcached，则除num_cache_hits、num_selects和cache_hit_ratio之外的所有列的值都显示为0。

表1 显示在show pool_cache中的项目

| 名字                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `num_cache_hits`              | 查询缓存的点击次数。                                         |
| `num_selects`                 | 未命中查询缓存的SELECT数。                                   |
| `cache_hit_ratio`             | 缓存命中率。计算公式为num_cache_hits/(num_cache_hits+num_selects)。 |
| `num_hash_entries`            | The number of entries in the hash table used to manage the cache. In order to manage large number of cache Pgpool-II uses the hash table. The number of hash entries is automatically adjusted to the nearest power of two greater than [memqcache_max_num_cache](https://www.pgpool.net/docs/latest/en/html/runtime-in-memory-query-cache.html#GUC-MEMQCACHE-MAX-NUM-CACHE). For example, 100,000, which is the default for [memqcache_max_num_cache](https://www.pgpool.net/docs/latest/en/html/runtime-in-memory-query-cache.html#GUC-MEMQCACHE-MAX-NUM-CACHE) is adjusted to 131,072 (2 to the 17th power). |
| `used_hash_entries`           | The number of used hash entries. If the value approaches `num_hash_entries`, it is recommended to increase `num_hash_entries`. Even if all the hash table entries are used, no error is raised. However, performance suffers because hash table entries and caches are reused to register new cache entries. |
| `num_cache_entries`           | The number of cache entries already used. In the current implementation the number should be identical to `used_hash_entries`. |
| `free_cache_entries_size`     | The size in bytes of the unused cache. As this value approaches 0, it removes the registered cache and registers a new cache, which does not cause an error, but reduces performance. Consider to increase [memqcache_total_size](https://www.pgpool.net/docs/latest/en/html/runtime-in-memory-query-cache.html#GUC-MEMQCACHE-TOTAL-SIZE). |
| `fragment_cache_entries_size` | The size in bytes of the fragmented cache. When a registered cache is evicted, the space becomes fragmented until the next time that block is reused. Pgpool-II writes cache in fixed-size blocks specified by [memqcache_cache_block_size](https://www.pgpool.net/docs/latest/en/html/runtime-in-memory-query-cache.html#GUC-MEMQCACHE-CACHE-BLOCK-SIZE). When a registered cache is evicted, the space becomes fragmented until the next time that block is reused. `fragment_cache_entries_size` displays the total size of such fragmented regions. |

# SHOW POOL_HEALTH_CHECK_STATS

## 名字

SHOW POOL_HEALTH_CHECK_STATS -- 显示健康检查统计数据

## 简介

```sql
SHOW POOL_HEALTH_CHECK_STATS
```

## 描述

`SHOW POOL_HEALTH_CHECK_STATS`显示健康检查（见[第5.9节](https://www.pgpool.net/docs/latest/en/html/runtime-config-health-check.html))统计数据主要通过健康检查过程收集。此命令帮助Pgpool-II管理员研究与健康检查相关的事件。例如，管理员可以通过查看“last_failed_health_check”列在日志文件中轻松定位故障转移事件。另一个例子是通过评估“average_retry_count”列来发现与后端的不稳定连接。如果特定节点的重试次数高于其他节点，则与后端的连接可能存在问题。

[表1](https://www.pgpool.net/docs/latest/en/html/sql-show-pool-health-check-stats.html#HEALTH-CHECK-STATS-DATA-TABLE)显示了每个列名及其描述。

**Table 1. Statistics data shown by pool_health_check_stats command**

| 列名                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| node_id                      | 后端节点id。                                                 |
| hostname                     | 后端主机名或UNIX域套接字路径。                               |
| port                         | 后端端口号。                                                 |
| status                       | 后端状态：`up`、`down`、`waiting`、`unused`、`quarantine`    |
| role                         | 节点的角色。流式复制模式下的主模式或备用模式。其他模式下的主模式或副本模式。 |
| last_status_change           | 上次后端状态更改的时间戳。                                   |
| total_count                  | 健康检查总数。                                               |
| success_count                | 成功的健康检查计数总数。                                     |
| fail_count                   | 未通过的健康检查计数总数。                                   |
| skip_count                   | 跳过的健康检查计数总数。如果节点已经关闭，健康检查将跳过该节点。 |
| retry_count                  | 重试的健康检查计数总数。                                     |
| average_retry_count          | 健康检查会话中重试的平均健康检查计数数。                     |
| max_retry_count              | 健康检查会话中重试的最大健康检查计数。                       |
| max_duration                 | 最长健康检查持续时间（毫秒）。如果健康检查会话重试，则健康检查持续时间是每次重试的健康检查的总和。 |
| min_duration                 | 最短健康检查持续时间（毫秒）。如果健康检查会话重试，则健康检查持续时间是每次重试的健康检查的总和。 |
| average_duration             | 平均健康检查持续时间（毫秒）。如果健康检查会话重试，则健康检查持续时间是每次重试的健康检查的总和。 |
| last_health_check            | 上次健康检查的时间戳。如果尚未执行健康检查，则为空字符串。   |
| last_successful_health_check | 上次成功健康检查的时间戳。如果健康检查尚未成功，则为空字符串。 |
| last_skip_health_check       | 上次跳过的健康检查的时间戳。如果健康检查尚未跳过，则为空字符串。请注意，即使状态为关闭，此字段也可能是空字符串。在这种情况下，故障转移是由健康检查过程以外的其他过程触发的。 |
| last_failed_health_check     | 上次健康检查失败的时间戳。如果健康检查尚未失败，则为空字符串。请注意，即使状态为关闭，此字段也可能是空字符串。在这种情况下，故障转移是由健康检查过程以外的其他过程触发的。 |

以下是一个示例会话：

```sql
test=# show pool_health_check_stats;
-[ RECORD 1 ]----------------+--------------------
node_id                      | 0
hostname                     | /tmp
port                         | 11002
status                       | up
role                         | primary
last_status_change           | 2020-01-26 19:08:45
total_count                  | 27
success_count                | 27
fail_count                   | 0
skip_count                   | 0
retry_count                  | 0
average_retry_count          | 0.000000
max_retry_count              | 0
max_duration                 | 9
min_duration                 | 2
average_duration             | 6.296296
last_health_check            | 2020-01-26 19:12:45
last_successful_health_check | 2020-01-26 19:12:45
last_skip_health_check       | 
last_failed_health_check     | 
-[ RECORD 2 ]----------------+--------------------
node_id                      | 1
hostname                     | /tmp
port                         | 11003
status                       | down
role                         | standby
last_status_change           | 2020-01-26 19:11:48
total_count                  | 19
success_count                | 12
fail_count                   | 1
skip_count                   | 6
retry_count                  | 3
average_retry_count          | 0.230769
max_retry_count              | 3
max_duration                 | 83003
min_duration                 | 0
average_duration             | 6390.307692
last_health_check            | 2020-01-26 19:12:48
last_successful_health_check | 2020-01-26 19:10:15
last_skip_health_check       | 2020-01-26 19:12:48
last_failed_health_check     | 2020-01-26 19:11:48
```

# SHOW POOL_BACKEND_STATS

## 名字

SHOW POOL_BACKEND_STATS -- 显示后端SQL命令统计信息

## 简介

```sql
SHOW POOL_BACKEND_STATS
```

## 描述

`SHOW POOL_BACKEND_STATS`显示节点id、主机名、端口、状态、角色、向每个后端发出的SELECT/INSERT/UPDATE/DELETE/DDL/其他查询计数。此外，后端返回的错误消息也会按严重程度分类并显示。节点id、主机名、端口、状态和角色与[SHOW POOL NODES](https://www.pgpool.net/docs/latest/en/html/sql-show-pool-nodes.html)相同.

select_cnt、insert_cnt、update_cnt、delete_cnt、ddl_cnt、other_cnt是自Pgpool-II启动以来发出的SQL命令的编号：select、insert、update、delete、ddl等。失败的命令（例如从不存在的表中SELECT）将被计数。回滚的命令也被计算在内。目前，除了SELECT/WITH/INSERT/UPDATE/DELETE/CHECKPOINT/DEALLOCATE/DISCARD/EXECUTE/EXPLAIN/LISTEN/LOAD/LOCK/NOTIFY/PREPARE/SET/SHOW/Transaction commands/UNLISTEN之外，其他命令都被视为DDL。

以下是一个示例会话：

```sql
test=# show pool_backend_stats;
 node_id | hostname | port  | status |  role   | select_cnt | insert_cnt | update_cnt | delete_cnt | ddl_cnt | other_cnt | panic_cnt | fatal_cnt | error_cnt 
---------+----------+-------+--------+---------+------------+------------+------------+------------+---------+-----------+-----------+-----------+-----------
 0       | /tmp     | 11002 | up     | primary | 12         | 10         | 30         | 0          | 2       | 30        | 0         | 0         | 1
 1       | /tmp     | 11003 | up     | standby | 12         | 0          | 0          | 0          | 0       | 23        | 0         | 0         | 1
(2 rows)
```
