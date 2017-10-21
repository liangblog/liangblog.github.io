---
title: Mysql Slave Config With Docker
date: 2017-09-29 16:35:30
tags:
- Ubuntu
- Mysql
- Slave
- Docker
---

### Server environment

```
$ cat /etc/issue
  Ubuntu 14.04.5 LTS
$ docker -v
  Docker version 1.6.2, build 7c8fca2
$ go version
  go version go1.8.4 linux/amd64

mysql> select version();
+------------+
| version()  |
+------------+
| 5.7.12-log |
+------------+
```
<!--more-->


### Get mysql basic docker
```
$ docker pull mysql
```

### Mkdir the config file directory
```
$ mkdir -p ./config/mysql/master/1
$ mkdir -p ./config/mysql/slave/11
$ mkdir -p ./config/mysql/slave/12
```
The directory tree:
```
config/
└── mysql
    ├── master
    │   └── 1
    └── slave
        ├── 11
        └── 12
```

### New the config files

master 1
```
$ vim config/mysql/master/1/master.cnf
```
```
[mysqld]
server_id = 1

character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
default-storage-engine=INNODB

#Optimize omit

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

log-bin     = /var/lib/mysql/binlog
log_bin_trust_function_creators=1
binlog_format = ROW
expire_logs_days = 99
sync_binlog = 0

slow-query-log=1
slow-query-log-file=/var/log/mysql/slow-queries.log
long_query_time = 3
log-queries-not-using-indexes
binlog-row-image=full
```

slave 11
```
$ vim config/mysql/slave/11/slave.cnf
```
```
[mysqld]
server_id = 11

character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
default-storage-engine=INNODB

#Optimize omit

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

log-bin     = /var/lib/mysql/binlog
log_bin_trust_function_creators=1
binlog_format = ROW
expire_logs_days = 99
sync_binlog = 0

relay_log=slave-relay-bin
log-slave-updates=1
slave-skip-errors=all

slow-query-log=1
slow-query-log-file=/var/log/mysql/slow-queries.log
long_query_time = 3
log-queries-not-using-indexes
binlog-row-image=full
```

slave 12
```
$ vim config/mysql/slave/12/slave.cnf
```
```
[mysqld]
server_id = 12

character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
default-storage-engine=INNODB

#Optimize omit

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

log-bin     = /var/lib/mysql/binlog
log_bin_trust_function_creators=1
binlog_format = ROW
expire_logs_days = 99
sync_binlog = 0

relay_log=slave-relay-bin
log-slave-updates=1
slave-skip-errors=all

slow-query-log=1
slow-query-log-file=/var/log/mysql/slow-queries.log
long_query_time = 3
log-queries-not-using-indexes
binlog-row-image=full
```


Now there will be master-1, slave1-1,slave1-2 three mysql config files
```
config
└── mysql
    ├── master
    │   └── 1
    │       └── master.cnf
    └── slave
        ├── 11
        │   └── slave.cnf
        └── 12
            └── slave.cnf
```

### Build the mysql containers
```
$ sudo docker run -v $PWD/config/mysql/master/1:/etc/mysql/conf.d -v $PWD/data/mysql/master/1:/var/lib/mysql -v $PWD/log/mysql/master/1:/var/log/mysql -e MYSQL_ROOT_PASSWORD=mysqlpassword -d -p 4306:3306 --name=mysql-master-1 mysql
$ sudo docker run -v $PWD/config/mysql/slave/11:/etc/mysql/conf.d -v $PWD/data/mysql/slave/11:/var/lib/mysql -v $PWD/log/mysql/slave/11:/var/log/mysql -e MYSQL_ROOT_PASSWORD=mysqlpassword -d -p 4307:3306 --name=mysql-slave-11 --link mysql-master-1:master mysql
$ sudo docker run -v $PWD/config/mysql/slave/12:/etc/mysql/conf.d -v $PWD/data/mysql/slave/12:/var/lib/mysql -v $PWD/log/mysql/slave/12:/var/log/mysql -e MYSQL_ROOT_PASSWORD=mysqlpassword -d -p 4308:3306 --name=mysql-slave-12 --link mysql-master-1:master mysql
```

```
$ sudo docker ps
```
```
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                    NAMES
9716b7e177ad        mysql:latest        "docker-entrypoint.s   34 minutes ago      Up 34 minutes       0.0.0.0:4308->3306/tcp   mysql-slave-12
b731f3b7a8d0        mysql:latest        "docker-entrypoint.s   34 minutes ago      Up 34 minutes       0.0.0.0:4307->3306/tcp   mysql-slave-11
6a3cbc3f50d4        mysql:latest        "docker-entrypoint.s   34 minutes ago      Up 34 minutes       0.0.0.0:4306->3306/tcp   mysql-master-1
```

### Entry the master database
```
$ sudo docker exec -it mysql-master-1 mysql -p
```
Then check the master status
```
mysql> show master status\G;
```
```
*************************** 1. row ***************************
File: binlog.000004
Position: 743
Binlog_Do_DB:
Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```
Remember the binlog file name and position. Here the file is **binlog.000004** and **743**

### Create master user

```
mysql> CREATE USER 'repl'@'%' IDENTIFIED BY 'repl';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
```
username is **'repl'** and password is **'repl'**

### Entry the slave Mysql and config them

```
$ sudo docker exec -it mysql-slave-1 mysql -p
```

```
mysql> CHANGE MASTER TO \
    -> MASTER_HOST='master',\
    -> MASTER_PORT=3306,\
    -> MASTER_USER='repl',\
    -> MASTER_PASSWORD='repl',\
    -> MASTER_LOG_FILE='binlog.000004',\
    -> MASTER_LOG_POS=743;
Query OK, 0 rows affected, 2 warnings (0.03 sec)
```
```
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```

- **MASTER_HOST='master'**  // 'master' is the docker link alias

- **MASTER_LOG_FILE='binlog.000004'** and **MASTER_LOG_POS=743;** //come from master status

### Check Slave status
```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: master
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000004
          Read_Master_Log_Pos: 743
               Relay_Log_File: slave-relay-bin.000002
                Relay_Log_Pos: 473
        Relay_Master_Log_File: binlog.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
...
```

Ok slave mysql has connected to the master database.
You can change something in master then check the changes on slave machine.