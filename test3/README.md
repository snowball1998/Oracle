**Oracle实验三：创建分区表**  
========
一、第1步：
-------
##### 登录用户yym1：  
[oracle@deep02 ~]$ sqlplus yym1/123@pdborcl  
SQL*Plus: Release 12.1.0.2.0 Production on 星期一 11月 12 12:48:15 2018  
Copyright (c) 1982, 2014, Oracle.  All rights reserved.  
上次成功登录时间: 星期一 11月 12 2018 12:40:56 +08:00  

连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL>    

##### - 创建订单表(orders)与订单详表(order_details)。
orders
 
    CREATE TABLE orders 
    (
    order_id NUMBER(10, 0) NOT NULL 
    , customer_name VARCHAR2(40 BYTE) NOT NULL 
    , customer_tel VARCHAR2(40 BYTE) NOT NULL 
    , order_date DATE NOT NULL 
    , employee_id NUMBER(6, 0) NOT NULL 
    , discount NUMBER(8, 2) DEFAULT 0 
    , trade_receivable NUMBER(8, 2) DEFAULT 0 
    ) 
    TABLESPACE USERS 
    PCTFREE 10 INITRANS 1 
    STORAGE (   BUFFER_POOL DEFAULT ) 
    NOCOMPRESS NOPARALLEL 
    PARTITION BY RANGE (order_date) 
    (
    PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (
    TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
    'NLS_CALENDAR=GREGORIAN')) 
    NOLOGGING 
    TABLESPACE USERS 
    PCTFREE 10 
    INITRANS 1 
    STORAGE 
    ( 
    INITIAL 8388608 
    NEXT 1048576 
    MINEXTENTS 1 
    MAXEXTENTS UNLIMITED 
    BUFFER_POOL DEFAULT 
    ) 
    NOCOMPRESS NO INMEMORY  
    , PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (
    TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
    'NLS_CALENDAR=GREGORIAN')) 
    NOLOGGING 
    TABLESPACE USERS02 
    PCTFREE 10 
    INITRANS 1 
    STORAGE 
    ( 
    INITIAL 8388608 
    NEXT 1048576 
    MINEXTENTS 1 
    MAXEXTENTS UNLIMITED 
    BUFFER_POOL DEFAULT 
    ) 
    NOCOMPRESS NO INMEMORY  
    , PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (
    TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
    'NLS_CALENDAR=GREGORIAN')) 
    NOLOGGING 
    TABLESPACE USERS03 
    );

order_details

    CREATE TABLE order_details   
    (  
    id NUMBER(10, 0) NOT NULL   
    , order_id NUMBER(10, 0) NOT NULL  
    , product_id VARCHAR2(40 BYTE) NOT NULL   
    , product_num NUMBER(8, 2) NOT NULL   
    , product_price NUMBER(8, 2) NOT NULL   
    , CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)  
    REFERENCES orders(order_id)  
    ENABLE   
    )   
    TABLESPACE USERS   
    PCTFREE 10 INITRANS 1    
    STORAGE (   BUFFER_POOL DEFAULT )   
    NOCOMPRESS NOPARALLEL  
    PARTITION BY REFERENCE (order_details_fk1)  
    (  
    PARTITION PARTITION_BEFORE_2016   
    NOLOGGING   
    TABLESPACE USERS  
    PCTFREE 10   
    INITRANS 1   
    STORAGE   
    (   
    INITIAL 8388608   
    NEXT 1048576   
    MINEXTENTS 1   
    MAXEXTENTS UNLIMITED   
    BUFFER_POOL DEFAULT   
    )   
    NOCOMPRESS NO INMEMORY,   
    PARTITION PARTITION_BEFORE_2017   
    NOLOGGING   
    TABLESPACE USERS02  
    PCTFREE 10   
    INITRANS 1   
    STORAGE    
    (   
    INITIAL 8388608   
    NEXT 1048576   
    MINEXTENTS 1   
    MAXEXTENTS UNLIMITED   
    BUFFER_POOL DEFAULT   
    )   
    NOCOMPRESS NO INMEMORY,  
    PARTITION PARTITION_BEFORE_2018   
    NOLOGGING   
    TABLESPACE USERS03  
    PCTFREE 10   
    INITRANS 1   
    STORAGE   
    (   
    INITIAL 8388608   
    NEXT 1048576   
    MINEXTENTS 1   
    MAXEXTENTS UNLIMITED   
    BUFFER_POOL DEFAULT   
    )   
    );  


二、第2步：给用户进行分配可查询的权限： 
---------
![image](https://github.com/snowball1998/Oracle/blob/master/test3/1.png) 

三、第3步：插入一万条数据语句及联合查询语句：  
--------
##### 向两个表中各自插入10000条数据：
    begin
    for i in 1..3000
    loop
    insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'全球',12345,to_date('2017-02-14','yyyy-mm-dd'),i,i);
    end loop;
    commit;
    
    for i in 3001..6000
    loop
     insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'威威',12345,to_date('2015-02-14','yyyy-mm-dd'),i,i);
    end loop;
    commit;
    
    for i in 6001..10000
    loop
     insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'嗯嗯',12345,to_date('2016-02-14','yyyy-mm-dd'),i,i);
    end loop;
    commit;
    
    for j in 1..3000
    loop
    insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',20,100);
    end loop;
    commit;
    
    for j in 3001..6000
    loop
    insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',30,200);
    end loop;
    commit;
    
        for j in 6001..10000
    loop
    insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',40,300);
    end loop;
    commit;
    end;  
    
 ##### 联合查询语句：
    SELECT *
    FROM orders,order_details  Where orders.order_id = order_details.order_id AND
    orders.order_date>=to_date('2016-02-14','yyyy-mm-dd')

    对查询语句，进行分析语句的执行计划：  
--------
##### 联合查询语句执行计划：
![image](https://github.com/snowball1998/Oracle/blob/master/test3/2.png)       

四、分区与不分区的对比分析
--------
分区功能能够将表、索引或索引组织表进一步细分为段，这些数据库对象的段叫做分区。从数据库管理员的角度来看，一个分区后的对象具有多个段，这些段既可进行集体管理，也可单独管理，这就使数据库管理员在管理分区后的对象时有相当大的灵活性。从应用程序的角度来看，分区后的表与非分区表完全相同。
