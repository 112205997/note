## 基本概念

域 Domain

活动目录 Active Directory 简称AD

域控制器 Domain Controller 简称DC（ADDS服务）

域控制器集群

1、AD数据库同步

2、RODC （read only Domain controller）只读域控制器 ，用于分支机构

3、GC（Global Category Server）全局编录服务器，用于多域环境中跟其它域进行AD的部分数据同步（例如Exchange Server）

-----

1 对象与属性 Object attribute ：ADDS内的资源以对象形式存在

2 容器Container与组织单位 OU：包含对象和容器，可以添加组策略（group policy）

3 架构Schema：一个林内所有的域树共享相同的架构。可以理解为表结构

4 DC：存储域（ADDS）目录服务，一个域可以有多个域控制器

5 DNS与DC：域环境需要支持ADDS的dns服务器，其它服务可以通过DNS找到该服务。AD需要符合域名结构，通常是部署在同一台机器。

6 LDAP（轻型目录访问协议）：查询与更新ADDS的目录服务通讯协议。

​		1）DN（Distinguished Name）：完整路径，包括CN、OU（组织单位）、DC（Domain componet）

​    	2）RDN（relative Distinguished Name）：相对路径

​		3）GUID：对象唯一ID

​    	4）UPN（user principal Name）：DN的短账号 例如 **@qq.com

​		5）SPN（Service principal Name）：根据DNS主机名建立。表述某台计算机支持的服务

7 站点（site）：由1或多个IP子网组成。域是逻辑分组，站点是实体分组



8 目录分区（directory partition）：

 AD DS数据库逻辑分为4个目录：

 架构目录分区：

 配置目录分区：

 域目录分区：

应用程序目录分区：



## 域的部署规划

1、单域

  单域可以包含100万个对象

2、多域

  1 IT管控边界

  2 公司在重组合并

  3 域的改造和迁移

域林 包含多个域

父域和子域的命名空间需保持相同和连续

![image-20200225140938919](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200225140938919.png)





![image-20200225141120466](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200225141120466.png)

![image-20200225141301800](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200225141301800.png)

![image-20200225141447567](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200225141447567.png)

命令

1 添加域管理员 

强制刷新组策略更新 gpupdate /force  ：更改组策略管理-》域控制器策略（Default Domain Controllers Policy），添加新的域管理员

2 CSVD批量创建和修改域用户

 csvd -i -f  C:\?***.txt  以为该导入方式不设置密码所以用户必须是禁用状态 userAccountControl为 514

![image-20200226115151701](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200226115151701.png)



启用使用 net user  ***（用户名） ***（密码）命令，写成bat脚本