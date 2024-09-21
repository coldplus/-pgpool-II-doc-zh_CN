# 第6章 客户端身份验证

[TOC]

由于Pgpool-II是一个在PostgreSQL服务器和PostgreSQL数据库客户端之间工作的中间件，因此当客户端应用程序连接到Pgpool-II时，Pgpool-II会使用相同的凭据连接到PostgreSQL服务器，为传入的客户端连接提供服务。因此，PostgreSQL中为用户定义的所有访问权限和限制都会自动应用于所有Pgpool-II客户端，但PostgreSQL端的身份验证除外，这取决于客户端的IP地址或主机名。原因是与PostgreSQL服务器的连接是由Pgpool-II代表连接的客户端进行的，PostgreSQL服务器只能看到Pgpool-II服务器的IP地址，而不能看到实际客户端的IP地址。因此，对于基于客户端主机的身份验证，Pgpool-II具有类似于pg_hba机制的pool_hba机制，用于对传入的客户端连接进行身份验证。

## 6.1. pool_hba.conf文件

就像PostgreSQL的pg_hba.conf文件一样，Pgpool-II使用名为pool_hba.conf的配置文件支持类似的客户端身份验证功能。如果Pgpool-II是从源代码安装的，它还包括默认配置目录（“/usr/local/etc”）中的示例pool_hba.conf.sample文件。默认情况下，pool_hba身份验证被禁用，将enable_pool_hba设置为on将启用它。请参阅enable_pool.hba配置参数。

> [!CAUTION]
> 注意：如果PostgreSQL服务器的数量只有一个，或者在原始模式下运行时（见第3.3.2节），pool_hba.conf是不必要的，因此enable_pool_hba可能被设置为关闭。在这种情况下，客户端身份验证方法完全由PostgreSQL管理。此外，如果PostgreSQL服务器的数量超过一个，或者没有在原始模式下运行，则可以将enable_pool_hba设置为关闭，只要PostgreSQL定义的身份验证方法是信任的。

pool_hba.conf文件的格式非常接近PostgreSQL的pg_hba.conf格式。

pool_hba.conf文件的一般格式是一组记录，每行一条。空行将被忽略，#注释字符后的任何文本也将被忽略。记录不能跨行继续。记录由多个字段组成，这些字段由空格和/或制表符分隔。如果字段值为双引号，则字段可以包含空格。引用数据库、用户或地址字段中的一个关键字（例如，全部或复制）会使该词失去其特殊含义，只需将数据库、用户或者主机与该名称匹配即可。

每条记录都指定了连接类型、客户端IP地址范围（如果与连接类型相关）、数据库名称、用户名以及用于匹配这些参数的连接的身份验证方法。具有匹配连接类型、客户端地址、请求的数据库和用户名的第一条记录用于执行身份验证。没有“失败”或“备份”：如果选择了一条记录并且身份验证失败，则不考虑后续记录。如果没有匹配的记录，则拒绝访问。

记录可以具有以下格式之一

```shell
local      database  user  auth-method  [auth-options]

host       database  user  IP-address IP-mask  auth-method  [auth-options]
hostssl    database  user  IP-address IP-mask  auth-method  [auth-options]
hostnossl  database  user  IP-address IP-mask  auth-method  [auth-options]

host       database  user  address  auth-method  [auth-options]
hostssl    database  user  address  auth-method  [auth-options]
hostnossl  database  user  address  auth-method  [auth-options]
```

字段的含义如下：

- local

	此记录与使用Unix域套接字的连接尝试相匹配。如果没有这种类型的记录，则不允许Unix域套接字连接。

- host

	此记录与使用TCP/IP进行的连接尝试相匹配。主机记录与SSL或非SSL连接尝试匹配。

	> [!CAUTION]
	> 注意：除非服务器以listen_addresses配置参数的适当值启动，否则无法进行远程TCP/IP连接，因为默认行为是仅在本地环回地址localhost上侦听TCP/IP连接。

- hostssl

	此记录与使用TCP/IP进行的连接尝试相匹配，但仅限于使用SSL加密进行连接时。

	要使用此选项，Pgpool-II必须构建SSL支持。此外，必须通过设置SSL配置参数来启用SSL。否则，将忽略hostssl记录。

- hostnossl

	指定此记录匹配的数据库名称。值all指定它与所有数据库匹配。

	> [!CAUTION]
	> 注意：数据库字段不支持“samegroup”：
	> 由于Pgpool-II对PostgreSQL后端服务器中的用户一无所知，因此只需将数据库名称与pool_hba.conf的数据库字段中的条目进行比较。

- user

	指定此记录匹配的数据库用户名。值all指定它匹配所有用户。否则，这是特定数据库用户的名称

	> [!CAUTION]
	> 注意：不支持用户字段“+”后面的组名：
	>这与数据库字段的“samegroup”的原因相同。用户名只需与pool_hba.conf的user字段中的条目进行核对。

- address

	指定此记录匹配的客户端计算机地址。此字段可以包含主机名、IP地址范围或下面提到的特殊关键字之一。

	IP地址范围的起始地址使用标准数字表示法指定，然后是斜线（/）和CIDR掩码长度。掩码长度表示必须匹配的客户端IP地址的高位位数。在给定的IP地址中，其右侧的位应为零。IP地址、/和CIDR掩码长度之间不得有任何空格。

	以这种方式指定的IPv4地址范围的典型示例是单个主机的172.20.143.89/32，小型网络的172.20.123.0/24，大型网络的10.6.0.0/16。对于单个主机，IPv6地址范围可能看起来像：1/128（在本例中为IPv6环回地址），对于小型网络，可能看起来像fe80:：7a31:c1ff:0000:0000/96。0.0.0.0/0表示所有IPv4地址，：0/0表示所有IPv6地址。要指定单个主机，请对IPv4使用32的掩码长度，对IPv6使用128的掩码长度。在网络地址中，不要省略尾随零。

	IPv4格式的条目将仅匹配IPv4连接，IPv6格式的条目也将仅匹配IPv6连接，即使所表示的地址在IPv4-in-IPv6范围内。请注意，如果系统的C库不支持IPv6地址，则IPv6格式的条目将被拒绝。

	您还可以写入全部以匹配任何IP地址，写入samehost以匹配任何服务器自己的IP地址，或写入samenet以匹配服务器直接连接到的任何子网中的任何地址。

	如果指定了主机名（任何不是IP地址范围或特殊关键字的名称都被视为主机名），则将该名称与客户端IP地址的反向名称解析结果进行比较（例如，如果使用DNS，则进行反向DNS查找）。主机名比较不区分大小写。如果匹配，则对主机名执行正向名称解析（例如，正向DNS查找），以检查它解析的任何地址是否等于客户端的IP地址。如果两个方向都匹配，则认为条目匹配。（pool_hba.conf中使用的主机名应该是客户端IP地址的地址到名称解析返回的主机名，否则该行将不匹配。一些主机名数据库允许将IP地址与多个主机名相关联，但操作系统在被要求解析IP地址时只会返回一个主机名。）

	以点（.）开头的主机名规范与实际主机名的后缀匹配。因此，.example.com将匹配foo.example.com（但不仅仅是example.com）。

	当在pool_hba.conf中指定主机名时，您应该确保名称解析速度相当快。设置本地名称解析缓存（如nscd）可能是有益的。

	此字段仅适用于主机、hostssl和hostnossl记录。

	Pgpool II V3.7之前不支持在地址字段中指定主机名。

- IP-address
- IP-mask

	这两个字段可以用作IP地址/掩码长度表示法的替代方案。实际掩码在单独的列中指定，而不是指定掩码长度。例如，255.0.0.0表示IPv4 CIDR掩码长度为8，255.255.255.255表示CIDR掩码长度32。

	此字段仅适用于主机、hostssl和hostnossl记录。

- auth-method

	指定连接与此记录匹配时使用的身份验证方法。这里总结了可能的选择；详情见第6.2节。

	- trust

		无条件允许连接。这种方法允许任何可以连接到Pgpool-II的人。

	- reject

		无条件拒绝连接。这对于“过滤”某些主机很有用，例如，拒绝线可能会阻止特定主机连接。

	- md5

		要求客户端提供双MD5密码进行身份验证。
		
		> [!CAUTION]
		> 注意：要使用md5身份验证，您需要在pool_passwd文件中注册用户名和密码。详见第6.2.3节。如果你不想使用pool_passwd管理密码，你可以使用allow_clear_text_front_auth。

	- scram-sha-256

		执行SCRAM-SHA-256身份验证以验证用户的密码。

		> [!CAUTION]
		> 注意：要使用scram-sha-256身份验证，您需要在pool_passwd文件中注册用户名和密码。详见第6.2.4节。如果你不想使用pool_passwd管理密码，你可以使用allow_clear_text_front_auth。

	- cert

		使用SSL客户端证书进行身份验证。详见第6.2.5节。

	- pam

		使用操作系统提供的可插拔身份验证模块（PAM）服务进行身份验证。详见第6.2.6节。

		使用运行Pgpool-II的主机上的用户信息支持PAM身份验证。要启用PAM支持，Pgpool-II必须配置为“--with-PAM”

		要启用PAM身份验证，您必须在系统的PAM配置目录（通常位于“/etc/PAM.d”）中为Pgpool-II创建一个服务配置文件。示例服务配置文件也作为“share/pgpool.pam”安装在安装目录下。

	- ldap

		使用LDAP服务器进行身份验证。详见第6.2.7节。

		要启用LDAP支持，Pgpool-II必须配置为“--with-LDAP”

- auth-options

	在auth方法字段之后，可以有name=value格式的字段，用于指定身份验证方法的选项。

由于每次连接尝试都会按顺序检查pool_hba.conf记录，因此记录的顺序很重要。通常，早期记录将具有紧密的连接匹配参数和较弱的身份验证方法，而后期记录将具有较松散的匹配参数和较强的身份验证方式。例如，人们可能希望对本地TCP/IP连接使用信任身份验证，但对远程TCP/IP连接需要密码。在这种情况下，为127.0.0.1中的连接指定信任身份验证的记录将出现在为更广泛的允许客户端IP地址指定密码身份验证的纪录之前。

> [!NOTE]
> 提示：本节中描述的所有pool_hba身份验证选项都是关于客户端和Pgpool-II之间发生的身份验证。客户端仍然必须经过PostgreSQL的身份验证过程，并且必须拥有后端PostgreSQL服务器上数据库的CONNECT权限。
> 就pool_hba而言，客户端给出的用户名和/或数据库名（即psql-U testuser testdb）是否真的存在于后端并不重要。pool_hba只关心pool_hba.conf中是否可以找到匹配项。

pool_hba.conf条目的一些示例。有关不同身份验证方法的详细信息，请参阅下一节。

示例6-1 pool_hba.conf条目示例

```shell
# Allow any user on the local system to connect to any database with
# any database user name using Unix-domain sockets (the default for local
# connections).
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust

# The same using local loopback TCP/IP connections.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             127.0.0.1/32            trust

# Allow any user from host 192.168.12.10 to connect to database
# "postgres" if the user's password is correctly supplied.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    postgres        all             192.168.12.10/32        md5
```

## 6.2. 身份验证方法

以下小节更详细地描述了身份验证方法。

### 6.2.1. 信任身份验证

当指定了信任身份验证时，Pgpool-II假设任何可以连接到服务器的人都有权使用他们指定的任何数据库用户名访问连接。

### 6.2.2. 明文密码验证

密码方法以明文形式发送密码，因此容易受到密码“嗅探”攻击。如果可能的话，应该尽量避免。不过，如果连接受SSL加密保护，则可以安全地使用密码。为此，建议在pool_hba.conf中使用hostssl，以便强制客户端使用SSL加密。

使用该方法的一个好处是，身份验证的密码由客户端提供，不咨询pool_passwd。因此，您可以避免维护pool_passwd文件。

您也可以使用allow_clear_text_front_auth来避免维护pool_passwd，但它不会强制使用SSL加密，因为pool_hba.conf不能与该参数一起使用。

### 6.2.3. MD5密码验证

这种身份验证方法是基于密码的身份验证方法，其中MD-5隐藏的密码由客户端发送。由于Pgpool-II不具有PostgreSQL数据库用户密码的可见性，客户端应用程序只发送密码的MD5哈希，因此Pgpool-II中的MD5身份验证支持使用pool_passwd身份验证文件。

> [!CAUTION]
> 注意：如果Pgpool-II在原始模式下运行，或者只配置了一个后端，则不需要设置pool_passwd。

### 6.2.3.1. 身份验证文件格式

要使用md5身份验证pool_passwd身份验证文件，必须包含纯文本md5或AES加密格式的用户密码。

pool_passwd文件应包含以下格式的行：

```shell
"username:plain_text_passwd"
```

```shell
"username:encrypted_passwd"
```

#### 6.2.3.2. 设置md5身份验证

以下是启用md5身份验证的步骤：

1. 以数据库的操作系统用户身份登录，键入“pg_md5--config-file=path_to_pgpool.conf--md5auth--username=username-password”，用户名和md5加密密码将注册到pool_passwd中。如果pool_passwd还不存在，pg_md5命令将自动为您创建它。

   > [!CAUTION]
   > 注意：用户名和密码必须与PostgreSQL服务器中注册的用户名和密码相同。

2. 在pool_hba.conf中添加适当的md5条目。有关更多详细信息，请参阅第6.1节。

3. 更改md5密码（当然在pool_passwd和PostgreSQL中）后，重新加载pgpool配置。

### 6.2.4. scram-sha-256身份验证

这种身份验证方法也称为SCRAM，是一种基于挑战-响应的身份验证，可以防止在不受信任的连接上嗅探密码。由于Pgpool-II没有PostgreSQL数据库用户密码的可见性，因此使用pool_passwd身份验证文件支持SCRAM身份验证。

#### 6.2.4.1. SCRAM的身份验证文件条目

要使用SCRAM身份验证pool_passwd身份验证文件，必须以纯文本或AES加密格式包含用户密码。

```shell
"username:plain_text_passwd"
```

```shell
"username:AES_encrypted_passwd"
```

注意：pool_passwd文件中的md5类型用户密码不能用于scram身份验证

#### 6.2.4.2. 设置scram-sha-256身份验证

以下是启用scram-sha-256身份验证的步骤：

1. 以纯文本或AES加密格式为数据库用户和密码创建pool_passwd文件条目。Pgpool-II附带的pg_enc实用程序可用于在pool_passwd文件中创建AES加密的密码条目。

	> [!CAUTION]
	> 注意：用户名和密码必须与PostgreSQL服务器中注册的用户名和密码相同。

2. 在pool_hba.conf中添加适当的scram-sha-256条目。有关更多详细信息，请参阅第6.1节。

3. 更改SCRAM密码（当然在pool_passwd和PostgreSQL中）后，重新加载Pgpool-II配置。

### 6.2.5. 证书身份验证

此身份验证方法使用SSL客户端证书执行身份验证。因此，它仅适用于SSL连接。使用此身份验证方法时，Pgpool-II将要求客户端提供有效的证书。不会向客户端发送密码提示。证书的cn（Common Name）属性将与请求的数据库用户名进行比较，如果它们匹配，则允许登录。

> [!CAUTION]
> 注意：证书身份验证仅在客户端和Pgpool-II之间有效。证书身份验证在Pgpool-II和PostgreSQL之间不起作用。对于后端身份验证，您可以使用任何其他身份验证方法。

### 6.2.6. PAM身份验证

此身份验证方法使用PAM（可插拔身份验证模块）作为身份验证机制。默认的PAM服务名称是pgpool。使用执行Pgpool-II的主机上的用户信息支持PAM身份验证。有关PAM的更多信息，请阅读Linux PAM页面。

要启用PAM身份验证，您需要在系统的PAM配置目录（通常位于“/etc/PAM.d”）中为Pgpool-II创建一个服务配置文件。示例服务配置文件作为“share/pgpool II/pgpool.pam”安装在安装目录下。

> [!CAUTION]
> 注意：要启用PAM支持，Pgpool-II必须配置为“--with-PAM”

### 6.2.7. LDAP身份验证

此身份验证方法使用LDAP作为密码认证方法。LDAP仅用于验证用户名/密码对。因此，在LDAP可用于身份验证之前，用户必须已经存在于数据库中。

LDAP身份验证可以在两种模式下运行。在第一种模式中，我们称之为简单绑定模式，服务器将绑定到构造为前缀用户名后缀的可分辨名称。通常，前缀参数用于在Active Directory环境中指定cn=或DOMAIN\。后缀用于指定非Active Directory环境中DN的剩余部分。

在第二种模式中，我们将称之为搜索+绑定模式，服务器首先使用ldapbinddn和ldapbindpasswd指定的固定用户名和密码绑定到LDAP目录，并对试图登录数据库的用户进行搜索。如果没有配置用户和密码，将尝试对目录进行匿名绑定。搜索将在ldapbasedn的子树上执行，并将尝试与ldapsecharattribute中指定的属性进行精确匹配。在此搜索中找到用户后，服务器将断开连接，并使用客户端指定的密码作为该用户重新绑定到目录，以验证登录是否正确。此模式与其他软件（如Apache mod_authnz_LDAP和pam_LDAP）中LDAP身份验证方案使用的模式相同。此方法允许用户对象在目录中的位置具有更大的灵活性，但会导致与LDAP服务器建立两个单独的连接。

以下配置选项在两种模式下都使用：

- ldapserver

	要连接的LDAP服务器的名称或IP地址。可以指定多个服务器，用空格分隔。

- ldapport

	LDAP服务器上要连接的端口号。如果未指定端口，将使用LDAP库的默认端口设置。

- ldapscheme

	设置为ldaps以使用ldaps。这是一种通过SSL使用LDAP的非标准方式，一些LDAP服务器实现支持这种方式。另请参阅ldaptls选项以获取替代方案。

- ldaptls

	设置为1，使Pgpool-II和LDAP服务器之间的连接使用TLS加密。这根据RFC 4513使用StartTLS操作。另请参阅ldapscheme选项以获取替代方案。

请注意，使用ldapscheme或ldaptls仅加密Pgpool-II服务器和LDAP服务器之间的流量。Pgpool-II服务器和客户端之间的连接仍然是未加密的，除非也使用SSL。

以下选项仅在简单绑定模式下使用：

- ldapprefix

	在进行简单绑定身份验证时，在形成要绑定为的DN时，在用户名前添加字符串。

- ldapsuffix

	在进行简单绑定身份验证时，在形成要绑定为的DN时附加到用户名的字符串。

以下选项仅在搜索+绑定模式下使用：

- ldapbasedn

	执行搜索+绑定身份验证时，在中开始搜索用户的根DN。

- ldapbinddn

	在执行搜索+绑定身份验证时，要绑定到目录以执行搜索的用户的DN。

- ldapbindpasswd

	用户在执行搜索+绑定身份验证时绑定到目录以执行搜索的密码。

- ldapsearchattribute

	执行搜索+绑定身份验证时，与搜索中的用户名匹配的属性。如果没有指定属性，则将使用uid属性。

- ldapsearchfilter

	执行搜索+绑定身份验证时使用的搜索筛选器。$username的出现将被用户名替换。这允许比ldapsecharattribute更灵活的搜索过滤器。

- ldapurl

	RFC 4516 LDAP URL。这是一种以更紧凑和标准的形式编写其他LDAP选项的替代方法。格式为

	```shell
	ldap[s]://host[:port]/basedn[?[attribute][?[scope][?[filter]]]]
	```

	作用域必须是base、one、sub之一，通常是最后一个。（默认值是base，这在这个应用程序中通常没有用。）属性可以指定一个属性，在这种情况下，它被用作ldapsecharattribute的值。若属性为空，则过滤器可以用作ldapsearchfilter的值。

	URL方案ldaps选择ldaps方法通过SSL进行LDAP连接，相当于使用ldapscheme=ldaps。要使用StartTLS操作使用加密的LDAP连接，请使用正常的URL方案LDAP，并在ldapurl之外指定ldaptls选项。

	对于非匿名绑定，必须将ldapbinddn和ldapbindpasswd指定为单独的选项。

- backend_use_passwd

	设置为1，使用于LDAP身份验证的密码使用Pgpool-II和后端之间的身份验证。

将简单绑定的配置选项与搜索+绑定的选项混合使用是错误的。

使用搜索+绑定模式时，可以使用ldapsearch属性指定的单个属性或使用ldapsearch过滤器指定的自定义搜索过滤器来执行搜索。指定ldapsearchattribute=foo等同于指定ldapsearchfilter=“（foo=$username）”。如果两个选项都没有指定，则默认值为ldapsecharattribute=uid。

如果Pgpool-II是使用OpenLDAP作为LDAP客户端库编译的，则可以省略ldapserver设置。在这种情况下，通过RFC 2782 DNS SRV记录查找主机名和端口列表。名称_ldap_tcp。查找域，从ldapbasedn中提取域。

以下是一个简单绑定LDAP配置的示例：

```shell
host ... ldap ldapserver=ldap.example.net ldapprefix="cn=" ldapsuffix=", dc=example, dc=net"
```

当请求以数据库用户foo的身份连接到数据库服务器时，Pgpool-II将尝试使用DN cn=foo，dc=example，dc=net和客户端提供的密码绑定到LDAP服务器。如果连接成功，则授予数据库访问权限。

以下是一个搜索+绑定配置的示例：

```shell
host ... ldap ldapserver=ldap.example.net ldapbasedn="dc=example, dc=net" ldapsearchattribute=uid
```

当请求以数据库用户foo的身份连接到数据库服务器时，Pgpool-II将尝试匿名绑定（因为未指定ldapbinddn）到LDAP服务器，在指定的基DN下搜索（uid=foo）。如果找到一个条目，它将尝试使用找到的信息和客户端提供的密码进行绑定。如果第二个连接成功，则授予数据库访问权限。

以下是以URL形式编写的搜索+绑定配置：

```shell
host ... ldap ldapurl="ldap://ldap.example.net/dc=example,dc=net?uid?sub"
```

其他一些支持LDAP身份验证的软件使用相同的URL格式，因此更容易共享配置。

以下是一个搜索+绑定配置的示例，该配置使用ldapsearchfilter而不是ldapsearchattribute来允许通过用户ID或电子邮件地址进行身份验证：

```shell
host ... ldap ldapserver=ldap.example.net ldapbasedn="dc=example, dc=net" ldapsearchfilter="(|(uid=$username)(mail=$username))"
```

以下是一个搜索+绑定配置的示例，该配置使用DNS SRV发现来查找域名example.net的LDAP服务的主机名和端口：

```shell
host ... ldap ldapbasedn="dc=example,dc=net"
```

> [!NOTE]
> 提示：由于LDAP经常使用逗号和空格来分隔DN的不同部分，因此在配置LDAP选项时通常需要使用双引号参数值，如示例所示。

> [!CAUTION]
> 注意：要启用LDAP支持，Pgpool-II必须配置为“--with-LDAP”

### 6.2.8. GSSAPI身份验证

GSSAPI是RFC 2743中定义的用于安全身份验证的行业标准协议。目前Pgpool-II不支持GSSAPI。客户端不应该使用GSSAPI身份验证，或者应该使用“如果可能的话，首选GSSAPI身份认证”选项（这是PostgreSQL客户端的默认设置）。若选择后者，Pgpool-II会向客户端请求非GSSAPI身份验证，客户端将退回到非GSSAPI的身份验证方法。因此，通常用户不需要担心Pgpool-II不接受GSSAPI身份验证。

## 6.3. 使用不同的方法进行前端和后端身份验证

自Pgpool-IIV4.0以来，可以对客户端应用程序和后端PostgreSQL服务器使用不同的身份验证。例如，客户端应用程序可以使用scram-sha-256连接到Pgpool-II，Pgpool-II反过来可以使用信任或md5身份验证连接到PostgreSQL后端进行同一会话。

## 6.4. 在pool_passwd中使用AES256加密密码

SCRAM身份验证可防止中间人类型的攻击，因此Pgpool-II需要用户密码才能通过PostgreSQL后端进行身份验证。

然而，将明文密码存储在“pool_passwd”文件中并不是一个好主意。

您可以存储AES256加密密码，用于身份验证。首先使用用户提供的密钥使用AES256加密对密码进行加密，然后对加密的密码进行base64编码，并在编码字符串中添加AES前缀。

> [!CAUTION]
> 注意：您可以使用pg_enc实用程序创建格式正确的AES256加密密码。

### 6.4.1. 创建加密密码条目

pg_enc可用于在pool_passwd文件中创建AES加密的密码条目。pg_enc需要密钥来加密密码条目。稍后，Pgpool-II将需要相同的密钥来解密用于身份验证的密码。

> [!CAUTION]
> 注意：Pgpool-II必须使用SSL（--with-openssl）支持构建，才能使用加密密码功能。

### 6.4.2. 为Pgpool-II提供解密密钥

如果你在pool_passwd文件中存储了AES加密的密码，那么Pgpool-II在使用密码之前将需要解密密钥来解密密码，Pgpool-II会在启动时尝试从.pgpoolkey文件中读取解密密钥。pgpoolkey是一个纯文本文件，其中包含解密密钥字符串。

默认情况下，Pgpool-II将在用户的主目录中查找.pgpoolkey文件或环境变量PGPOOLKEYFILE引用的文件。您还可以使用pgpool命令的（-k，--key file=key_file）命令行参数指定密钥文件。.pgpoolkey的权限必须禁止对世界或组的任何访问。通过命令chmod 0600~/.pgpoolkey更改文件权限。

