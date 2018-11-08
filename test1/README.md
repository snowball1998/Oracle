# Oracle实验二： 用户管理 - 掌握管理角色、权根、用户的能力，并在用户之间共享对象
## 实验步骤：
- 第一步：以system登录到pdborcl，创建角色con_res_view和用户new_user，并授权和分配空间：
```SQL
[oracle@deep02 ~]$ sqlplus system/123@pdborcl

SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 08:55:20 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

上次成功登录时间: 星期三 10月 24 2018 08:55:12 +08:00

连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> CREATE ROLE con_wwh_view;

角色已创建。

SQL>  GRANT connect,resource,CREATE VIEW TO con_wwh_view;

授权成功。

SQL> CREATE USER new_usr IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;

用户已创建。

SQL> ALTER USER new_usr QUOTA 50M ON users;

用户已更改。

SQL> GRANT con_wwh_view TO new_usr;

授权成功。

SQL> exit

```
- 第二步：新用户new_user连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
```SQL
$ sqlplus new_usr/123@pdborcl
SQL> show user;
USER 为 "SYSTEM"
SQL> CREATE TABLE mytable (id number,name varchar(50));

表已创建。

SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');

已创建 1 行。

SQL> INSERT INTO mytable(id,name)VALUES (2,'wang');

已创建 1 行。

SQL> CREATE VIEW myview AS SELECT name FROM mytable;

视图已创建。

SQL> SELECT * FROM myview;

NAME
--------------------------------------------------------------------------------
zhang
wang

SQL> GRANT SELECT ON myview TO hr;

授权成功。

SQL> exit

```

- 第三步：用户hr连接到pdborcl，查询new_user授予它的视图myview
```SQL
$ sqlplus new_usr/123@pdborcl
SQL> SELECT * FROM new_usr.myview;

NAME
--------------------------------------------------------------------------------
zhang
wang
zhang
WANG

SQL> exit

```
