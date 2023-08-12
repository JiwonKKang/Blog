---
title: "SpringBoot에서 Redis를 적용해보자"
date: 2023-08-12T23:26:47+09:00
draft: false
pin: true
summary: "카카오API를 활용한 약국길찾기 서비스에 Redis를 적용해보자"
tags: [spring]
---

> # Redis란?

**메모리상에 key value store (KVS) 를 구축할수 있게 해주는 소프트웨어이다.**

KVS는 저장하고 싶은 데이터(값 : value) 에 대해, 대응하는 키를 설정해, 이것을 pair 로 보존하는 데이터베이스의 한 종류며, Redis 는 컴퓨터의 메인 메모리상에 KVS를 구축하여, 외부 프로그램으로부터 데이터의 보존및 읽기가 가능케 한다.
ANSI C로 만들어져있으며, 모든 데이터셋을 메모리내에 읽어들이기에, 엄청난 수준의 속도로 동작한다.

엔트리급 리눅스 서버로 **110,000 SET / 초, 81,000 GET / 초** 의 속도를 내게 할수있다.

또한 Redis는 커멘드의 파이프라이닝 을 지원하기에 복수의 값을 하나의 커멘드로 취득하게 설정이 가능하기에, 
클라이언트 라이브러리와의 통신속도 향상도 가능하다

> # 데이터 구조

- 문자열
- 바이너리 데이터
- 리스트
- 집합(set)
- 해쉬

이런 데이터에 대해 요소의 pop/ push, add/remove, 또는 서버사이드에 set 사이에서 합, 곱, 차이등 형에 의해 여러가지 조작을 지원하고 있다. Redis는 set형과 list형 에 대해 다른종류의 정렬기능을 지원하고 있다.

> # 데이터 영속화

Redis는 인메모리 데이터베이스라고 들었기때문에 처음에는 어플리케이션이 꺼지거나 redis서버가 꺼지면
데이터가 다 날라가는거 아닌가? 라는 생각을 했었다.

하지만 Redis에서는 DB로서도 활용할수있게 데이터 영속화를 실행한다.

Redis가 데이터 영속화를 하는 방법 스냅샷을 찍어 정기적으로 데이터베이스의 내용을 디스크에 쓴다.

따라서 Redis를 재기동하게되면 이 파일로부터 데이터를 불러와 복원시킨다. 일정횟수, 일정 간격으로 디스크에 파일을 쓴다.

파일의 쓰기 타이밍은 설정파일, config 커맨드로 설정 가능하다.

> # SpringBoot에서 Redis 적용

Spring Boot 에서 Redis 를 사용하는 방법은 `RedisRepository` 와 `RedisTemplate` 두 가지가 있습니다.

그 전에 먼저 공통 세팅이 필요합니다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

우선 build.gradle에 의존성을 추가해줍니다.

```yml
spring:
	redis: 
		host: localhost 
		port: 6379
```

- `application.yaml` 에 host 와 port 를 설정합니다.
- `localhost:6379` 는 기본값이기 때문에 만약 Redis 를 `localhost:6379` 로 띄웠다면 따로 설정하지 않아도 연결이 됩니다.
- 하지만 일반적으로 운영 서버에서는 별도의 Host 를 사용하기 때문에 값을 이렇게 별도의 값을 세팅하고 Configuration 에서 Bean 에 등록해줍니다.

```java
@Configuration  
public class RedisConfig {  
  
    @Value("${spring.redis.host}")  
    private String redisHost;  
  
    @Value("${spring.redis.port}")  
    private int redisPort;  
  
    @Bean  
    public RedisConnectionFactory redisConnectionFactory() {  
        return new LettuceConnectionFactory(redisHost, redisPort);  
    }    
    
    @Bean  
	public RedisTemplate<String, Object> redisTemplate() {  
	
	    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();  
	    redisTemplate.setConnectionFactory(redisConnectionFactory());  
	    redisTemplate.setKeySerializer(new StringRedisSerializer());  
	    redisTemplate.setHashKeySerializer(new StringRedisSerializer());  
	    redisTemplate.setHashValueSerializer(new StringRedisSerializer());  
	    return redisTemplate;  
	}
}
```

Spring Data Redis 의 Redis Repository 를 이용하면 간단하게 Domain Entity 를 Redis Hash 로 만들 수 있습니다.

다만 트랜잭션을 지원하지 않기 때문에 만약 트랜잭션을 적용하고 싶다면 `RedisTemplate` 을 사용해야 합니다.

제 프로젝트에서는 `RedisTemplate`을 사용합니다.

`RedisTemplate` 을 사용하면 특정 Entity 뿐만 아니라 여러가지 원하는 타입을 넣을 수 있습니다.

`template` 을 선언한 후 원하는 타입에 맞는 `Operations` 을 꺼내서 사용합니다.

> # RedisTemplate 예제

```java
@SpringBootTest  
public class RedisTemplateTest {  
  
    @Autowired  
    private RedisTemplate<String, String> redisTemplate;  
  
    @Test  
    void testStrings() {  
        // given  
        ValueOperations<String, String> valueOperations = redisTemplate.opsForValue();  
        String key = "stringKey";  
  
        // when  
        valueOperations.set(key, "hello");  
  
        // then  
        String value = valueOperations.get(key);  
        assertThat(value).isEqualTo("hello");  
    }  
  
  
    @Test  
    void testSet() {  
        // given  
        SetOperations<String, String> setOperations = redisTemplate.opsForSet();  
        String key = "setKey";  
  
        // when  
        setOperations.add(key, "h", "e", "l", "l", "o");  
  
        // then  
        Set<String> members = setOperations.members(key);  
        Long size = setOperations.size(key);  
  
        assertThat(members).containsOnly("h", "e", "l", "o");  
        assertThat(size).isEqualTo(4);  
    }  
  
    @Test  
    void testHash() {  
        // given  
        HashOperations<String, Object, Object> hashOperations = redisTemplate.opsForHash();  
        String key = "hashKey";  
  
        // when  
        hashOperations.put(key, "hello", "world");  
  
        // then  
        Object value = hashOperations.get(key, "hello");  
        assertThat(value).isEqualTo("world");  
  
        Map<Object, Object> entries = hashOperations.entries(key);  
        assertThat(entries.keySet()).containsExactly("hello");  
        assertThat(entries.values()).containsExactly("world");  
  
        Long size = hashOperations.size(key);  
        assertThat(size).isEqualTo(entries.size());  
    }  
}
```
- 위에서부터 차례대로 Strings, Set, Hash 자료구조에 대한 Operations 입니다.
- `redisTemplate` 을 주입받은 후에 원하는 Key, Value 타입에 맞게 Operations 을 선언해서 사용할 수 있습니다.
- 가장 흔하게 사용되는 `RedisTemplate<String, String>` 을 지원하는 `StringRedisTemplate` 타입도 따로 있습니다.

> # 적용

```java
public class PharmacyRedisTemplateService {  
  
    private static final String CACHE_KEY = "PHARMACY";  
  
    private final RedisTemplate<String, Object> redisTemplate;  
    private final ObjectMapper objectMapper;  
  
    private HashOperations<String, String, String> hashOperations;  
  
    @PostConstruct  
    public void init() {  
        this.hashOperations = redisTemplate.opsForHash();  
    }  
  
    public void save(PharmacyDto pharmacyDto) {  
  
        if (Objects.isNull(pharmacyDto) || Objects.isNull(pharmacyDto.getId())) {  
            return;  
        }  
  
        try {  
            hashOperations.put(  
                    CACHE_KEY,  
                    pharmacyDto.getId().toString(),  
                    serializePharmacyDto(pharmacyDto));  
            log.info("[PharmacyRedisTemplateService save success] id : {}", pharmacyDto.getId());  
        } catch (JsonProcessingException e) {  
            log.error("[PharmacyRedisTemplateService save error] : {}", e.getMessage());  
        }  
    }  
  
    public List<PharmacyDto> findAll() {  
        try {  
            ArrayList<PharmacyDto> list = new ArrayList<>();  
            for (String s : hashOperations.entries(CACHE_KEY).values()) {  
                PharmacyDto pharmacyDto = deserializePharmacyDto(s);  
                list.add(pharmacyDto);  
            }  
            return list;  
        } catch (Exception e) {  
            log.error("[PharmacyRedisTemplateService findAll error] : {}", e.getMessage());  
            return Collections.emptyList();  
        }  
    }  
  
    public void delete(Long id) {  
        hashOperations.delete(CACHE_KEY, String.valueOf(id));  
        log.info("[PharmacyRedisTemplateService delete] id: {}", id);  
    }  
  
    private String serializePharmacyDto(PharmacyDto pharmacyDto) throws   JsonProcessingException {  
        return objectMapper.writeValueAsString(pharmacyDto);  
    }  
  
    private PharmacyDto deserializePharmacyDto(String value) throws JsonProcessingException {  
        return objectMapper.readValue(value, PharmacyDto.class);  
    }  
}
```
---
# _Reference_

https://bcp0109.tistory.com/328