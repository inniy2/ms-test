## create-slaves

This method works under the following condition

1. The master instance can be shut downed.

2. The master and slaves share the same directory architecture.

3. The master and slaves must be same MySQL version.

## Procedure status
This procedure has been used for the following operation.
```
Procedure construction phase
```

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
export mysql_error_log_file=/MYSQL/$mysql_instance_name/log/$mysql_instance_name.err
export working_directory=/MYSQL/$mysql_instance_name/work/work_bae
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
mysql> echo " show master status\G " | mysql > $working_directory/master_status_`date +%Y-%m-%d`.log
mysql> echo " show slave status\G  " | mysql > $working_directory/slave_status_`date +%Y-%m-%d`.log
root > ~mysql/script/my_server.sh stop $mysql_instance_name
```

5. Backup directories on slaves
```
mysql> cd /MYSQL/$mysql_instance_name
mysql> mv data data_`date +%Y-%m-%d`
mysql> mv binlog binlog_`date +%Y-%m-%d`
mysql> mv innodb innodb_`date +%Y-%m-%d`
mysql> mv innodb-log innodb-log_`date +%Y-%m-%d`
mysql> mv var/my_$mysql_instance_name.cnf var/my_$mysql_instance_name.`date +%Y-%m-%d`
```

5. Stop master 
```
mysql> tail -f $mysql_error_log_file
mysql> echo " set global innodb_fast_shutdown = 0  " | mysql
mysql> echo " stop slave " | mysql
mysql> echo " show master status\G " | mysql > $working_directory/master_status_`date +%Y-%m-%d`.log
mysql> echo " show slave status\G  " | mysql > $working_directory/slave_status_`date +%Y-%m-%d`.log
root > ~mysql/script/my_server.sh stop $mysql_instance_name
```

6. Copy data from master
```
root> loop ( ls -al $slave_hostname ) 
      scp -r /MYSQL/$mysql_instance_name/data          baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/binlog        baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/innodb        baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/innodb-log    baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf    baesangsun01@$slave_hostname:$working_directory
```












