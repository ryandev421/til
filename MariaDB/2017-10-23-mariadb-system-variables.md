---
layout: post
title: MariaDB 시스템 변수
date: 2017-10-23
excerpt: ""
tags: [gtid, mariadb, mysql, replication, system, variables]
comments: true
category: development
---

## log_slave_updates
해당 옵션이 켜져있어야 Master로부터 받아온 변경사항을 자신의 binlog에 쓰게 된다. <br>
3개의 DB(A,B,C)가 있고, C의 마스터가 B, B의 마스터가 A인 경우가 있다고 가정할 때(A > B > C). <br>
B에 `log_slave_updates` 옵션이 켜져있어야 A에서 B로 복제된 데이터가 B에서 C로 복제된다.

## read_only
쓰기작업을 불가능하게 만든다. 보통 해당 DB가 slave일 경우 데이터 수정을 막기위해 설정한다.

## max_connections
connection 가능한 최대 갯수

## back_log
접속이 몰릴 경우 max_connections를 넘어갈 수 있는데, 이 때 들어오는 접속을 무조건 drop하지 않고 조금 더 대기할 수 있도록 여유롭게 지정