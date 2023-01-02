# mysql8-tpcc-read-write-monitor
This project modifies mysql 8.0.28 code to monitor the read and write statistics of tpcc tables.
<br/><br/>


## Build/Installation
1. **MySQL 8.0.28 installation**: To install mysql 8.0.28, please refer to [link](https://github.com/kyongs/How-To/blob/main/MySQL/mysql_8_0_28_installation_with_source_code.md).
2. **TPC-C installation**: To install tpcc, please refer to [link](https://github.com/kyongs/How-To/blob/main/MySQL/tpcc_mysql_installation.md).
<br/>


## Modified Files
The major modifications are made in the following files. Type `ctrl+f, kyong` to search the related modifications.


- `storage/innobase/include/srv0srv.h`
- `storage/innobase/fil/fil0fil.cc`
- `storage/innobase/srv/srv0srv.cc`
- `storage/innobase/buf/buf0buf.cc`
- `storage/innobase/buf/buf0flu.cc`
- `storage/innobase/buf/buf0rea.cc`

<br/>

## How to change the buffer size
To change the buffer size in MySQL, you have to modify the `my.cnf`. Change the line where `#### CHANGE` is written. `innodb_buffer_pool_size` is the buffer size related configuration.
```
#
# The MySQL database server configuration file
#
[client]
user    = root
port    = 3306
socket  = /tmp/mysql.sock

[mysql]
prompt  = \u:\d>\_

[mysqld_safe]
socket  = /tmp/mysql.sock

[mysqld]
# Basic settings
default-storage-engine = innodb
pid-file        = /home/vldb/data-disk/test-data/mysql.pid
socket          = /tmp/mysql.sock
port            = 3306
datadir         = /home/vldb/data-disk/test-data/ #### CHANGE
log-error       = /home/vldb/log-data/mysql_error.log #### CHANGE
log_error_verbosity = 3
disable_log_bin

#
# Innodb settings
#
# Page size
innodb_page_size=4KB

# Buffer pool settings
innodb_buffer_pool_size=7G ##### CHANGE
innodb_buffer_pool_instances=8

# Transaction log settings
innodb_log_file_size=1G
innodb_log_files_in_group=2
innodb_log_buffer_size=32M

# Log group path (iblog0, iblog1)
# If you separate the log device, uncomment and correct the path
innodb_log_group_home_dir=/home/vldb/log-data #### CHANGE

# Flush settings (SSD-optimized)
# 0: every 1 seconds, 1: fsync on commits, 2: writes on commits
innodb_flush_log_at_trx_commit=0
innodb_flush_neighbors=0
innodb_flush_method=O_DIRECT
                               
```

## Process
**1. Run MySQL source code.**
  ```
  $ cd /usr/local/mysql
  $ ./bin/mysqld_safe --defaults-file=/usr/local/mysql/my.cnf
  ```

**2. Run TPC-C.**
  ```
  $ cd ~/tpcc-mysql
  $ ./tpcc_start -h 127.0.0.1 -S /tmp/mysql.sock -d tpcc -u root -p1234 -w {warehouse_num} -c {client_num} -r {rampup_time} -l {execution_time} | tee tpcc-result.txt
  ```

**3. Monitor the InnoDB Status. The result will show like below.**
  ```
  $ cd /usr/local/mysql
  $ ./bin/mysql -uroot -p
  
  
  > SHOW ENGINE INNODB STATUS;
  ```
  
<img width="300" alt="스크린샷 2022-09-15 오후 12 46 34" src="https://user-images.githubusercontent.com/84424998/210204775-7c0325d7-d880-4d40-957b-0f9481df6d1f.png">

<img width="294" alt="스크린샷 2022-09-15 오후 12 46 23" src="https://user-images.githubusercontent.com/84424998/210204794-b70bbc62-419b-4b7c-90f6-ee5c04a9c48e.png">


**4. Collect the InnoDB status when the experiment ends, and calculate checkpoint flush rate.**

  Use excel or shell script to calculate specific statistics.
  
  <br/><br/>
  ---
<span>Please refer to the link to see the detailed results.</span> [link✈️](https://carrotcakes.notion.site/TPC-C-IO-Pattern-f39bf8faf82444449b6205bb408a9671)
  
  



