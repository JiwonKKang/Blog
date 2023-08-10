---
title: "Schema-validation: missing table [name] 에러"
date: 2023-08-10T19:02:31+09:00
draft: false
pin: true 
summary: "missing table error"
tags: [spring, error, mysql]
---

# Schema-validation: missing table [name] 에러

##### <span style="color:red">문제현상</span>

분명 테이블이 있는데 없다고 한다.
ddl-auto 를 validate 옵션으로 설정해놓아서 schema validation 이 진행된다.
그런데 
```
.... Schema-validation: missing table [direction]
```
같은 에러가 발생한다.

컨테이너에 접속해 확인해보았지만 분명 direction 테이블을 존재했다.


---


#### 원인

-MySQL 기본 설정으로 테이블을 대소문자 구분하여 검사
-DB이름이 소문자인데, hibernate가 자꾸 대문자로 바꿔서 조회
-- 그 이유는 Hibernate가 MySQL 의 `@@lower_case_table_names` 의 설정을 따라가기 때문이다

### 문제해결

mysql의 설정파일(/etc/mysql/conf.d)을 수정

나의 경우 Volume 옵션을 통해 컨테이너에 마운트했기떄문에 프로젝트 폴더의 마운트로 설정한 cnf파일 수정

```
[mysqld] lower_case_table_names = 1
```
