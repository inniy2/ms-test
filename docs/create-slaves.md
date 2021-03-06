## create-slaves

This method works under the following condition

1. The master instance can be shut downed.

2. The master is also slave of another master.

3. The master and slaves share the same directory architecture.

4. The master and slaves must be same MySQL version.

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
mysql> source /MYSQL/XXXXX/env/XXXXX.sh

```

2. Export common
```
export master_hostname=
export slave_hostname=
export git_repository_name=ms202
export mysql_instance_name=XXXXX
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

6. Rmove previous data on slaves if exists
```
root> cd $working_directory
root> rm -rf ./data
root> rm -rf ./binlog
root> rm -rf ./innodb
root> rm -rf ./innodb-log
root> rm -rf ./my_$mysql_instance_name.cnf
```

7. Copy data from master
```
root> loop ( echo $slave_hostname ) 
      scp -r /MYSQL/$mysql_instance_name/data          XXXXX@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/binlog        XXXXX@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/innodb        XXXXX@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/innodb-log    XXXXX@$slave_hostname:$working_directory
      scp -r /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf    XXXXX@$slave_hostname:$working_directory
```
8. Modify master configuration if necessary

Create git repository if there is no repository

Setup git for the first time
```
root> cd /usr/local/src
root> if not exists ( ls -al git) 
      then: mkdir git && cd git
            export http_proxy=http://dev-proxy.db.XXXXX.co.jp:XXXXX ; export https_proxy=https://dev-proxy.db.XXXXX.co.jp:XXXXX
            git config --global user.name "XXXXX XXXXX | Sunny | XXXXX"
            git config --global user.email "XXXXX.bae@XXXXX.com"
            git clone https://XXXXX.bae@git.XXXXX-it.com/scm/XXXXX/$git_repository_name.git
```

Copy  my.cnf file for modification
```
root> cd /usr/local/src/git/$git_repository_name
root> git pull origin master
root> cp    /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf   ./my_$mysql_instance_name.cnf.`hostname`
root> diff  /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf   ./my_$mysql_instance_name.cnf.`hostname`
```

Modify my.cnf using vim 
```
root> cd /usr/local/src/git/$git_repository_name
root> git pull origin master
root> vim   ./my_$mysql_instance_name.cnf.`hostname`
root> diff  ./my_$mysql_instance_name.cnf.`hostname`     /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf
root> cp    ./my_$mysql_instance_name.cnf.`hostname`     /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.cnf
```

Push to master
```
root> git add . && git commit -m 'modified my.cnf'
root> git push -u origin master
```

9. Start up the master & start replication
```
mysql> tail -f $mysql_error_log_file
root > ~mysql/script/my_server.sh start $mysql_instance_name
mysql> echo " start slave " | mysql
```

10. Restore directories
```
root> mv $working_directory/data        /MYSQL/$mysql_instance_name/
root> mv $working_directory/binlog      /MYSQL/$mysql_instance_name/
root> mv $working_directory/innodb      /MYSQL/$mysql_instance_name/
root> mv $working_directory/innodb-log  /MYSQL/$mysql_instance_name/

root> chown -R mysql:mysql /MYSQL/$mysql_instance_name/data
root> chown -R mysql:mysql /MYSQL/$mysql_instance_name/binlog
root> chown -R mysql:mysql /MYSQL/$mysql_instance_name/innodb
root> chown -R mysql:mysql /MYSQL/$mysql_instance_name/innodb-log
```

11. Modify slaves configuration if necessary
```
Go to 8 and use the procedure 
```

12. Delete auto.cnf on slaves
```
mysql> if exists ( ls -al /MYSQL/$mysql_instance_name/data/auto.cnf )
       then: rm -rf /MYSQL/$mysql_instance_name/data/auto.cnf
```

13. Start up slaves
```
mysql> tail -f $mysql_error_log_file
root > ~mysql/script/my_server.sh start $mysql_instance_name
```

14. Reset replication
```
mysql> mysql -h $master_hostname -u repl -p -P3301

mysql> if master does have repl user
       then: 
       use mysql;
       set session sql_log_bin = 0 ;
       CREATE USER 'repl'@'ds-ms202%' IDENTIFIED BY '';
       GRANT REPLICATION SLAVE ON *.* TO 'repl'@'ds-ms202%';
       flush privileges;
       exit;

mysql> reset slave all;
mysql> CHANGE MASTER TO \
	MASTER_HOST='' ,\
	MASTER_USER='repl' ,\
	MASTER_PASSWORD='' ,\
        MASTER_PORT=3301 ,\
	MASTER_LOG_FILE='' ,\
	MASTER_LOG_POS=  ;
mysql> start slave;
```

100. Finish up work
```
unset http_proxys; unset http_proxy
```

101. Delete backup directories on slaves
```
mysql> if exits ( ls -al /MYSQL/$mysql_instance_name/data_`date +%Y-%m-%d`)
       then: rm -rf /MYSQL/$mysql_instance_name/data_`date +%Y-%m-%d`
mysql> if exits ( ls -al /MYSQL/$mysql_instance_name/binlog_`date +%Y-%m-%d`)
       then: rm -rf /MYSQL/$mysql_instance_name/binlog_`date +%Y-%m-%d`
mysql> if exits ( ls -al /MYSQL/$mysql_instance_name/innodb_`date +%Y-%m-%d`)
       then: rm -rf /MYSQL/$mysql_instance_name/innodb_`date +%Y-%m-%d`
mysql> if exits ( ls -al /MYSQL/$mysql_instance_name/innodb-log_`date +%Y-%m-%d`)
       then: rm -rf /MYSQL/$mysql_instance_name/innodb-log_`date +%Y-%m-%d`
mysql> if exits ( ls -al /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.`date +%Y-%m-%d`)
       then: rm -rf /MYSQL/$mysql_instance_name/var/my_$mysql_instance_name.`date +%Y-%m-%d`
```














