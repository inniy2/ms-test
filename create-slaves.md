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
export git_repository_name=ms202
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
mysql> echo " set global innodb_fast_shutdown = 0  " | mysql
mysql> echo " stop slave " | mysql
mysql> echo " show master status\G " | mysql > $working_directory/master_status_`date +%Y-%m-%d`.log
mysql> echo " show slave status\G  " | mysql > $working_directory/slave_status_`date +%Y-%m-%d`.log
mysql> tail -f $mysql_error_log_file
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
mysql> echo " set global innodb_fast_shutdown = 0  " | mysql
mysql> echo " stop slave " | mysql
mysql> echo " show master status\G " | mysql > $working_directory/master_status_`date +%Y-%m-%d`.log
mysql> echo " show slave status\G  " | mysql > $working_directory/slave_status_`date +%Y-%m-%d`.log
mysql> tail -f $mysql_error_log_file
root > ~mysql/script/my_server.sh stop $mysql_instance_name
```

6. Copy data from master
```
root> loop ( echo $slave_hostname ) 
      scp -r /MYSQL/$mysql_instance_name/data          baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/binlog        baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/innodb        baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/innodb-log    baesangsun01@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf    baesangsun01@$slave_hostname:$working_directory
```
7. Modify master configuration if necessary

Create git repository if there is no repository

Setup git for the first time
```
root> cd /usr/local/usr
root> if not exists ( ls -al $git_repository_name ) 
      then: mkdir $git_repository_name
            git clone https://sangsun.bae@git.rakuten-it.com/scm/ops-dba/$git_repository_name.git
```

Copy  my.cnf file for modification
```
root> git pull origin master
root> cp    /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf   ./my_$mysql_instance_name.cnf.`hostname`
root> diff  /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf   ./my_$mysql_instance_name.cnf.`hostname`
```

Modify my.cnf using vim 
```
root> vim   ./my_$mysql_instance_name.cnf.`hostname`
root> diff  ./my_$mysql_instance_name.cnf.`hostname`     /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf
root> cp    ./my_$mysql_instance_name.cnf.`hostname`     /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf
```

Push to master
```
root> git add . && git commit -m 'modified my.cnf'
root> git push -u origin master
```

8. Start up the master & start replication
```
mysql> tail -f $mysql_error_log_file
root > ~mysql/script/my_server.sh start $mysql_instance_name
mysql> echo " start slave " | mysql
```

9. Restore directories
```
root> mv $working_directory/data        /MYSQL/$mysql_instance_name/
root> mv $working_directory/binlog      /MYSQL/$mysql_instance_name/
root> mv $working_directory/innodb      /MYSQL/$mysql_instance_name/
root> mv $working_directory/innodb-log  /MYSQL/$mysql_instance_name/

root> chown -R mysql:mysql $/MYSQL/$mysql_instance_name/data
root> chown -R mysql:mysql $/MYSQL/$mysql_instance_name/binlog
root> chown -R mysql:mysql $/MYSQL/$mysql_instance_name/innodb
root> chown -R mysql:mysql $/MYSQL/$mysql_instance_name/innodb-log
```















