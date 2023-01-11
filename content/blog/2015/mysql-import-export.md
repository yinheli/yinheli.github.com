---
draft: false
date: 2015-02-10
title: "mysql 数据导出和导入"
tags: ["MySQL"]
---

导出

```bash
mysqldump -C -u root --opt --flush-logs --quick  dbname | gzip > file.gz
```

```
-C 使用压缩协议
-u 用户名
--opt Same as --add-drop-table, --add-locks, --create-options, 等, 其实默认是开启的
```

导入

```bash
gunzip <  file.tgz | mysql -u root -p "password" dbname
```

一个全量备份脚本

```bash
#!/bin/sh
PATH=$PATH:$HOME/bin:/home/mysql/app/mysql/bin
export PATH
cd `dirname $0`
db=hpos
dd=`date +%Y_%m%d_%H%M`
sql_file=db_backup_$dd.sql.gz
ig=''

# 忽略一些表
for i in table_ignore_1 table_ignore_2
do
   ig=$ig' --ignore-table='$db.$i
done

#echo $ig
mysqldump -u root --opt --flush-logs --quick $ig $db | gzip > $sql_file
```
