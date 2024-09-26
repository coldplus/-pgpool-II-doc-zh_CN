# 限制

本节介绍Pgpool-II的当前限制。

- 使用pg_terminate_backend

	如果您使用pg_terminate_backend（）停止后端，这将触发故障转移。发生这种情况的原因是PostgreSQL为终止的后端发送的消息与完全关闭邮局主管的消息完全相同。在3.6版本之前没有解决方法。从3.6版本开始，这一限制得到了缓解。如果函数的参数（即进程id）是常量，则可以安全地使用该函数。在扩展协议模式下，您无法使用该功能。在4.3或更高版本中，您可以通过设置failover_on_backend_shutdown来完全防止由pg_terminate_backend（）引起的故障转移，但这将防止因邮局主管终止而导致的故障转移。

- 负载平衡

	多语句查询（单行上的多个SQL命令）总是发送到主节点（在流复制模式下）或主节点（其他模式下）。通常Pgpool-II会将查询分派到适当的节点，但它不适用于多语句查询。

- 身份验证/访问控制

	支持信任和pam方法。从Pgpool-II 3.0开始也支持md5。md5通过使用身份验证文件pool_passwd来支持。自PgpoolII4.0以来，还支持scram-sha-256、证书和明文密码。pool_passwd是身份验证文件的默认名称。以下是启用md5身份验证的步骤：

	1. 以数据库的操作系统用户身份登录，然后键入：

	```shell
	pg_md5 --md5auth --username=your_username your_passwd
	```

	用户名和md5加密密码注册到pool_passwd中。如果pool_passwd还不存在，pg_md5命令将自动为您创建它。pool_passwd的格式为用户名：encrypted_passwd。

	2. 您还需要在pool_hba.conf中添加一个适当的md5条目。有关更多详细信息，请参阅第6.1节。

	3. 请注意，用户名和密码必须与PostgreSQL中注册的用户名和密码相同。

	4. 更改md5密码后（当然是在pool_passwd和PostgreSQL中），您需要执行pgpool-reload。

	有关设置scram-sha-256身份验证的详细信息，请参阅第6.2.4.2节。

- 大型对象

	在流式复制模式下，Pgpool-II支持大型对象。

	在快照隔离模式和本机复制模式下，如果后端是PostgreSQL 8.1或更高版本，Pgpool-II支持大型对象。为此，您需要在pgpool.conf中启用lobj_lock_table指令。但是，不支持使用后端函数lo_import进行大型对象复制。

	在其他模式下，包括Slony模式，不支持大型对象。

- 临时表

	在本机复制模式下，创建/插入/更新/删除临时表始终在主表上执行。这些表上的SELECT也在主表上执行。但是，如果临时表名在SELECT中用作文字，则无法检测到它，SELECT将进行负载平衡。这将触发“找不到表”错误，或者将找到另一个同名表。为了避免这个问题，请使用SQL注释。

	请注意，在访问系统目录的查询中使用的这种文字表名确实会导致上述问题。psql的\d命令生成这样的查询：

	```shell
	SELECT 't1'::regclass::oid;
	```
	
	在这种情况下，Pgpool-II总是将查询发送到主服务器，不会造成问题。
	
	如果您使用的是PostgreSQL 8.3或更高版本，则通过在reset_query_list中指定DISCARD ALL，将在会话结束时删除由CREATE TEMP TABLE创建的表。
	
	对于8.2.x或更早版本，退出会话后不会删除由CREATE TEMP TABLE创建的表。这是因为连接池，从PostgreSQL后端的角度来看，它使会话保持活动状态。为了避免这种情况，您必须通过发出drop TABLE来显式删除临时表，或者使用CREATE TEMP TABLE。。。事务块内的ON COMMIT DROP。
	
- 快照隔离模式和本机复制模式下的功能等

	无法保证使用上下文相关机制（例如随机数、事务ID、OID、SERIAL、序列等）提供的任何数据都能在多个后端上正确复制。对于SERIAL，启用insert_lock将有助于复制数据。insert_lock还有助于SELECT setval（）和SELECT nextval（）。
	
	使用CURRENT_TIMESTAMP、CURRENT_DATE、now（）进行插入/更新将被正确复制。使用CURRENT_TIMESTAMP、CURRENT_DATE、now（）作为默认值的表的INSERT/UPDATE也将被正确复制。这是通过在查询执行时用从PostgreSQL获取的常量替换这些函数来实现的。然而，存在一些局限性：
	
	在Pgpool-II 3.0或更早版本中，表默认值中的时间数据计算在某些情况下并不准确。例如，下表定义：

	```sql
	CREATE TABLE rel1(
	d1 date DEFAULT CURRENT_DATE + 1
	)
	```

	被视为：

	```sql
	CREATE TABLE rel1(
	d1 date DEFAULT CURRENT_DATE
	)
	```

	Pgpool-II 3.1或更高版本可以正确处理这些情况。因此，列“d1”将明天作为默认值。但是，如果使用扩展协议（例如在JDBC、PHP PDO中使用）或PREPARE，则此增强不适用。

	请注意，如果列类型不是时态类型，则不会执行重写。例如：

	```shell
	foo bigint default (date_part('epoch'::text,('now'::text)::timestamp(3) with time zone) * (1000)::double precision)
	```

	假设我们有下表：

	```sql
	CREATE TABLE rel1(
	c1 int,
	c2 timestamp default now()
	)
	```

	我们可以复制

	```sql
	INSERT INTO rel1(c1) VALUES(1)
	```

	因为这变成了

	```sql
	INSERT INTO rel1(c1, c2) VALUES(1, '2009-01-01 23:59:59.123456+09')
	```

	然而，

	```sql
	INSERT INTO rel1(c1) SELECT 1
	```

	 无法转换，因此无法在当前实现中正确复制。值仍将被插入，根本不进行转换。

- SQL类型命令

	SQL类型的命令不能在扩展查询模式下使用。

- 多字节字符

	Pgpool-II不执行客户端和PostgreSQL之间的多字节字符编码转换。客户端和后端的编码必须相同。

- libpq

	在构建Pgpool-II时，libpq是链接的。libpq版本必须是3.0或更高版本。使用libpq 2.0版本构建Pgpool-II将失败。

- 参数状态

	当客户端连接到PostgreSQL时，PostgreSQL会向客户端发送一些参数/值对。此协议称为ParameterStatus。可以使用一些API（如libpq的PQParameterStatus）提取参数/值对。实际参数名称可以在此处找到。Pgpool-II从多个PostgreSQL服务器收集ParameterStatus值，这些值可能因服务器而异。一个典型的例子是in_hot_standby，它在PostgreSQL 14中引入。该变量的值在主服务器上处于关闭状态，在备用服务器上处于打开状态。问题是，Pgpool-II只需向客户端返回其中一个。在这种情况下，它选择主服务器报告的值。因此，PQParameterStatus将返回off。另一方面，当客户端问题显示in_hot_standby时，返回的值可以是On或off，具体取决于会话的负载平衡节点。

	请注意，如果服务器之间的值不同，Pgpool-II将发出除in_hot_standby之外的日志消息。这是为了防止日志文件被淹没，因为in_hot_standby总是不同的。

- set_config

	PostgreSQL具有set_config函数，允许在当前会话中更改参数值，如set命令（实际上set_config比set具有更多功能。有关更多详细信息，请参阅PostgreSQL手册）。当Pgpool-II在群集模式设置为streaming_replication的情况下运行时，它只将函数发送到主服务器。由于该函数未发送到备用服务器，因此每个服务器的参数值都不同。为了避免这个问题，您可以使用SET命令而不是SET_config。由于SET命令被发送到用于此会话的所有服务器，因此不会发生此问题。但是，如果您使用2个以上的PostgreSQL服务器，则需要禁用statement_level_load_balance并使用SET命令。这是因为，如果启用了statement_level_load_balance，除了主服务器和分配给负载平衡节点的服务器外，查询还可能被发送到第三台服务器。

	如果需要使用set_config，请关闭会话的负载平衡（不仅是set_config的负载平衡，还应在整个会话中禁用负载平衡）。你可以通过牺牲性能来避免这个问题。
