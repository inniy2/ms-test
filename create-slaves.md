## create-slaves

This method works under the following condition

1. The master instance can be shut downed.

2. The master and slaves share the same directory architecture

# Procedure
Preparation for slaves
1. Source mysql
```
root > su mysql
mysql> bash
mysql> source /MYSQL/cloud101/env/cloud101.sh

```

2. Stop instances on slaves
```
mysql> export mysql_error_log_file=/MYSQL/cloud101/log/cloud101.err
mysql> export working_dir=/MYSQL/cloud101/work/work_bae
if not exists ( ls -al $working_directory )
    mkdir -pvm 777 $working_directory
mysql> tail -f $mysql_error_log_file
mysql> echo " set global innodb_fast_shutdown = 0  " | mysql
mysql> echo " stop slave " | mysql
mysql> echo " show master status\G | mysql > $working_directory/master_status_`date +%Y-%m-%d`.log
mysql> echo " show slave status\G  | mysql > $working_directory/slave_status_`date +%Y-%m-%d`.log
root > ~mysql/script/my_server.sh stop cloud101
```


