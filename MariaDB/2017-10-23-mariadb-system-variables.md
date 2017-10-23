---
layout: post
title: MariaDB 시스템 변수
date: 2017-10-23
excerpt: ""
tags: [gtid, mariadb, mysql, replication, system, variables]
comments: true
category: development
---

## 1. log_slave_updates
해당 옵션이 켜져있어야 Master로부터 받아온 변경사항을 자신의 binlog에 쓰게 된다. <br>
3개의 DB(A,B,C)가 있고, C의 마스터가 B, B의 마스터가 A인 경우가 있다고 가정할 때(A > B > C). <br>
B에 `log_slave_updates` 옵션이 켜져있어야 A에서 B로 복제된 데이터가 B에서 C로 복제된다.