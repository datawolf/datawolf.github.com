---
layout: post
title: "自动备份网站和数据库的shell脚本"
date: 2013-04-06 16:05
comments: true
categories: [mysql, shell, website]
---

## 备份网站内容
我的网站是用xampp集成环境搭建的。为了方便迁移和备份数据。使用如下脚本对我网站内容进行备份。其中，`BACKUPDIR`指定备份文件的存储位置。
每一次备份的文件都用备份时系统的时间来进行唯一的命名，获取实际使用命令`date -I`。
``` sh
#!/bin/bash

# where to store the backup file
BACKUPDIR="/home/wanglong"

DATE=`date -I`
[ ! -d $BACKUPDIR ] && mkdir -p $BACKUPDIR
cd /opt/lampp
tar  zcf $BACKUPDIR/htdocs.$DATE.tar.gz htdocs 
```

## 备份mysql数据库

备份数据库前，需要指定一下shell变量，包括mysql数据库的用户名、密码、运行数据库的主机和数据库相关命令的位置。
``` sh
MYUSER="user"
MYPASS="password"
HOST="localhost"
BACKUPDIR="/home/wanglong"
MYSQL="/opt/lampp/bin/mysql"
MYSQL_DUMP="/opt/lampp/bin/mysqldump"
```
完整shell脚本如下，首先将所有的数据库作为一个整体进行备份，然后对每一个数据库分别进行了备份。
``` sh
#!/bin/bash

MYUSER="user"
MYPASS="password"
HOST="localhost"
BACKUPDIR="/home/wanglong"
MYSQL="/opt/lampp/bin/mysql"
MYSQL_DUMP="/opt/lampp/bin/mysqldump"
DATE=`date -I`

[ ! -d $BACKUPDIR/$DATE ] && mkdir -p $BACKUPDIR/$DATE

# backup all databases in a file
$MYSQL_DUMP -u$MYUSER -p$MYPASS -h$HOST  --all-databases > $BACKUPDIR/$DATE/all-databases.sql

# backup each database in separate file
DBS=`$MYSQL -u$MYUSER -p$MYPASS -Bse "show databases"|grep -v "information_schema" | grep -v "performance_schema"|grep -v "Database"`
for db_name in $DBS
  do
    $MYSQL_DUMP  -u$MYUSER -p$MYPASS -h$HOST  $db_name  > $BACKUPDIR/$DATE/$db_name.sql
  done

cd $BACKUPDIR
tar zcf mysql.$DATE.tar.gz $DATE && rm -rf $BACKUPDIR/$DATE
```
## 自动执行脚本

计划每周自动备份一次网站和数据库，所以需要将上述脚本拷贝到目录`/etc/cron.weekly/`中。



