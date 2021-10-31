### mysql 安装

#### mysql 安装

```sql
linux 下的mysql 的安装启动

mysql下载
     wget http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz
解压
    //解压文件并放置到指定的文件目录
    tar -xzvf mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz
    //复制解压后的mysql的文件夹
    cp -r ysql-5.6.33-linux-glibc2.5-x86_64 /usr/local/mysql
添加用户组和用户
    //添加用户组
    groupadd mysql
    //添加用户组mysql到用户组mysql
    useradd -g mysql mysql
安装mysql
    //切换到mysql的文件目录下
    cd /usr/local/mysql
    mkdir ./data/mysql
    chown -R mysql:mysql ./
    ./scripts/mysql_install_db --user=mysql --datadir=/usr/local/mysql/data/mysql
    cp support-files/mysql.server /etc/init.d/mysqld
    chmod 755 /etc/init.d/mysqld
    cp support-files/my-default.cnf /etc/my.cnf

 修改启动脚本
     vi /etc/init.d/mysqld
     //修改内容如下
     //basedir = /usr/local/mysql/mysql-5.6.33-linux-glibc2.5-x86_64
     //datadir = /usr/local/mysql/data/mysql

启动,查看和关闭服务
    service mysqld start //启动mysql
    service mysqld stop  //关闭mysql
    service mysqld status //查看mysql的状态



```

#### 安装遇到的问题

```sql
常见的错误信息
 [root@ssgao mysql_tar]# service mysqld start
 /etc/init.d/mysqld: line 256: my_print_defaults: command not found
 Starting MySQL ERROR! Couldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)

```

#### 登录配置

```sql
mysql -h 服务器ip地址 -P 3306 -u root -p

连接远程主机上的Mysql
    mysql -h10.10.100.101 -uroot -p123
```

```sql
服务端开启远程访问
// 方法一: 编辑mysql配置文件,把其中的bind-address=127.0.0.1注释掉


 //首先进入mysql数据库,然后输入下面两个命令
  mysql> grant all privileges on *.* to 'root'@'%' identified by 'ssgao1987' with grant option;
    Query OK, 0 rows affected (0.00 sec)
 mysql> flush privileges;
    Query OK, 0 rows affected (0.00 sec)
 > 第一个* 是数据库,可以改成允许访问的数据库名称
 > 第二个 是数据库表名称,代表可以访问任意的表
 > root 代表远程登录使用的用户名,可以自定义
 > % 代表允许任意ip登录,如果你想指定特定的IP,可以把%替换掉就可以了
 > password代表远程登录时使用的密码,可以自定义
 > flush privileges 让权限立即生效。
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635284475939-3f07358d-cbc6-45c6-98bc-4fbde7f7992a.png#clientId=uda4572dc-c25e-4&from=paste&height=76&id=u9035d6ad&margin=%5Bobject%20Object%5D&name=image.png&originHeight=110&originWidth=902&originalType=binary&ratio=1&size=8162&status=done&style=none&taskId=uc4ca44fb-f7e8-45d0-9583-96f1b14be07&width=624)

#### 修改 root 密码

```sql
//方法一: 使用set password密码
    mysql -u root
    mysql> set password for 'root'@'localhost' = password('ssgao');
  或mysql> set password = password ('ssgao1987')
     -> ;
    Query OK, 0 rows affected (0.00 sec)
//方法二: 用mysqladmin
    mysqladmin -u root password 'ssgao'
    如果root已经设置过密码,采用如下方法
    mysqladmin -u root password oldpass 'newpass'

//方法三: 用update直接编辑user表
    mysql -u root
    mysql> use mysql
    mysql> update user set password = password('newpass') where user='root';
    mysql> flush privileges;

```

### mysql 基础内容

#### key 的解释

```sql
key 是数据库的物理结构,包含两层意义和作用
    1> 约束(偏重于约束和规范数据库结构完整性)
    2> 索引(辅助查询用)

primary key
    ** 两个作用,一个是约束作用(constraint)用来规范存储主键和唯一性,但同时在此可以上建立了一个主键索引
    > primary key约束,唯一标识数据库表中的每条记录
    > 主键必须包含唯一的值
    > 主键列不能包含NULL值
    > 每个表都应该有一个主键,并且每个表只能有一个主键(PRIMARY KEY 拥有自动定义的UNIQUE约束)

unique key
    ** 两个作用,一个是约束作用(constraint),规范数据的唯一性,但同时也在key上建立一个唯一索引
    > unique 约束,唯一标识数据库中的每条记录
    > unique和primary key约束均为列或列集合提供了唯一性保证
      (每个表可以有多个UNIQUE约束,但是每个表只能有一个Primary Key约束)

foreign key
    ** 两个作用,一个是约束作用(constraint),规范数据的引用完整性,但同时也在这个key上建立一个index





```

#### 索引

```sql
index是数据库的物理结构,只是辅助查询的,创建时会在另外的表空间(mysql中的innodb表空间),以一个类似目录的结构存储
索引要分类的话,分为前缀索引,全文本索引等;

   索引只是索引,它不会去约束索引的字段行为(那是key要做的事情)

索引分类:
 > 主键索引(必须指定为"primary key")
 > 唯一索引(unique index,一般写成unique key)
 > 普通索引(index, 只有这一种才是纯粹的index)等,也是基于是不是把index看做了key.



```

#### mysql 远程连接

```sql
解决mysql的远程连接的问题
localhost 修改为 %


进入mysql的bin目录中
mysql -u root -p
mysql> use mysql;
mysql> update user set host='%' where user = 'root';
mysql> flush privileges;

具体分析
在本机登入mysql后，更改"mysql"数据库里面的"user"表中的host选项，从localhost修改为"%"。
mysql>
mysql>use mysql;
mysql>select 'host' from user where user='root';

#查看mysql库中的user表的host值(即可进行连接访问的主机/IP名称)
mysql>update user set host='%' where user='root';
#修改host值(以通配符%的内容增加主机/IP地址，当然也可以直接增加某个特定IP地址，
如果执行update语句时出现ERROR 1062(23000); Duplicate entry '%-root' for key 'PRIMARY' 错误，
需要select host from user where user ='root';
#查看一下host是否已经有%这个值，如果有了直接执行下面的flush privileges;即可
mysql> flush priviledges;
mysql> select host,user from user where user='root';
mysql> quit;

```

#### mysql 类型

```sql
char和varchar类型
 char和varchar很类似,都是用来保存mysql中较短的字符串
 主要区别在于：
    char列的长度固定为创建表时声明的长度可以为从0~255的任何值
    varchar的值可以使变长字符串,长度可以指定为0~65535之间的值,检索的时候char列会删除尾部的空格,而varchar则保留了这些空格

```

#### mysql 管理操作

```sql
mysql 显示所有的数据库
    show databases;   //显示所有的数据库
    use dbname;       //选择数据库
    show tables;      //显示所有表信息
    desc tablename;   //查看某个表的详细信息,包括列名

显示库中的所有的数据表
   > use mysql;
   > show tables;

显示数据表的结构
   > describe 表名;
创建数据库信息
   > create databse 表名;
   > create table 表名 (字段设定列表);
删除数据库和表
   > drop database 库名;
   > drop table 表名;
将表中记录清空
   > delete from 表名;

查看当前使用的数据库
    mysql> select database();
        +------------+
        | database() |
        +------------+
        | test       |
        +------------+
        1 row in set (0.00 sec)

查看当前数据使用的端口
    mysql> show variables like 'port';
        +---------------+-------+
        | Variable_name | Value |
        +---------------+-------+
        | port          | 3306  |
        +---------------+-------+
        1 row in set (0.00 sec)

 查看数据文件存放路径
     mysql> show variables like '%datadir%';
        +---------------+------------------------------+
        | Variable_name | Value                        |
        +---------------+------------------------------+
        | datadir       | /usr/local/mysql/data/mysql/ |
        +---------------+------------------------------+
        1 row in set (0.00 sec)

 查看数据库的最大连接数
    mysql> show variables like '%max_connections%';
        +-----------------+-------+
        | Variable_name   | Value |
        +-----------------+-------+
        | max_connections | 151   |
        +-----------------+-------+
        1 row in set (0.00 sec)

 查看数据库当前连接数,并发数
    mysql> show status like 'Threads%';
        +-------------------+-------+
        | Variable_name     | Value |
        +-------------------+-------+
        | Threads_cached    | 1     | //代表当前此时此刻线程缓存中有多少空闲线程
        | Threads_connected | 5     | //代表当前已建立的数量,因为一个连接就需要一个线程,所有也可以看成当前被使用的线程数
        | Threads_created   | 6     | //代表从最近一次服务启动,已经创建线程的数量
        | Threads_running   | 1     | //代表当前激活(非睡眠态)的线程数。并不代表正使用的线程数,有时连接已经建立,但连接处于sleep状态。
        +-------------------+-------+
        4 rows in set (0.00 sec)
```

#### mysql 库表操作

##### 库操作

```sql
//使用create 命令创建数据库
mysql 创建数据库: create databse 数据库名;
首先登录mysql
    [root@ssgao etc]# mysql -u root -p
    Enter password: (//输如密码)
    mysql> create database ssgao;
        Query OK, 1 row affected (0.00 sec)


//使用mysqladmin创建数据库
    使用普通用户的时候,我们可能需要特定的权限来创建或删除Mysql数据库。
    所以我们使用root用户登录,root用户拥有最高权限,可以使用 mysqladmin 命令来创建数据库。如下创建名为aouo的数据库
    [root@ssgao etc]# mysqladmin -u root -p create aouo
    Enter password:
    [root@ssgao etc]#

在mysql>提示窗口中,可以很简单的选择指定的数据库。我们可以使用SQL命令来选择指定的数据库
		[root@ssgao etc]# mysql -u root -p
    Enter password:
    mysql> show databases; //查看当前所有的数据库
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | aouo               |
    | mysql              |
    | performance_schema |
    | ssgao              |
    | test               |
    +--------------------+
    6 rows in set (0.00 sec)

mysql> use ssgao; //选择指定的数据库,注意mysql中数据库名,表名,字段名都是区分大小写的。所以我们使用SQL命令的时候需要输入正确的名称
Database changed

```

##### 表操作

```sql
CREATE TABLE `schedule_item_targeting` (
    ->   `id` bigint(20) NOT NULL AUTO_INCREMENT,
    ->   `uid` bigint(20) NOT NULL COMMENT '记录唯一uid',
    ->   `schedule_item_uid` bigint(50) DEFAULT NULL COMMENT '排期条目UID',
    ->   `target_type` int(10) NOT NULL COMMENT '定向类型，如1-性别，2-地域',
    ->   `target_value`  varchar(512) NOT NULL COMMENT '定向类型内容' ,
    ->   `status` int(10) NOT NULL COMMENT '状态，1-正常，2-更新删除，3-快照删除',
    ->   `db_create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ->   `db_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ->   PRIMARY KEY (`id`),
    ->   UNIQUE KEY `uk_uid` (`uid`),
    ->   UNIQUE KEY `uk_siu_target` (`schedule_item_uid`,`target_type`), // `列名` 之间不可以存在空格
    ->   KEY `idx_name` (`schedule_item_uid`,`target_type`,`status`)
    ->
    -> ) ENGINE=InnoDB AUTO_INCREMENT=514 DEFAULT CHARSET=utf8mb4 COMMENT='排期条目的定向条件表'
    -> ; //AUTO_INCREMENT=514 表示自增的键初始值为514
Query OK, 0 rows affected (0.01 sec)



```

```sql
查看表的列名和列注释
    SELECT
	    s.COLUMN_NAME,
	    s.COLUMN_COMMENT
    FROM
	    information_schema.`COLUMNS` s
    WHERE
	    s.TABLE_NAME = '表名'
        AND s.TABLE_SCHEMA = '表名';

> 修改表的名称
  mysql> alter table test_a rename to test_b;

> 删除表字段
  mysql> alter table schedule_item drop has_targeting;
    Query OK, 0 rows affected (0.10 sec)
    Records: 0  Duplicates: 0  Warnings: 0


> 表添加列名
    mysql> alter table schedule_item add schedule_traffic_type int(10) default null comment '排期流量类型';
    Query OK, 0 rows affected (0.02 sec)
    Records: 0  Duplicates: 0  Warnings: 0


> 修改原有字段名称以及类型
   alter table 表名 change "old列名" "new列名" 类型 默认 comment 注释;
   mysql> alter table emplyee change name_old name_new int(20) default 1 comment '名称编码';
    Query OK, 0 rows affected (0.06 sec)
    Records: 0  Duplicates: 0  Warnings: 0

> 修改原有字段名的注释 (没有单独字段注释的命令)
  mysql> alter table schedule_item modify column show_time bigint(20) default 0 comment '播放时长';
    Query OK, 0 rows affected (0.01 sec)
    Records: 0  Duplicates: 0  Warnings: 0
  // 设置默认非空
  mysql> alter table schedule_item modify column  schedule_item_name varchar(200) not null  COMMENT '排期条目名称';
    Query OK, 203 rows affected, 203 warnings (0.02 sec)
    Records: 203  Duplicates: 0  Warnings: 203


> 在某个字段后增加一个字段
  mysql> alter table schedule_item add schedule_traffic_type int(10) DEFAULT NULL COMMENT '排期流量类型' after amount_daily;
     Query OK, 0 rows affected (0.02 sec)
     Records: 0  Duplicates: 0  Warnings: 0
  // 设置默认非空
  mysql> alter table schedule_item add schedule_traffic_type int(10) NOT NULL COMMENT '排期流量类型' after amount_daily;

> 修改字段位置
  mysql> alter table schedule_item modify schedule_item_name varchar(150) after frequence_limit;
    Query OK, 0 rows affected (0.09 sec)
    Records: 0  Duplicates: 0  Warnings: 0


```

###

```sql
> 添加主键索引(add primary key)
  mysql> alter table 'table_name' add primary key ('column');


> 添加unique(唯一索引)
 mysql> alter table 'table_name' add unique('column');
 mysql> alter table schedule_item add unique(`schedule_item_name`);
     Query OK, 0 rows affected (0.02 sec)
     Records: 0  Duplicates: 0  Warnings: 0
  a) 创建唯一键约束的时候,需要保证表中对应列上的数据,不可以重复.
     数据库默认值一般都为Null,此时创建唯一索引时需要注意了,此时数据库会把空作为多个重复值.

  b) 提示错误"Specified key was too long; max key length is 767 bytes"
     由于MySQL Innodb 引擎表索引表字段长度的限制为767字节,因此对于多字节字符集的大字段(或者多字段组合索引),创建索引会出现上面的错误.
     以utf8mb4字符集,字符串类型字段为例: utf8mb4是4字节字符集.
     默认支持的索引字段最大长度为: 767字节/4字节每字符=191字符,因此在varchar(255)或char(255)类型字段上创建索引会失败.

> 添加index(普通索引)
    mysql> alter table 'table_name' add index index_name('column');

> 添加fulltext(全文索引)
    mysql> alter table 'table_name' add fullText('column')

> 删除唯一键约束
    mysql> alter table schedule_item_targeting drop index uk_siu_target;
        Query OK, 0 rows affected (0.08 sec)
        Records: 0  Duplicates: 0  Warnings: 0
     这里的index,指代unique key

```

### sql 语句

```scala
select d,count(*) from ( select DATE_FORMAT(db_update_time, '%Y-%m-%d') as d from ad_material where db_update_time >= '2020-11-01' ) a group by  d ;

日期格式 date_format(date,'%Y-%m-%d')
```
