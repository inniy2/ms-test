## create-slaves

This method works under the following condition

1. The master instance can be shut downed.

2. The master and slaves share the same directory architecture.

3. The master and slaves must be same MySQL version.

## Procedure
Preparation for slaves
1. Source mysql on common
```
root > su mysql
mysql> bash
mysql> source /MYSQL/cloud101/env/cloud101.sh

```

2. Export common
```
export master_hostname=
export slave_hostname=
export mysql_instance_name=cloud101
export mysql_start_script=/usr/local/mysql/script/my_server.sh
export mysql_error_log_file=/MYSQL/cloud101/log/cloud101.err
export working_dir=/MYSQL/cloud101/work/work_bae
```

3. Create working directory common
```
root > if not exists ( ls -al $working_directory )
       then:   mkdir -pvm 777 $working_directory
```

4. Stop instances on slaves
```
mysql> tail -f $mysql_error_log_file
mysql> echo " set global innodb_fast_shutdown = 0  " | mysql
mysql> echo " stop slave " | mysql
mysql> echo " show master status\G | mysql > $working_directory/master_status_`date +%Y-%m-%d`.log
mysql> echo " show slave status\G  | mysql > $working_directory/slave_status_`date +%Y-%m-%d`.log
root > ~mysql/script/my_server.sh stop cloud101
```

5. Backup directories on slaves
```
mysql> cd /MYSQL/$mysql_instance_name
mysql> mv data data_`date +%Y-%m-%d`
mysql> mv binlog binlog_`date +%Y-%m-%d`
mysql> mv innodb innodb_`date +%Y-%m-%d`
mysql> mv innodb-log innodb-log_`date +%Y-%m-%d`
mysql> mv var/my_cloud101.cnf my_cloud101.`date +%Y-%m-%d`
```



