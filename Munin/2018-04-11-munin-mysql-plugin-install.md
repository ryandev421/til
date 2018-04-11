---
layout: post
title: Munin에서 mysql plugin 세팅방법
date: 2018-04-11
excerpt: ""
tags: [mariadb, mysql, munin, monitoring]
comments: true
category: development
---

### 1. "/etc/munin/plugin-conf.d/munin-node" 내용 확인

```
...
[mysql*]
user root
env.mysqlopts --defaults-file=/etc/mysql/debian.cnf
#env.mysqluser debian-sys-maint
env.mysqluser root
env.mysqlconnection DBI:mysql:mysql;mysql_read_default_file=/etc/mysql/debian.cnf
...
```
해당 내용을 통해 `/etc/mysql/debian.cnf`을 이용해 접속한다는 것을 확인 가능하다.

### 2. "/etc/mysql/debian.cnf" 에서 접속정보(user, password) 변경

```
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
#user     = {$sys_maint_user}
#password = {$sys_maint_password}
user     = {$user}
password = {$password}
socket   = /var/run/mysqld/mysqld.sock
 
[mysql_upgrade]
host     = localhost
#user     = {$sys_maint_user}
#password = {$sys_maint_password}
socket   = /var/run/mysqld/mysqld.sock
basedir  = /usr
```

### 3. munin 플러그인 설정 명령어 확인

```
# munin-node-configure --suggest  // mysql used가 no임을 확인
Plugin                     | Used | Suggestions
------                     | ---- | -----------
.....
mysql_                     | no   | yes (+bin_relay_log +binlog_groupcommit +commands +connections +files +handler_read +handler_tmp +handler_transaction +handler_write +icp +innodb_bpool +innodb_bpool_act +innodb_history_list_length +innodb_insert_buf +innodb_io +innodb_io_pend +innodb_log +innodb_queries +innodb_read_views +innodb_rows +innodb_semaphores +innodb_srv_master_thread +innodb_tnx +max_mem +mrr +myisam_indexes +network_traffic +performance +qcache +qcache_mem +replication +rows +select_types +slow +sorts +table_definitions +table_locks +tables +tmp_tables)
 
 
# munin-node-configure --shell // 설정 명령어를 찾고 그대로 실행
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_bin_relay_log'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_binlog_groupcommit'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_commands'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_connections'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_files'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_handler_read'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_handler_tmp'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_handler_transaction'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_handler_write'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_icp'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_bpool'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_bpool_act'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_history_list_length'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_insert_buf'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_io'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_io_pend'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_log'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_queries'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_read_views'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_rows'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_semaphores'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_srv_master_thread'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_innodb_tnx'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_max_mem'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_mrr'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_myisam_indexes'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_network_traffic'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_performance'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_qcache'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_qcache_mem'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_replication'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_rows'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_select_types'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_slow'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_sorts'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_table_definitions'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_table_locks'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_tables'
ln -s '/usr/share/munin/plugins/mysql_' '/etc/munin/plugins/mysql_tmp_tables'
```

### 4. munin-node 재시작

```
# sudo service munin-node restart
 
// 재시작 후 munin 그래프가 생성이 안될시에 아예 stop 후 start 시도
# sudo service munin-node stop
# sleep 5
# sudo service munin-node start
```

**참고**
- [http://munin-monitoring.org/wiki/munin-node-configure](http://munin-monitoring.org/wiki/munin-node-configure)