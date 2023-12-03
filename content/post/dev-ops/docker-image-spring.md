---
title: "[Docker] Dockerfile - SpringBoot "
date: 2023-08-18T20:53:42+09:00
draft: false
pin: false 
summary: "SpringBoot에서 Dockerfile로 도커 이미지를 빌드해보자"
tags: [docker, spring]
---

> # A Better Dockerfile

Spring Boot fat JAR은 JAR 자체가 패키징되는 방식 때문에 자연스럽게 “layers"를 갖습니다.   
먼저 압축을 풀면 이미 외부 종속성과 내부 종속성이 구분되어 있습니다.  
Docker 빌드의 한 단계에서 이 작업을 수행하려면 먼저 JAR의 압축을 풀어야 합니다.   
다음 명령은 Spring Boot fat JAR의 압축을 풉니다.  

압축해제

```bash
mkdir build/dependency 
cd build/dependency 
jar -xf ../libs/sbb-{버전넘버}.jar
```

**Dockerfile**

```dockerfile
FROM eclipse-temurin:17-jdk-alpine
VOLUME /tmp
ARG DEPENDENCY=build/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","com.mygroup.sbb.SbbApplication"]
```

도커 이미지 빌드

```bash
docker build -t sbb .
```

도커 컨테이너 실행

```bash
docker run -p 8080:8080  sbb
```

**이제 세 개의 계층이 있으며 모든 애플리케이션 리소스는 이후 두 계층에 있습니다. 
응용 프로그램 종속성이 변경되지 않으면 첫 번째 계층(BOOT-INF/lib에서)을 변경할 필요가 없으므로 
기본 계층이 이미 캐시되어 있는 한 빌드가 더 빠르고 런타임 시 컨테이너 시작도 더 빠릅니다. **

<aside> 💡 **하드 코딩된 기본 애플리케이션 클래스인 com.mygroup.sbb.SbbApplication을 사용했습니다. 
이는 귀하의 응용 프로그램에 따라 다를 수 있습니다. 원하는 경우 다른 ARG로 매개변수화할 수 있습니다. Spring Boot Fat JarLauncher를 이미지에 복사하여 애플리케이션을 실행하는 데 사용할 수도 있습니다.
작동하고 기본 클래스를 지정할 필요가 없지만 시작 시 속도가 약간 느려집니다.**

</aside>

> ## Spring Boot Layer Index

**Spring Boot 2.3.0부터 Spring Boot Maven 또는 Gradle 플러그인으로 빌드된 JAR 파일에는 JAR 파일에 레이어 정보가 포함됩니다. 
이 계층 정보는 응용 프로그램 빌드 간에 변경될 가능성에 따라 응용 프로그램의 일부를 구분합니다. 이를 사용하여 Docker 이미지 레이어를 더욱 효율적으로 만들 수 있습니다.**

**레이어 정보는 JAR 콘텐츠를 각 레이어의 디렉터리로 추출하는 데 사용할 수 있습니다. **

```bash
mkdir build/extracted
```

```bash
java -Djarmode=layertools -jar build/libs/sbb-{버전넘버}.jar extract --destination build/extracted
```

`Dockerfile`

```bash
FROM eclipse-temurin:17-jdk-alpine
VOLUME /tmp
ARG EXTRACTED=build/extracted
COPY ${EXTRACTED}/dependencies/ ./
COPY ${EXTRACTED}/spring-boot-loader/ ./
COPY ${EXTRACTED}/snapshot-dependencies/ ./
COPY ${EXTRACTED}/application/ ./
ENTRYPOINT ["java","org.springframework.boot.loader.JarLauncher"]
```

도커 이미지 빌드

```bash
docker build -t sbb .
```

도커 컨테이너 실행

```bash
docker run -p 8080:8080  sbb
```

<br>

~~~
 💡 Spring Boot Fat `JarLauncher`는 JAR에서 이미지로 추출되므로 기본 애플리케이션 클래스를 하드 코딩하지 않고 애플리케이션을 시작하는 데 사용할 수 있습니다.
~~~

<br>

> ## Multi-Stage Build - DEPENDECY

위에서 만든 Dockerfile은 fat JAR이 이미 command line에서 빌드되었다고 가정했습니다. 
multi-stage 빌드를 사용하고 한 이미지에서 다른 이미지로 결과를 복사하여 
docker에서 해당 단계를 수행할 수도 있습니다. 

다음 예제에서는 Gradle을 사용하여 이를 수행합니다.

```docker
FROM eclipse-temurin:17-jdk-alpine as build
WORKDIR /workspace/app

COPY gradlew .
COPY .gradle .gradle
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .
COPY src src

RUN ./gradlew build -x test
RUN mkdir build/dependency && (cd build/dependency; jar -xf ../libs/sbb-{버전정보}.jar)

FROM eclipse-temurin:17-jdk-alpine
VOLUME /tmp
ARG DEPENDENCY=/workspace/app/build/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","com.mygroup.sbb.SbbApplication"]
```

도커 이미지 빌드

```bash
docker build -t sbb .
```

도커 컨테이너 실행

```bash
docker run -p 8080:8080  sbb
```

<br>

> ## Multi-Stage Build - EXTRACT

이번엔 jar파일의 extract를 사용하여 레이어를 추출하고 그 레이어들을 이용해서 
Docker image를 Multi-Stage build로 만들어보겠습니다.

```dockerfile
FROM eclipse-temurin:17-jdk-alpine as build  
WORKDIR /workspace/app  
  
COPY gradlew .               
COPY .gradle .gradle  
COPY gradle gradle  
COPY build.gradle .  
COPY settings.gradle .docker  
COPY src src  
  
RUN ./gradlew build -x test  
RUN mkdir build/extracted && (java -Djarmode=layertools -jar build/libs/sbb-0.0.8.jar extract --destination build/extracted)  
  
FROM eclipse-temurin:17-jdk-alpine  
VOLUME /tmp  
ARG EXTRACTED=/workspace/app/build/extracted  
COPY --from=build ${EXTRACTED}/dependencies/ ./  
COPY --from=build ${EXTRACTED}/spring-boot-loader/ ./  
COPY --from=build ${EXTRACTED}/snapshot-dependencies/ ./  
COPY --from=build ${EXTRACTED}/application/ ./  
ENTRYPOINT ["java","org.springframework.boot.loader.JarLauncher"]
```

위 코드를 간략히 설명하자면

**스테이지 1**
1. 베이스이미지 eclipse-temurin:17-jdk-alpine로 설정
2. 디렉토리를 /workspace/app 이동(cd와는 레이어가 생기지않는다는 차이점이있습니다)
3. 빌드에 필요한 파일들 모두 컨테이너에 COPY
4. 테스트는 제외하고 빌드
5. build/extracted 디렉토리를 만들고 그곳에 레이어 추출

**스테이지 2**
1. 베이스이미지 eclipse-temurin:17-jdk-alpine로 설정
2. 컨테이너의 /tmp를 볼륨으로 설정
3. 매개변수 경로 설정 EXTRACTED=/workspace/app/build/extracted  
4. 스테이지1에서 만들어진 이미지에서 레이어들 COPY
5. 엔트리포인트 실행

이렇게하면 빌드에만 필요한 파일(스테이지1)들을 도커이미지(스테이지2)에 넣지않기때문에
도커 이미지를 경량화 시킬수있고 도커이미지가 경량화되면 빌드 및 배포가 빨라질 수 있습니다.

