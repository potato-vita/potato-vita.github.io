---
title: 【Seed-Labs 2.0】SQL Injection Attack Lab
tags:
  - SQL Injection
  - Web Security
  - Seed-Labs
  - Database Security
categories:
  - Security
mathjax: true
abbrlink: d33c94e
date: 2025-12-09 22:57:47
description: Seed-Labs 2.0 SQL注入攻击实验室，学习SQL注入漏洞和防御方法
---

# 【Seed-Labs 2.0】SQL Injection Attack Lab

### Overview

SQL 注入是一种代码注入技术，利用的是网络应用程序与数据库服务器之间接口的漏洞。当用户输入的信息在发送到后端数据库服务器之前没有在网络应用程序中进行正确检查时，就会出现这种漏洞。

许多网络应用程序从用户那里获取输入，然后使用这些输入构建 SQL 查询，以便从数据库中获取信息。网络应用程序还使用 SQL 查询将信息存储到数据库中。这些都是开发网络应用程序的常见做法。如果不仔细构造 SQL 查询，就会出现 SQL 注入漏洞。SQL 注入是对网络应用程序最常见的攻击之一。

在本实验室中，我们创建了一个易受 SQL 注入攻击的网络应用程序。我们的网络应用程序包含许多网络开发人员常犯的错误。学生的目标是找到利用 SQL 注入漏洞的方法，演示攻击可能造成的破坏，并掌握有助于防御此类攻击的技术。本实验涵盖以下主题：

- SQL 语句： SELECT 和 UPDATE 语句

- SQL 注入
- 预编译语句（Prepared statement）

### LabEnvironment.

- 虚拟机环境：SEED-labs-ubuntu 20.04

- 使用软件：Oracle VM VirtualBox7.1.0

### LabEnvironment Setup

说在前面：
在开始之前请确认自己的实验环境中是否有其他的docker镜像或是mysql数据库。有的话建议先删除再开始实验。

1.DNS 定向确认。打开 /etc/hosts 查看是否建立正确的映射。

![image-20251209170558677](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230150683.png)

2.使用docker建立网络环境。

![image-20251209170710796](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230150866.png)

3.查看docker的id。

![image-20251209171018053](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230150775.png)

4.启动网络服务。

![image-20251209171041174](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230150936.png)

5.启动过程中发现有报错，随即打开对应的网站，发现是默认网页。

![image-20251209171741154](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230150991.png)

这个报错的解决方案参考satckoverflow的帖子得出：
apache - Getting message AH00558: apache2: - Stack Overflow
注： 这个错误消息表示 Apache HTTP 服务器无法确定其完全合格域名（FQDN）。要解决这个问题并消除警告，我们需要在apache的配置文件中写清楚我们的 ServerName 为 10.9.0.5。

6.修改 image_www/apache_sql_injection.conf 将ServerName 换成已建立DNS定向映射的网站后保存。

```
<VirtualHost *:80>
        DocumentRoot /var/www/SQL_Injection
        ServerName   www.SeedLabSQLInjection.com
</VirtualHost>

```

7.进入docker后找到apache2的配置文件 etc/apache2/apache2.conf，在文件的最后加上我们使用的 10.9.0.5。

8.全部保存后，成功打开网页。

![image-20251210103222739](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230151045.png)

### Task 1: Get Familiar with SQL Statements

**Part1:熟悉mysql的基本操作.**

1.进入装载mysql的docker，然后登录root账号，进入mysql client。

![image-20251210103637903](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230153182.png)

2.加载已存在的 sqllab_users 数据库，并查看该数据库中的表。

![image-20251210103833925](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230154486.png)

**Part2:使用SQL命令打印Alice的所有信息。**

1.使用SQL语句打印 Alice的所有信息。

```
SELECT * FROM credential WHERE Name=’Alice’;
```

![image-20251210104036852](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230155539.png)

### Task 2: SQLInjection Attack on SELECT Statement

#### Task 2.1: SQL Injection Attack from webpage.

任务： 在不知道admin的密码的情况下，登录进该账号并查看。

1.因为我们只知道账号 admin，因此在username的输入上思考，查看原始代码，注意到逻辑的登录逻辑是同时对账号和密码在数据库中进行匹配。我们应该想办法通过输入username来忽略输入密码的部分。

```
WHERE name= '$input_uname' and Password='$hashed_Pwd'";
```

2.“忽略”的实现可以通过注释掉后面输入密码的部分来实现，因此尝试输入admin’#，以构造目标结构，结构如下：

```
WHERE name='admin'# ' and Password='$hased_pwd'";
```

3.尝试输入 admin'#。

![image-20251210104534980](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230156525.png)

4.成功登录，能够查看到所有的数据。

![image-20251210104647241](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230157452.png)

#### Task 2.2: SQL Injection Attack from command line.

任务： 不使用web页面，只用shell界面进行2.1任务。

1.参考题目提供代码，使用shell发送请求，成功获得返回值。为方便阅读将结果保存下来，这样可以更清晰的看见结果。

注：题目上提到要将所有的符号进行编码，#的编码形式为%23。

![image-20251210105156231](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230158506.png)

#### Task 2.3: Append a new SQL statement.

任务 一次执行两个sql语句实现对数据库的内容进行更改。

1.输入

```
admin'; UPDATE credential SET Name='Austin' WHERE ID = '1';#
```

![image-20251210105530789](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230159504.png)

2.发现无法成功达成预期效果，观察题目提供的原始代码，可知程序使用的是query() 函数运行我们输入的sql语句， 然而PHP 中 mysqli 扩展的 query（） 函数不允许在数据库服务器中运行多条语句，所以我们会执行失败，这也是本网站的针对sql注入攻击的一种countermeasure。

3.为解决该问题，最直接的方法就是不使用 query() 函数，换成能够执行多条语句的函数。进入docker，找到var/www/SQL_Injection/目录下的unsafe_home.php，在其中更改 query() 函数为 multi_query() 函数（共有两处）。

4.保存更改后，重新使用之前的语句进行攻击，可以看到成功执行输入的两条sql语句，数据库的内容成功被篡改。

```
输入的语句：admin'; UPDATE credential SET Name='Austin' WHERE ID = '1';#
欲实现将 `Name`=’Alice’的行的’Alice’改为’Austin’
```

![image-20251210111351241](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230200520.png)

注：此处应在mysql中直接查看credential数据库的值，因为query()和multi_query()的函数返回值不同，所以执行新的语句后，不会有信息显示在webpage上。

### Task 3: SQLInjection Attack on UPDATE Statement

#### Task 3.1: Modify your own salary.

任务： 使用Alice的账号，在个人信息编辑页面利用update增加salary的值。

在这之前我们要意识到在task2中我们把Alice改成了Austin，因此还需要先改回来

1.观察题目所给代码，发现我们可以按照TASK2的思路，使用’使上一句闭合后，嵌入我们想要的语句。随即找一个注入点，修改salary=10000000。

```
aaa',Salary='88888888
```

![image-20251210112649653](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230201236.png)

2.注入完成后，查看salary确实修改成功。

![image-20251210112709100](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230202007.png)

#### Task 3.2: Modify other people’ salary.

任务： 将Boby的salary降为1。

1.观察参考代码，发现定位用的 WHERE 语句我们无法直接修改，所以参考 TASK2 的思路将后面的语句直接用#注释掉。

![image-20251210114320396](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230202742.png)

2.随机找一个注入点输入下面的内容后保存。

```
aaa',Salary=1 Where ID=2#
```

![image-20251210114515277](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230203527.png)

3.可以看到Boby的Salay已被成功更改。

![image-20251210143624106](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230204433.png)

#### Task 3.3: Modify other people’ password.

任务： 将Boby的Password修改成我们设定的值。

1.根据题目提示，在数据库中存储的password是经过SHA1加密后的结果。所以我们应将我们设定的Password的plaintext经过SHA1加密后的结果存进数据库。我们设定 Austin为Password的plaintext。

[SHA1 在线加密工具 | 菜鸟工具](https://www.jyshare.com/crypto/sha1/)

![image-20251210143818259](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230205421.png)

2.将对应的密码存入数据库。

3.成功使用修改后的密码登录进入Boby的账号。

![image-20251210144707113](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230206500.png)

### Task 4: Countermeasure — Prepared Statement

预处理语句 （Prepared Statement）的工作方式

SQL 注入漏洞的根本问题在于未能将代码与数据分开。在编写 SQL 语句时，程序（如 PHP 程序）知道哪一部分是数据，哪一部分是代码。不幸的是，当 SQL 语句被发送到数据库时，边界已经消失；SQL 解释器看到的边界可能与开发人员设置的原始边界不同。要解决这个问题，必须确保服务器端代码和数据库中的边界视图保持一致。最安全的方法是使用准备语句。

要了解准备语句如何防止 SQL 注入，我们需要了解 SQL 服务器接收查询时会发生什么。执行查询的高级工作流程如图 3 所示。在编译步骤中，查询首先要经过解析和规范化阶段，即根据语法和语义检查查询。下一阶段是编译阶段，在这一阶段，关键字（如 SELECT、FROM、UPDATE 等）被转换成机器可以理解的格式。基本上，在这一阶段，查询会被解释。在查询优化阶段，会考虑多种不同的查询执行计划，从中选出最佳优化计划。被选中的计划会存储在缓存中，因此每当有下一个查询进来时，就会根据缓存中的内容进行检查；如果缓存中已经存在该计划，则会跳过解析、编译和查询优化阶段。然后，编译后的查询将被传递到执行阶段，并在那里得到实际执行。

预处理语句出现在编译之后，执行步骤之前。预编译语句将经过编译步骤，并变成一个预编译查询，其中的数据占位符为空。要运行这个预编译查询，需要提供数据，但这些数据不会经过编译步骤，而是直接插入预编译查询，然后发送到执行引擎。因此，即使数据中包含 SQL 代码，在不经过编译步骤的情况下，代码也会被简单地视为数据的一部分，没有任何特殊含义。这就是准备语句防止 SQL 注入攻击的方法。

下面是一个如何在 PHP 中编写准备语句的示例。在下面的示例中，我们使用了 SELECT 语句。我们展示了如何使用准备语句重写易受 SQL 注入攻击的代码。

```
 $sql = "SELECT name, local, gender
	 	FROM USER_TABLE
	 	WHERE id = $id AND password =’$pwd’ ";
 $result = $conn->query($sql)
```

上述代码容易受到 SQL 注入攻击。可将其重写如下

```
$stmt = $conn->prepare("SELECT name, local, gender
 		FROM USER_TABLE
		 WHERE id = ? and password = ? ");
 // Bind parameters to the query
 $stmt->bind_param("is", $id, $pwd);
 $stmt->execute();
 $stmt->bind_result($bind_name, $bind_local, $bind_gender);
 $stmt->fetch();
```

利用预处理语句机制，我们将向数据库发送 SQL 语句的过程分为两步。第一步是只发送代码部分，即不包含实际数据的 SQL 语句。这就是准备步骤。从上面的代码片段中我们可以看到，实际数据被问号（?） 完成这一步后，我们使用 bindparam() 将数据发送到数据库。数据库只会将这一步发送的所有数据视为数据，而不再是代码。它会将数据与准备语句中相应的问号绑定。在 bindparam() 方法中，第一个参数 “is ”表示参数的类型： i “表示 $id 中的数据为整数类型，”s "表示 $pwd 中的数据为字符串类型。

![image-20251210144951106](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230207544.png)

任务： 进入 www.seedlabsqlinjection.com/defense，通过修改 unsafe.php使该网站能够防御SQL注入攻击。

1.进入该网站，对其进行SQL攻击测试，发现其能够被SQL注入攻击。

![image-20251210145053170](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230208449.png)

![image-20251210145109678](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230209321.png)

2.选择题目中的第二个方法在运行中的容器中进行修改，进入docker，进入/var/www/SQL_Injection/defense，编辑unsafe.php。

修改前：

```
<?php
$result = $conn->query("SELECT id, name, eid, salary, ssn
                        FROM credential
                        WHERE name= '$input_uname' and Password= '$hashed_pwd'");
```

修改后：

```
<?php
$stmt = $conn->prepare("SELECT id, name, eid, salary, ssn FROM credential WHERE name= ? and Password= ?");
$stmt->bind_param("ss", $input_uname, $hashed_pwd);
$stmt->execute();
$result = $stmt->get_result();
```

3. 再次尝试使用单引号攻击，发现攻击失败，成功防御。

![image-20251210145436251](https://potatojiang.oss-cn-chengdu.aliyuncs.com/20251210230210276.png)