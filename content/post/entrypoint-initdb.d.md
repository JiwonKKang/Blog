---
title: "entrypoint-initdb.d sql파일을 못읽는 오류"
date: 2023-08-10T15:33:27+09:00
draft: false
pin: true
summary: "entrypoint-initdb.d sql 파일을 못읽는 오류"
tags: [Docker, Error]
---

# entrypoint-initdb.d sql 파일을 못읽는 오류

도커 컴포즈로 컨테이너를 올릴때 volume 옵션을 사용하면 
설정파일이나 sql파일을 컨테이너에 마운트 시킬 수 있다.

하지만 sql파일이 마운트까지는 되었으나 sql 파일을 못읽는 오류가 발생했다.

<br>

## 해결 방법

#### 도커 컨테이너 정리

```bash
$ docker system prune
```
- 춰있는 컨테이너 제거

- 컨테이너에서 사용되지 않는 네트워크 제거

- 불필요한 이미지 및 빌드 캐시 제거

#### 도커 컨테이너 띄울때 재빌드옵션 추가

```bash
docker-compose -f docker-compose-local.yml up --build
```

