---
layout: post
title: REPEATABLE READ
date: 2018-03-30
excerpt: "Transaction Isolation Level 중에서 innodb의 기본값인 Repeatable read에 대해 알아보자"
tags: [mariadb, mysql, isolation-level, repeatable-read, innodb]
comments: true
category: development
---

# 1. Isolation Level
여러개의 트랜잭션에서 변경되고 수행되는 쿼리가 동시에 일어났을 때, 성능과 신뢰, 일관성 사이에서 균형을 잡는 수준이다.
예를 들어서 한 트랜잭션에서 A라는 데이터를 수정하고 아직 커밋이 되지 않은 경우, 다른 트랜잭션은 A라는 데이터에 락을 걸어 접근할 수가 없게 할 수 있다(Read Committed Isolation Level인 경우).
반면 Isolation Level을 조정하여, 한 트랜잭션에서 A라는 데이터를 B라고 수정하고 아직 커밋이 되지 않은 경우, 다른 트랜잭션은 B라는 데이터에 접근할 수 있게 할 수도 있다(Read Uncommitted)
이렇게 위에 예시로 든 2개를 포함하여 총 4개의 Level이 존재하는데, 이 중에서 innoDB의 기본값인 `Repeatable Read Isolation Level`에 대해서 설명하고자 한다.

# 2. Repeatable Read Isolation Level
하나의 트랜잭션내에서 **첫 번째 SELECT 쿼리가 실행되었을 때 스냅샷을 찍어서** 타 트랜잭션의 변경사항과 상관없이 SELECT 결과를 계속해서 같은 상태로 보여주는 것이다.
예를 들어 아래와 같은 테이블이 있다고 가정해 보자.

```sql
CREATE TABLE `table_1` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `val` tinyint(3) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

그리고 아래와 같이 (1), (2) (3) 순으로 쿼리가 실행되었다고 가정해 보자
```
## A 트랜잭션
START TRANSACTION;
SELECT * FROM table_1 WHERE id = 1; # (1)  => 결과가 (id =1, val = 5) 나왔다고 가정.
SELECT * FROM table_1 WHERE id = 1; # (3)  => B 트랜잭션의 (2)에서 update가 이루어졌지만
                                                repeatable isolation level에 의해 
                                                (1)의 결과와 동일하게 나옴
COMMIT;


# B 트랜잭션
START TRANSACTION;
UPDATE table_1 SET val = 2018 WHERE id = 1; # (2) => (1) 에서 select 했던 row의 데이터를 변경함
COMMIT;
```

여기에서 주의해야 할 것이 있는데, 바로 **첫 SELECT 쿼리가 실행되었을 때 스냅샷을 찍는다**는 것이다. 아래 두가지 예시를 비교해보면 이해가 될 것이다.

### 1) 첫번째 예시

```
## A 트랜잭션
START TRANSACTION;
SELECT * FROM table_1 WHERE id = 500; # (1) => 해당 row가 비어서 empty set이 나왔다고 가정
SELECT * FROM table_1 WHERE id = 500; # (3) => (1)과 동일하게 empty set이 나옴.
                                              (1)의 첫 SELECT 쿼리에서 스냅샷을 찍고 스냅샷의
                                               데이터를 읽기 때문에 계속해서 동일하게 (1)번의 결과가 나옴
COMMIT;


## B 트랜잭션
START TRANSACTION;
INSERT INTO table_1 (`id`, `val`) VALUES(500, 999);  # (2) => id=500에 데이터를 새로 추가해 줌
COMMIT;
```

### 2) 두번째 예시
위의 첫번째 예시와 동일하게 id=500인 row가 없다고 가정.
```
## A 트랜잭션
START TRANSACTION;
INSERT INTO table_1 (`id`, `val`) VALUES(200, 300); # (1) => id=500과 상관없는 데이터를 삽입.
SELECT * FROM table_1 WHERE id = 500;   # (3) => (id = 500, val = 999) 데이터가 나옴
                                                 (3)번 이전에 스냅샷을 찍은적이 없기에
                                                 B 트랜잭션의 (2)에서 새로 추가된 값을 읽음
COMMIT;


## B 트랜잭션
START TRANSACTION;
INSERT INTO table_1 (`id`, `val`) VALUES(500, 999);  # (2) => id=500에 데이터를 새로 추가해 줌
COMMIT;
```