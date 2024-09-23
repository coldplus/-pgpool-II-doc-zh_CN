# 第4章 看门狗

## 4.1. 介绍

看门狗是Pgpool-II的一个子进程，用于增加高可用性。看门狗用于通过协调多个Pgpool-II节点来解决单点故障。看门狗最初是在pgpool II V3.2中引入的，在pgpool II V3.5中得到了显著增强，以确保始终存在法定人数。看门狗的这一新功能使其在处理和防范分裂脑综合征和网络分区方面更具容错性和鲁棒性。此外，V3.7引入了仲裁故障转移（见第5.15.6节），以减少PostgreSQL服务器故障的误报。为确保仲裁机制正常工作，Pgpool-II节点的数量必须为奇数且大于或等于3。

### 4.1.1. 协调多个Pgpool-II节点

看门狗通过相互交换信息来协调多个Pgpool-II节点。

在启动时，如果启用了看门狗，Pgpool-II节点会从领导者看门狗节点同步所有配置的后端节点的状态。如果节点本身继续成为领导者节点，它将在本地初始化后端状态。当后端节点状态因故障转移等而发生变化时，看门狗会将信息通知给其他Pgpool-II节点并同步它们。当发生在线恢复时，看门狗会限制客户端与其他Pgpool-II节点的连接，以避免后端之间的不一致。

看门狗还与所有连接的Pgpool-II节点协调，以确保故障回复、故障转移和follow_primary命令只能在一个Pgpool-II节点上执行。

### 4.1.2. 其他Pgpool-II节点的寿命检查

看门狗生命检查是看门狗的子组件，用于监控参与看门狗集群的Pgpool-II节点的健康状况，以提供高可用性。传统上，Pgpool-II看门狗提供两种远程节点健康检查方法。“heartbeat”和“query”模式。Pgpool-II V3.5中的看门狗在wd_lifecheck_method中添加了一个新的“external”，该方法能够将外部第三方健康检查系统与Pgpool-II看门狗挂钩。

除了远程节点健康检查监视器之外，lifecheck还可以通过监视与上游服务器的连接来检查其安装的节点的健康状况。如果监控失败，看门狗会将其视为本地Pgpool-II节点故障。

在heartbeat模式下，看门狗通过heartbeat信号监视其他Pgpool-II进程。看门狗定期接收其他Pgpool-II发送的心跳信号。如果在一段时间内没有信号，看门狗会认为这是Pgpool-II的故障。为了冗余，您可以使用多个网络连接在Pgpool-II节点之间进行心跳交换。这是用于健康检查的默认和推荐模式。

在query模式下，监视器监视Pgpool-II服务而不是进程。在此模式下，看门狗向其他Pgpool-II发送查询并检查响应。

注意：请注意，此方法需要来自其他Pgpool-II的连接，因此如果num_init_children参数不够大，它将无法进行监控。此模式已弃用，并保留以实现向后兼容性。

Pgpool-II V3.5引入了external模式。此模式基本上禁用了Pgpool-II看门狗的内置生命检查，并期望外部系统将通知看门狗参与看门狗集群的本地和所有远程节点的健康状况。

### 4.1.3. 所有Pgpool-II节点上配置参数的一致性

启动时，看门狗验证本地节点的Pgpool-II配置是否与领导者看门狗节点上的配置一致，并警告用户任何差异。这消除了由于不同Pgpool-II节点上的不同配置而可能发生的不期望行为的可能性。

### 4.1.4. 检测到特定故障时更改活动/待机状态

当检测到Pgpool-II的故障时，监视器会通知其他监视器。如果这是活动的Pgpool-II，监视器会通过投票决定新的活动Pgpool-II并更改活动/待机状态。

### 4.1.5. 自动虚拟IP交换

当备用Pgpool-II服务器升级为活动状态时，新的活动服务器会打开虚拟IP接口。同时，之前的活动服务器会关闭虚拟IP接口。这使得活动Pgpool-II即使在服务器切换时也能使用相同的IP地址工作。

### 4.1.6. 自动将服务器注册为恢复中的备用服务器

当损坏的服务器恢复或连接新服务器时，监视进程会将此情况与新服务器的信息一起通知给集群中的其他监视进程，监视进程还会接收活动服务器和其他服务器的信息。然后，将连接的服务器注册为备用服务器。

### 4.1.7. 启动/停止监视器

看门狗进程作为Pgpool-II的子进程自动启动和停止，因此没有专门的命令来启动和停止看门狗。

监视器控制虚拟IP接口，监视器执行的启动和关闭VIP的命令需要root权限。当监视器与虚拟IP一起启用时，Pgpool-II要求运行Pgpool-II的用户具有root权限。然而，以root用户身份运行Pgpool-II并不是一种好的安全做法，另一种首选的方法是以普通用户身份运行Pgpool-II，并使用sudo对if_up_cmd、if_down_cmd和arping_cmd使用自定义命令，或者对if_*命令使用setuid（“执行时设置用户ID”）

生命检查进程是看门狗的一个子组件，其工作是监视参与看门狗集群的Pgpool-II节点的健康状况。当看门狗配置为使用内置生命检查时，生命检查进程会自动启动，它会在看门狗主进程初始化完成后启动。然而，只有当所有配置的监视节点加入集群并处于活动状态时，生命检查过程才会启动。如果某个远程节点在生命检查激活之前发生故障，该故障将不会被生命检查捕获。

## 4.2. 将外部生命检查与监视器集成

Pgpool-II监视器进程使用BSD套接字与所有Pgpool-II进程通信，任何第三方系统也可以使用同一BSD套接字为本地和远程Pgpool-II监视节点提供生命检查功能。IPC的BSD套接字文件名是通过在“s.PGPOOLWD_CMD.”字符串后附加Pgpool-II wd_port来构造的，套接字文件放置在wd_IPC_socket_dir目录中。

### 4.2.1. 看门狗IPC命令包格式

监视器IPC命令包由三个字段组成。下表详细介绍了消息字段和描述。

表4-1 看门狗IPC命令包格式

| Field | 类型                        | 说明                  |
| --------- | --------------------------- | ---------------------------- |
| TYPE      | BYTE1                       | Command Type                 |
| LENGTH    | INT32 in network byte order | The length of data to follow |
| DATA      | DATA in JSON format         | Command data in JSON format  |

### 4.2.2. 看门狗IPC结果包格式

看门狗IPC命令结果包由三个字段组成。下表详细介绍了消息字段和描述。

表4-2 看门狗IPC结果包格式

| Field  | Type                        | Description                        |
| ------ | --------------------------- | ---------------------------------- |
| TYPE   | BYTE1                       | Command Type                       |
| LENGTH | INT32 in network byte order | The length of data to follow       |
| DATA   | DATA in JSON format         | Command result data in JSON format |

### 4.2.3. 看门狗IPC命令包类型

发送给看门狗进程的IPC命令包的第一个字节和看门狗进程返回的结果被标识为命令或命令结果类型。下表列出了所有有效类型及其含义

表4-3 看门狗IPC命令包类型

| Name                       | Byte Value | Type           | Description                                                  |
| -------------------------- | ---------- | -------------- | ------------------------------------------------------------ |
| REGISTER FOR NOTIFICATIONS | '0'        | Command packet | Command to register the current connection to receive watchdog notifications |
| NODE STATUS CHANGE         | '2'        | Command packet | Command to inform watchdog about node status change of watchdog node |
| GET NODES LIST             | '3'        | Command packet | Command to get the list of all configured watchdog nodes     |
| NODES LIST DATA            | '4'        | Result packet  | The JSON data in packet contains the list of all configured watchdog nodes |
| CLUSTER IN TRANSITION      | '7'        | Result packet  | Watchdog returns this packet type when it is not possible to process the command because the cluster is transitioning. |
| RESULT BAD                 | '8'        | Result packet  | Watchdog returns this packet type when the IPC command fails |
| RESULT OK                  | '9'        | Result packet  | Watchdog returns this packet type when IPC command succeeds  |

### 4.2.4. 外部生命检查IPC数据包和数据

看门狗的“GET NODES LIST”、“NODES LIST DATA”和“NODE STATUS CHANGE”IPC消息可用于集成外部生命检查系统。请注意，pgpool的内置生命检查也使用相同的通道和技术。

#### 4.2.4.1. 获取已配置的监视节点列表

如果设置了wd_authkey，任何第三方生命检查系统都可以在看门狗IPC套接字上发送“GET NODES LIST”数据包，其中包含包含授权密钥和值的JSON数据，或者在wd_authkey未配置为获取“NODES LIST data”结果数据包时发送空数据包数据。

看门狗为“GET NODES LIST”返回的结果包将包含所有配置的看门狗节点的列表，以JSON格式进行健康检查。监视器节点的JSON包含所有监视器节点的“监视器节点”数组。每个监视器JSON节点都包含每个节点的“ID”、“NodeName”、“HostName”、”DelegateIP“、”WdPort“和”PgpoolPort“。

```shell
-- The example JSON data contained in "NODES LIST DATA"

{
"NodeCount":3,
"WatchdogNodes":
[
{
"ID":0,
"State":1,
"NodeName":"Linux_ubuntu_9999",
"HostName":"watchdog-host1",
"DelegateIP":"172.16.5.133",
"WdPort":9000,
"PgpoolPort":9999
},
{
"ID":1,
"State":1,
"NodeName":"Linux_ubuntu_9991",
"HostName":"watchdog-host2",
"DelegateIP":"172.16.5.133",
"WdPort":9000,
"PgpoolPort":9991
},
{
"ID":2,
"State":1,
"NodeName":"Linux_ubuntu_9992",
"HostName":"watchdog-host3",
"DelegateIP":"172.16.5.133",
"WdPort":9000,
"PgpoolPort":9992
}
]
}

-- Note that ID 0 is always reserved for local watchdog node
```

从看门狗获取配置的看门狗节点信息后，外部生命检查系统可以继续对看门狗节点进行健康检查，当它检测到任何节点的状态发生变化时，它可以使用看门狗的“节点状态变化”IPC消息通知看门狗。消息中的数据应包含JSON，其中包含状态更改的节点的节点ID（节点ID必须与监视器在监视器节点列表中为该节点返回的节点ID相同）和节点的新状态。

```shell
-- The example JSON to inform pgpool-II watchdog about health check
failed on node with ID 1 will look like

{
"NodeID":1,
"NodeStatus":1,
"Message":"optional message string to log by watchdog for this event"
"IPCAuthKey":"wd_authkey configuration parameter value"
}

-- NodeStatus values meanings are as follows
NODE STATUS DEAD  =  1
NODE STATUS ALIVE =  2
```

## 4.3. 对看门狗的限制

### 4.3.1. 具有查询模式生命检查的监视器限制

在查询模式下，当由于PostgreSQL服务器故障或发出pcp_deach_node，所有DB节点都从Pgpool-II中分离出来时，看门狗会认为Pgpool-II服务处于关闭状态，并关闭分配给看门狗的虚拟IP。因此，Pgpool-II的客户端无法再使用虚拟IP连接到Pgpool-II。这对于避免大脑分裂是必要的，即存在多个活跃的Pgpool-II的情况。

### 4.3.2. 连接到监视器状态为关闭的Pgpool-II

不要使用真实IP连接到处于关闭状态的Pgpool-II。因为处于关闭状态的Pgpool-II无法从其他Pgpool-II监视器接收信息，所以它的后端状态可能与其他PgpoolII不同。

### 4.3.3. 监视器状态为关闭的Pgpool-II需要重新启动

处于关闭状态的Pgpool-II不能变为活动状态，也不能变为备用Pgpool-II。从停机状态恢复需要重新启动Pgpool-II。

### 4.3.4. 看门狗晋升为活跃状态只需几秒钟

在活动Pgpool-II停止后，需要几秒钟的时间，直到备用Pgpool-II升级到新的活动状态，以确保在向其他Pgpool-II发送关闭通知包之前，关闭之前的虚拟IP。

## 4.4. 看门狗的架构

看门狗是Pgpool-II的一个子进程，它通过协调多个Pgpool-II来增加高可用性并解决单点故障。当Pgpool-II启动时，看门狗进程会自动启动（如果启用），它由两个主要组件组成，看门狗核心和生命检查系统。

### 4.4.1. 看门狗核心

被称为“看门狗”的看门狗核心是一个Pgpool-II子进程，它管理与集群中存在的Pgpool-II节点的所有看门狗相关通信，也与Pgpool-II父进程和生命检查进程通信。

看门狗进程的核心是一个状态机，它从初始状态（WD_LOADING）开始，向待机（WD_standby）或领导者/协调器（WD_coordinator）状态过渡。备用和领导者/协调器状态都是看门狗状态机的稳定状态，节点保持在备用或领导者/协调者状态，直到检测到本地Pgpool-II节点中的某些问题或远程Pgpool-II与集群断开连接。

看门狗进程执行以下任务：

- 管理和协调本地节点监视器状态。

- 与内置或外部生命检查系统交互，用于本地和远程Pgpool-II节点健康检查。

- 与Pgpool-II主进程交互，并为Pgpool-II父进程提供通过看门狗通道执行集群命令的机制。

- 与所有参与的Pgpool-II节点进行通信，以协调领导者/协调员节点的选择，并确保集群中的法定人数。

- 管理活动/协调器节点上的虚拟IP，并允许用户提供用于升级和降级的自定义脚本。

- 验证监视器集群中参与的Pgpool-II节点之间Pgpool-II配置的一致性。

- 启动时同步所有PostgreSQL后端的状态。

- 为Pgpool-II主进程提供分布式锁定功能，以同步不同的故障转移命令。

#### 4.4.1.1. 与群集中其他节点的通信

看门狗使用TCP/IP套接字与其他节点进行所有通信。每个看门狗节点可以有两个与每个节点打开的套接字。一个是该节点创建并发起与远程节点连接的输出（客户端）套接字，第二个套接字是监听远程监视节点发起的入站连接的套接字。一旦与远程节点的套接字连接成功，看门狗就会在该套接字上发送ADD node（WD_ADD_node_MESSAGE）消息。在收到ADD NODE消息后，看门狗节点使用该节点的Pgpool-II配置验证消息中封装的节点信息，如果该节点通过验证测试，则将其添加到集群中，否则连接将断开。

#### 4.4.1.2. IPC和数据格式

监视器进程为IPC通信公开一个UNIX域套接字，该套接字接受并提供JSON格式的数据。所有内部Pgpool-II进程，包括Pgpool-II的内置生命检查和Pgpool-II主进程，都使用此IPC套接字接口与监视器进行交互。任何外部/第三方系统都可以使用此IPC套接字与看门狗进行交互。

有关如何使用看门狗IPC接口集成外部/第三方系统的详细信息，请参阅第4.2节。

### 4.4.2. 看门狗生命检查

看门狗生命检查是看门狗的子组件，用于监视参与看门狗集群的Pgpool-II节点的健康状况。Pgpool-II看门狗提供了三种内置的远程节点健康检查方法，即“心跳”、“查询”和“外部”模式。

在“心跳”模式下，生命检查进程通过UDP套接字发送和接收数据，以检查远程节点的可用性，对于每个节点，父生命检查进程生成两个子进程，一个用于发送心跳信号，另一个用于接收心跳。在“查询”模式下，生命检查过程使用PostgreSQL libpq接口查询远程Pgpool-II。在这种模式下，生命检查进程为每个健康检查查询创建一个新线程，一旦查询完成，该线程就会被销毁。在“外部”模式下，此模式禁用Pgpool-II的内置生命检查，并期望外部系统将监视本地和远程节点。

除了远程节点健康检查监视器之外，lifecheck还可以通过监视与上游服务器的连接来检查其安装的节点的健康状况。为了监控与上游服务器Pgpool-II的连接，lifecheck使用execv（）函数执行“ping-q-c3 hostname”命令。因此，会生成一个新的子进程来执行每个ping命令。这意味着，对于每个健康检查周期，每个配置的上游服务器都会创建和销毁一个子进程。例如，如果在生命检查中配置了两个上游服务器，并要求其每隔十秒进行一次健康检查，那么在每十秒的生命检查后，将生成两个子进程，每个上游服务器一个，每个进程将一直运行到ping命令完成。