---
title: Mysql 问题汇总
date: 2017-09-05 10:55:11
categories: mysql
type: "categories"
comments: false
tags:
- Mysql
---
### Mysql问题汇总
这里总结一下，在使用Mysql中可能会遇到的一些问题。

如果mysql忘记登陆密码，可以通过忽略密码先登陆进入，
在mysqld的后面加入–skip-grant-tables来重新启动msyql服务器，
mysqld –skip-grant-tables
或者关闭mysql服务然后在my.ini(Windows)或my.cnf中的[mysqld]内添加
skip-grant-tables
再重新启动mysql就可以了。
现在可以直接mysql连接进去了。

修改密码，有几种方式，
使用mysqladmin 修改密码，
mysqladmin -u root password “123456”;

使用SET PASSWORD，修改如果是忽略密码模式这个可能会无效，使用这个命令不需要flush privileges刷新
mysql>SET PASSWORD FOR ‘root’@’localhost’ = PASSWORD(‘123456’);

用UPDATE直接编辑user表
mysql>USE mysql;
mysql>UPDATE user SET Password = PASSWORD(‘123456′) WHERE user=’root’;
mysql>FLUSH PRIVILEGES;

另外修改用户权限使用
mysql>grant all privileges on *.* to username@localhost identified by ‘password’ with grant option;
mysql>flush privileges;
localhost 可以使用%替换，表示所有机器都可以

另外，对于mysql的字符集，utf8和utfmb4，建议都采用utf8mb4，因为微信里的昵称评论会有表情出现，这是utf编码是4位的，而一般的utf8编码是3位，这时就会出现
java.sql.SQLException: Incorrect string value: ‘\xF0\x9F\x92\x93′
类似的错误，所以尽量采用utf8mb4编码代替utf8编码。
如果是现有数据库，使用sql语句
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
修改当前编码
或者使用navicat等数据库工具修改

同时修改mysql服务器配置
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect=’SET NAMES utf8mb4’
重启服务器
在mysql中使用sql语句
SHOW VARIABLES WHERE Variable_name LIKE ‘character\_set\_%’ OR Variable_name LIKE ‘collation%’;
确认结果
+————————–+——————–+
| Variable_name | Value |
+————————–+——————–+
| character_set_client | utf8mb4 |
| character_set_connection | utf8mb4 |
| character_set_database | utf8mb4 |
| character_set_filesystem | binary |
| character_set_results | utf8mb4 |
| character_set_server | utf8mb4 |
| character_set_system | utf8 |
| collation_connection | utf8mb4_unicode_ci |
| collation_database | utf8mb4_unicode_ci |
| collation_server | utf8mb4_unicode_ci |
+————————–+——————–+

 

如果你用的是java服务器，升级或确保你的mysql connector版本高于5.1.13。

jdbc:mysql://localhost:3306/database?useUnicode=true&characterEncoding=utf8&autoReconnect=true&rewriteBatchedStatements=TRUE

其中的characterEncoding=utf8可以被自动被识别为utf8mb4（当然也兼容原来的utf8），而autoReconnect配置我强烈建议配上

 

一般报错Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Communications link failure
是因为数据库连接超时，可以修改mysql数据库超时时间来处理
mysql﹥ show global variables like ‘wait_timeout’; 可以查看超时时间
mysql> set global wait_timeout=2592000; 可以修改超时时间为2592000，30天
修改配置文件/etc/my.cnf
[mysqld] wait_timeout=2592000

报错com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Data source rejected establishment of connection, message from server: “Too many connections”，一般是设定的并发连接数太少或者系统繁忙导致连接数被占满，
mysql> show global variables like ‘max_connections’; 可以查看连接数
mysql> set global max_connections=1000; 修改连接数为1000
修改配置文件/etc/my.cnf
```java 
[mysqld] max_connections=1000
 ```
---
