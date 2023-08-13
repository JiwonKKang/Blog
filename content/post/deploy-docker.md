---
title: "도커 컨테이너로 배포중 삽질"
date: 2023-08-13T18:47:40+09:00
draft: false
pin: false 
summary: "도커 컨테이너로 서버 배포중에 에러들"
tags: [docker, error]
---

> # 변경사항이 컨테이너에 반영안된 오류

정말 바보같은 오류였다. 도커 컨테이너를 올릴때 build된 .jar파일을 이용해서 올리는데
계속 코드만 변경하고 컨테이너를 올리면서 변경한게 왜 반영안되지 이러고있었다.

```bash
./gradlew clean build
```

로 재 빌드하고나니 변경사항이 잘 적용되어 컨테이너가 올라갔다

> # gradle 빌드시 환경변수 적용안되는 오류

테스트코드 빌드 시 카카오 API KEY 환경변수가 필요해서 

```bash
./gradlew clean build -PKAKAO_REST_API_KEY={KEY}
```

이런식으로 실행했지만 환경변수 적용이 안되었다.

이유는 모르겠지만

```bash
export KAKAO_REST_API_KEY={KEY}
```

로 환경변수를 터미널에 적용하니 정상적으로 빌드가 되었다.

위의 명령어로 왜 안됐는지는 모르겠다.


> 추가

yml파일은 들여쓰기가 굉장히 중요하다..


