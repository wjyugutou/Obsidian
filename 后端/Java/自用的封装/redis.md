

```java
@Component
public class RedisJsonClient {

    @Autowired
    private StringRedisTemplate redisTemplate;

    private final ObjectMapper objectMapper = new ObjectMapper();

    // 存对象（自动转JSON）
    public <T> void set(String key, T value) {
        try {
            String json = objectMapper.writeValueAsString(value);
            redisTemplate.opsForValue().set(key, json);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("JSON序列化异常", e);
        }
    }

    // 取对象（自动转回对象）
    public <T> T get(String key, Class<T> clazz) {
        String json = redisTemplate.opsForValue().get(key);
        if (json == null) {
            return null;
        }
        try {
            return objectMapper.readValue(json, clazz);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("JSON反序列化异常", e);
        }
    }

    // 对于泛型集合（List<User>），提供带 TypeReference 的重载方法
    public <T> T get(String key, TypeReference<T> typeRef) {
        // ... 类似逻辑，使用 objectMapper.readValue(json, typeRef)
    }
}
```