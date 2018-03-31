---
layout: post
title: sshguard에 의해 접속 실패하는 경우 해결방법
date: 2018-04-01
excerpt: ""
tags: [ssh, sshguard, password-block]
comments: true
category: development
---

ssh로 접속시에 실수에 의해 비밀번호를 자주 틀리게 되면 sshguard에 의한 해당 ip가 차단되는 경우가 있다.
이럴 경우 다른 서버를 통해 접속하고자 하는 서버에 접속 후 아래 명령어들을 통해 해결할 수 있다.

```shell
$ sudo iptables -L sshguard --line-numbers --numeric
Chain sshguard (1 references)
num  target     prot opt source               destination         
1    DROP       all  --  192.168.0.123         0.0.0.0/0           

$ sudo iptables -D sshguard {num}
```