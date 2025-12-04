# Redis (1)

[[10분 테코톡] 마크의 Redis](https://www.youtube.com/watch?v=UkYdk1KKVCA)

### Remote Dictionary Server

- key - value의 형태로 데이터를 관리
- NoSQL

### In-memory DataBase

- 데이터를 디스크가 아닌 메모리에 저장한다.
- 디스크에 저장하는 것 보다 데이터 접근 속도가 빠르다.

### Redis 자료구조

- Strings, Hashes, Lists(연결 리스트) , Sets, Sorted Set (정렬된 데이터가 필요할 때) 제공

### 사용 사례

1. Redis에 캐싱을 하게 되면 같은 API에 대해 조회를 빠르게 할 수 있기 때문에 캐시의 용도로 사용된다.
2. 다중 서버에서 세션 기반의 인증을 사용할 때 세션을 관리할 때 사용 

1. Refresh 인증 토큰, 토큰 Black list 저장
2. 유저 API Limit
3. Sns Feed, 채팅 기능 

### 알아두면 좋은 개념

1. 캐싱 전략 (캐시 읽기 쓰기 전략)
2. O(N) 명령어 주의 
    1. Single Thread 기반이기 때문에 동시에 여러개의 명령어를 처리할 수 없음. 
    2. 한 개가 오래 걸리면 다른 명령을 실행할 수 없음
3. Redis Replication  : 데이터 복제
4. Redis Persistence : 영구 저장

### 스프링부트 Redis

1. 의존성 추가
2. Config Class 설정

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory();
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());

        // Key - String
        template.setKeySerializer(new StringRedisSerializer());

        // Value - JSON 직렬화
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return template;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate() {
        return new StringRedisTemplate(redisConnectionFactory());
    }
}

```

1. 값 저장 조회

```java
@Service
@RequiredArgsConstructor
public class RedisService {

    private final RedisTemplate<String, Object> redisTemplate;

    public void saveValue(String key, Object value) {
        redisTemplate.opsForValue().set(key, value);
    }

    public Object getValue(String key) {
        return redisTemplate.opsForValue().get(key);
    }

    public void delete(String key) {
        redisTemplate.delete(key);
    }
}

```

- TTL 설정 가능
- 아래 코드와 같이 하면 Hash 자료구조도 저장 및 조회 가능

```java
redisTemplate.opsForHash().put("user:1", "name", "세영");
redisTemplate.opsForHash().put("user:1", "age", "25");

Object name = redisTemplate.opsForHash().get("user:1", "name");

```