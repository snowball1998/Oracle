# Oracle实验二： 用户管理 - 掌握管理角色、权根、用户的能力，并在用户之间共享对象
## 实验步骤：
- 第一步：以system登录到pdborcl，创建角色yym和用户yym1，并授权和分配空间：
```SQL
yym@DESKTOP-1R80DSI MINGW32 ~/Desktop
$ ssh oracle@202.115.82.8
oracle@202.115.82.8's password:
[oracle@deep02 ~]$ sqlplus system/123@pdborcl

SQL*Plus: Release 12.1.0.2.0 Production on 星期四 11月 8 17:00:43 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

上次成功登录时间: 星期四 11月 08 2018 16:58:00 +08:00

连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> CREATE ROLE yym;

角色已创建。

SQL> Role created.
SP2-0734: 未知的命令开头 "Role creat..." - 忽略了剩余的行。
SQL> GRANT connect,resource,CREATE VIEW TO yym;

授权成功。

SQL> CREATE USER yym1 IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;

用户已创建。

SQL> ALTER USER yym1 QUOTA 50M ON users;

用户已更改。

SQL> GRANT yym TO yym1;

授权成功。

SQL> exit
从 Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options 断开


```
- 第二步：新用户yym1连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
```SQL
[oracle@deep02 ~]$ sqlplus yym1/123@pdborcl

SQL*Plus: Release 12.1.0.2.0 Production on 星期四 11月 8 17:15:47 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> show user;
USER 为 "YYM1"
SQL> CREATE TABLE mytable (id number,name varchar(50));

表已创建。

SQL> INSERT INTO mytable(id,name)VALUES（1，'zhang');

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
从 Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options 断开

```

- 第三步：用户hr连接到pdborcl，查询new_user授予它的视图myview
```SQL
[oracle@deep02 ~]$ sqlplus hr/123@pdborcl

SQL*Plus: Release 12.1.0.2.0 Production on 星期四 11月 8 17:22:24 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

上次成功登录时间: 星期四 11月 08 2018 17:18:34 +08:00

连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> SELECT * FROM yym1.myview;

NAME
--------------------------------------------------------------------------------
zhang
wang

SQL> exit
从 Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options 断开

```
- 第四步：查看数据库的使用情况
```SQL
[oracle@deep02 ~]$ sqlplus system/123@pdborcl  

SQL*Plus: Release 12.1.0.2.0 Production on 星期四 11月 8 17:24:57 2018 

Copyright (c) 1982, 2014, Oracle.  All rights reserved.  

上次成功登录时间: 星期四 11月 08 2018 17:22:24 +08:00 
连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options  
SQL> SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';  
TABLESPACE_NAME

FILE_NAME

        MB     MAX_MB AUTOEXTEN
---------- ---------- ---------
USERS
/home/oracle/app/oracle/oradata/orcl/pdborcl/SAMPLE_SCHEMA_users01.dbf
         5 32767.9844 YES


SELECT from (SELECT tablespace_name,Sum(bytes)free
SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
  8   where  a.tablespace_name = b.tablespace_name;

表空间名
---------- ----------------------------------- ---------------------------------
    大小MB     剩余MB     使用MB    使用率%
---------- ---------- ---------- ----------
SYSAUX
       630         37        591      93.81

USERS
         5        1.3        3.5         73

SYSTEM
       270     3.5625   266.4375      98.68


表空间名
---------- -------------------------------- ------------------------------------
    大小MB     剩余MB     使用MB    使用率%
---------- ---------- ---------- ----------
EXAMPLE
  1281.875      61.79   1220.085      95.18  
  
  
```

