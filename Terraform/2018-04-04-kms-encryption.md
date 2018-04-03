---
layout: post
title: Terraform에서 KMS Encryption 사용법
date: 2018-04-04
excerpt: "terraform에서 RDS의 master password를 KMS통해 관리하자"
tags: [terraform, aws, kms]
comments: true
category: development
---

terraform을 이용해서 인프라를 관리하다가 보면 노출되면 안되는 값을 코드로 작성해야 될 상황이 생긴다.
예를 들어, terraform으로 RDS를 생성할 때 마스터의 비밀번호가 한 예이다.
terraform을 아래와 같이 작성해 버리면 해당 코드나 관련 저장소를 볼 수 있는 모두가 DB의 마스터 비밀번호를 알게되서 보안상 취약하게 된다

```
resource "aws_db_instance" "db-sample" {
  ...
  password = "master-password123!"
  ...
}
```

그래서 이를 해결하기 위해 AWS에서 제공하는 KMS를 통해 비밀번호를 암호화하는 방법을 공유하고자 한다.

### 1. KMS Key 생성
KMS 암호화를 하는데 사용될 key를 생성해야 한다. AWS [공식문서](https://docs.aws.amazon.com/ko_kr/kms/latest/developerguide/create-keys.html)를 따라하면 쉽게 만들수 있다.

### 2. 비밀번호를 암호화
터미널창을 열고 아래 비밀번호를 별도 파일로 만들어둔다.

```shell
$ echo -n 'master-password123!' > plaintext-password
```

그리고 아래 명령어를 실행한다. 이때 `$key_id`는 1번에서 KMS key를 생성하고 나서 생성된 key의 id를 넣어줘야 한다.
혹시 터미널에 AWS CLI 구성이 안되어 있으면 [여기](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-getting-started.html)를 참고하기 바란다.
```shell
$ aws kms encrypt \
--key-id $key_id \
--plaintext fileb://plaintext-password \
--output text --query CiphertextBlob
```

실행하고 나면 아래와 같이 암호화된 문구가 출력이 된다.
```shell
$ aws kms encrypt \
--key-id $key_id \
--plaintext fileb://plaintext-password \
--output text --query CiphertextBlob

AQICAHi1tzJHZQT30hQmf3yueEfl/t9tZ2L+7jqr6IcHddQEV1guYe/AlotboEYBuPAAAAajBoBgkqhkiG9w0BBwagAzPleEZAgEAMFQGCSqGSIb3DwEQQMYzETfPvyHhzpT8tvGAntNu/P2fexBnxRTa2L6T4w/d3z/5mxeTGk/aM=
```

이것을 terraform에에 아래와 같이 추가하여 세팅해주면 끝난다.
```
data "aws_kms_secret" "db" {
  secret {
    name    = "master_password"
    payload = "AQICAHi1tzJHZQT30hQmf3yueEfl/t9tZ2L+7jqr6IcHddQEV1guYe/AlotboEYBuPAAAAajBoBgkqhkiG9w0BBwagAzPleEZAgEAMFQGCSqGSIb3DwEQQMYzETfPvyHhzpT8tvGAntNu/P2fexBnxRTa2L6T4w/d3z/5mxeTGk/aM="
  }
}

resource "aws_db_instance" "db-sample" {
  ...
  password = "${data.aws_kms_secret.db.master_password}"
  ...
}
```