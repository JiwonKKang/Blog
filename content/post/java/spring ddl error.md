---
title: "[ Spring ] - table doesn't exist error"
date: 2023-12-17T06:25:52+09:00
draft: false
pin: false
summary: table doesn't exist error 해결 방법
tags:
  - error
  - spring
---

> # table doesn't exist error 해결 방법

먼저 스프링에서 데이터베이스를 연결해 ```ddl-auto```를 create로 설정한뒤에 스프링 부트를 시작하니

```
Caused by: java.sql.SQLSyntaxErrorException: Table 'market.trade' doesn't exist
...
```

위와같은 오류가 떴다.

위 오류의 원인에는 여러 가지원인이 있다.

1. 엔티티의 컬럼명을 DB의 예약어로 설정한 경우
2. ddl-auto를 create로 설정한 경우

크게 위 2가지 원인정도가있는데

첫번째로 DB의 예약어를 컬럼으로 설정할 경우 하이버네이트가 테이블을 생성하지 못해 위와같은 오류가 뜰수있다.

두번째로 ddl-auto를 create로 설정하게 되면 세션 팩토리가 시작될때 drop을 실행하고 다시 테이블을 만드는데 여기서 drop을 할때 오류가 발생하게 된다. 왜냐하면 drop을 할때 

```sql
alter table trade 
drop foreign key FKohj82lqwigajbrwqijj9eun8v
```

이런 쿼리를 실행하게되는데 이때 테이블이 존재하지 않아서 발생하는 오류이다

> ## 해결방법

```
spring:
	jpa:  
	  hibernate:  
	    ddl-auto: update
```

이렇게 ddl-auto를 변경된 스키마만 적용하는 update로 설정해주면 해결할수있다.