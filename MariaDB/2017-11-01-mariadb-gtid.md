---
layout: post
title: MariaDB GTID(Global Transaction ID)
date: 2017-10-30
excerpt: "GTID가 나온 배경과 관련 시스템 변수 요약"
tags: [mariadb, mysql, replication, gtid]
comments: true
category: development
---

## 1. GTID가 나온 배경
Master와 여러개의 Slave 구성에서 Master가 다운되었을 때, Slave 군 중 복제 지연이 가장 적은 Slave를 새로운 마스터로 승격 시키게 된다.
다른 Slave들은 새로운 Master에게 CHANGE MASTER를 실행하여 Master를 전환해야하는데, 이때 기존의 전동적인 방식에는 문제가 발생한다.
각 Slave마다 이전 Master의 binlog 이름과 포지션에서 어디까지 복제했는지는 알고 있지만, 새로운 Master에서의 파일 이름, 포지션이 어디인지는 모르기 때문이다.
이를 해결하기 위해서 GTID라는 개념이 나오게 된 것이다.

## 2. GTID의 장점

### 2-1. Master서버 변경이 쉽다.
Slave 는 GTID를 이용하여 기존 Master DB에서 마지막으로 적용된 이벤트를 알고 있기 때문에 새로운 Master DB에서 어디서 부터 재개를 해야하는지 쉽게 알 수 있다. <br>
(모든 DB가 GTID를 알고 있고 이 값은 유니크하기 때문에 가능)

### 2-2. Slave의 상태가 크래시에도 안전하게 기록된다.

#### 2-2-a. 기존의 방법
slave의 상태는 relay-log.info에 파일로 자장이 되었고, 이것은 실제 데이터의 변화와 독립적으로 업데이트가 되어 CRASH가 잘생할 경우 쉽게 sync 맞추기가 어려웠다.

#### 2-2-b. GTID를 이용한 방법

- 복제된 위치를 같은 트랜젝션 안에서 mysql.gtid_slave_pos 시스템 테이블에 기록한다.
- update가 이루어질 때 하나의 트랜젝션안에서 그 값을 업데이트하기 때문에 CRASH-SAFE한 상태를 유지하게 된다.
- CRASH가 발생한 경우, CRASH 이후 재시작하면 기록된 복제위치가 실제 복제위치와 일치하는지 확인한다.

## 3. GTID의 구성 및 특징

- { DOMAIN ID } - { SERVER ID } - { SEQUENCE NUMBER } 로 구성 (예: 0-141-2123 은 domain id = 0, server id = 141, sequence number = 2123 이다)
- SEQUENCE ID는 binlog에 트랜젝션이 기록될때마다 증가하는 숫자
- 트랜잭션마다 부여되는 아이디(하나의 트랜젝션이 여러개의 업데이트가 있는 경우도 있으므로, 1개의 GTID에 매칭되는 업데이트도 여러개 일 수 있다.)
- 전역적으로 유니크한 값

## 4. GTID에 대한 시스템 변수

#### gtid_slave_pos

- slave 서버에서 마지막으로 복제된 이벤트 그룹 GTID
- 현재 복제 위치를 변경하기 위해 사용자가 변경 가능

#### gtid_binlog_state

- 한번이라도 binlog에 남겨졌다면 표시
- { DOMAIN ID}-{SERVER ID} 조합으로 마지막으로 기록된 GTID 표시

#### gtid_binlog_pos

- 각 도메인에 대해 binlog에 쓰인 마지막 GTID

#### gtid_current_pos

- 각 도메인별로 마지막으로 변경된 GTID
- 각 변경은 서버 자신에게 발생한 이벤트 일수도 있고, 자신의 마스터 서버로부터 복제된 이벤트일 수도 있다.

#### gtid_slave_pos VS gtid_current_pos

- gtid_slave_pos는 마스터에서 전송된 트랜잭션 중 마지막으로 실행된 GTID.
- 반면, gtid_current_pos는 마스터에서 전송된 트랜잭션 뿐만 아니라 자기자신에서 직접 실행된 트랜잭션을 포함하여 마지막으로 처리한 GTID.
- 자기 자신에서 write가 없다면, gtid_slave_pos과 gtid_current_pos는 항상 같다
- gtid_current_pos는 자신이 Master이었다가 slave로 변경되었을 경우 주로 사용한다 (Slave인 적이 없기에 gtid_slave_pos에 값이 없을 것이기 때문

**예시)** 146(Master) -> 143(Master이자 Slave) -> 144(Slave)로 구성되었을 때, 143서버의 GTID 변경 상태
<img src="/assets/img/posts/gtid-pos.jpg" style="width: 100%"/>

- (1): 계속해서 146서버로부터 복제 하다가 143에서 write가 이루어지면, slave_pos는 변경되지 않으나, current_pos은 변경된다. (gtid의 서버id가 변경됨에 주목)
- (2): (1)번 이후에 계속해서 143에서 write가 이루어지면서 gtid의 sequence_number가 증가 되었다. 하지만 146을 통해 복제가 이루어지면, 146은 143의 sequence_number를 모르기 때문에 자신이 사용했던 sequence_number를 이어서 사용한다
- (3): 바로 직전의 gtid_binlog_pos에서 sequence_number가 182 여서 (3)의 gtid_binlog_pos의 sequence_number가 183으로 예상했지만, 이미 위에 0-143-183이 있기에 0-143-184가 되었다.
- (4): slave의 gtid_binlog_pos의 sequence number는 항상 master의 sequence number + 1로 시작됨을 알 수 있다.