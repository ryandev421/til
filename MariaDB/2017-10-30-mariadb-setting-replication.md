---
layout: post
title: MariaDB Replication 설정
date: 2017-10-30
excerpt: ""
tags: [gtid, mariadb, mysql, replication, extrabackup, percona-toolkit]
comments: true
category: development
---

## 1. MariaDB 설치

```shell
sudo apt-get install software-properties-common
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://ftp.kaist.ac.kr/mariadb/repo/10.2/ubuntu xenial main'
sudo apt update
sudo apt -y install mariadb-server
```

## 2. my.cnf 사전 세팅



## 3. percona-xtrabackup 2.4 설치

```shell
wget https://repo.percona.com/apt/percona-release_0.1-4.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_0.1-4.$(lsb_release -sc)_all.deb
sudo apt-get update
sudo apt-get install percona-xtrabackup-24
```

## 4. qpress 설치 (innobackupex --decompress에 사용됨)

```shell
# slave 서버에서
cd /tmp
wget http://www.quicklz.com/qpress-11-linux-x64.tar
tar xvf qpress-11-linux-x64.tar
chmod 755 qpress
chown root:root qpress
mv qpress /usr/bin/qpress
```



## 4. MariaDB Stop & 초기화

- innobackupex 실행을 위해 mysql 서비스 정지

```shell
# slave 서버에서
service mysql stop
```

- /storage/mysql 비우기

```shell
# slave 서버에서
rm -rf /storage/mysql
mkdir /storage/mysql
cd /storage/mysql
```



## 5. innobackupex로 복제

- slave 역할을 할 곳 (ip: 192.168.0.A)


```shell
cd /storage/mysql
nc -l -p 1234 | xbstream -x
```

- master역할을 할 곳

```shell
sudo innobackupex --defaults-file=/etc/mysql/my.cnf --no-lock --compress --compress-threads=8 --password=$DB_PASSWORD --stream=xbstream /storage/mysql/ | nc $slave_ip 1234

# 주의사항
$DB_USER, $DB_PASSWORD는 master서버의 /etc/mysql/debian.cnf 파일내의 user, password 값
```



## 6. 압축해제

```shell
# slave 서버에서
innobackupex --decompress --parallel=4 /storage/mysql/
innobackupex --apply-log --parallel=4 --ibbackup=xtrabackup_56 /storage/mysql/
chown -R mysql:mysql /storage/mysql/
```



## 7. Replication 적용

```shell
#slave 서버에서
echo "SET GLOBAL gtid_slave_pos = {GTID};" | mysql -uroot -p$DB_PASSWORD
echo "CHANGE MASTER TO master_host="$MASTER_HOST", master_user="replicator",master_password="$DB_PASSWORD", master_use_gtid=slave_pos" | mysql -uroot -p$DB_PASSWORD
echo "START SLAVE" | mysql -uroot -p$DB_PASSWORD
sleep 2
echo "SHOW SLAVE STATUS" | mysql -uroot -p$DB_PASSWORD

# 주의사항
$DB_USER, $DB_PASSWORD는 master서버의 /etc/mysql/debian.cnf 파일내의 user, password 값
```



## 8. 압축파일 삭제

- XtraBackup 2.2.10부터 qp 파일을 자동으로 삭제하지 않아 수동으로 삭제 필요. 아래 명령 실행하여 삭제.

```shell
sudo find /storage/mysql -type f -name "*.qp" -exec rm -rf {} \;
```

