<h2 id="문제-발생">문제 발생</h2>
<p>Spring Security를 사용한 사용자 인증 과정에서 Redis를 활용하려는 중에 다음과 같은 에러 문구를 확인할 수 있었다.</p>
<blockquote>
<p>java.lang.ClassCastException: class java.util.LinkedHashMap cannot be cast to class com.shineidle.tripf.user.entity.User</p>
</blockquote>
<p>이 문제는 사용자의 인증 정보를 Redis에 저장하고 이를 다시 조회하는 과정에서 발생했다.</p>
<h2 id="원인-추론">원인 추론</h2>
<ol>
<li>Redis 데이터 직렬화 문제 : Redis에 저장된 데이터를 역직렬화할 때 java.util.LinkedHashMap으로 변환되면서 클래스 캐스팅 에러가 발생했다. 이는 기본 직렬화 방식이 JDK 직렬화를 따르지 않거나, 사용자 객체(User)를 직렬화/역직렬화하는 설정이 불완전했기 때문이다.</li>
<li>Spring Security와 Redis 간의 호환성 문제 : Redis에 저장된 데이터를 올바른 User 객체로 변환하지 못하면서 Spring Security에서 기대하는 UserDetails 타입으로 매핑할 수 없었다.</li>
</ol>
<h2 id="해결-방법">해결 방법</h2>
<h4 id="redis-설정-수정">Redis 설정 수정</h4>
<ul>
<li>Redis와 객체 간의 직렬화 및 역직렬화를 원활히 처리하기 위해 RedisTemplate의 설정을 수정했다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/f77652da-d954-43f5-ad3a-3f7882ca4bc7/image.png" /></p>
<pre><code class="language-java">@Bean
public RedisTemplate&lt;String, Object&gt; redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate&lt;String, Object&gt; template = new RedisTemplate&lt;&gt;();
    template.setConnectionFactory(connectionFactory);

    template.setKeySerializer(new StringRedisSerializer());

    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.registerModule(new JavaTimeModule());
    objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

    objectMapper.activateDefaultTyping(
            objectMapper.getPolymorphicTypeValidator(),
            ObjectMapper.DefaultTyping.NON_FINAL
    );

    template.setValueSerializer(new GenericJackson2JsonRedisSerializer(objectMapper));
    return template;
}</code></pre>
<p>이 설정을 통해 Redis에 저장되는 데이터가 JSON 형식으로 직렬화되며, 저장된 데이터를 역직렬화할 때도 올바른 객체 타입으로 변환된다.</p>
<h4 id="loaduserbyusername-메서드-수정">loadUserByUsername 메서드 수정</h4>
<p>사용자 정보를 Redis에서 우선적으로 조회하고, 없을 경우 데이터베이스에서 가져오도록 수정했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/b9e2ff83-e314-4add-93b9-491104e2061d/image.png" /></p>
<pre><code class="language-java">public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User orFetchFromDB = redisUtils.getOrFetchFromDB(username, () -&gt; loadUser(username), Duration.ofMinutes(3));
    return new UserDetailsImpl(orFetchFromDB);
}

private User loadUser(String username) {
    return this.userRepository.findByEmail(username)
            .orElseGet(() -&gt; this.userRepository.findByProviderId(username)
                    .orElseThrow(() -&gt; new UsernameNotFoundException(&quot;유저를 찾을 수 없습니다.&quot;)));
}</code></pre>
<p>redisUtils.getOrFetchFromDB 메서드를 사용하여 Redis에 사용자 정보가 없으면 DB에서 조회하고, 해당 데이터를 Redis에 저장하는 방식으로 동작한다. 또한, 조회된 사용자 정보를 UserDetails로 변환해 반환한다.</p>
<ul>
<li>RedisUtils 클래스 코드<pre><code class="language-java">package com.shineidle.tripf.common.util;
</code></pre>
</li>
</ul>
<p>import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;</p>
<p>import java.time.Duration;</p>
<p>@Component
@RequiredArgsConstructor
public class RedisUtils {
    private final RedisTemplate&lt;String, Object&gt; redisTemplate;</p>
<pre><code>/**
 * Redis에 데이터를 저장
 * @param key Redis 키
 * @param value 저장할 값
 * @param ttl 만료 시간 (Duration)
 */
public void saveToRedis(String key, Object value, Duration ttl) {
    redisTemplate.opsForValue().set(key, value, ttl);
}

/**
 * Redis에서 데이터를 조회
 * @param key Redis 키
 * @return 조회한 값 (없으면 null)
 */
public Object getFromRedis(String key) {
    return redisTemplate.opsForValue().get(key);
}

/**
 * Redis에서 데이터를 조회하고 없으면 DB에서 가져옴
 * @param key Redis 키
 * @param dbFetcher DB에서 데이터를 조회하는 메서드
 * @param ttl 만료 시간 (Duration)
 * @return Redis 또는 DB에서 가져온 값
 */
public &lt;T&gt; T getOrFetchFromDB(String key, DbFetcher&lt;T&gt; dbFetcher, Duration ttl) {
    T value = (T) redisTemplate.opsForValue().get(key);

    if (value == null) {
        value = dbFetcher.fetch();
        saveToRedis(key, value, ttl);
    }

    return value;
}

// DB에서 데이터를 조회하는 메서드의 인터페이스
public interface DbFetcher&lt;T&gt; {
    T fetch();
}</code></pre><p>}</p>
<pre><code>
## 결과 확인

![](https://velog.velcdn.com/images/jelog_131/post/5d9ea2c8-58e8-4938-b2f3-501066868acb/image.png)

직렬화/역질렬화 문제가 해결되고 제대로 조회가 되는 모습을 확인할 수 있다.

![](https://velog.velcdn.com/images/jelog_131/post/141187cf-af17-4020-ac16-720c7c12cc18/image.png)

유저를 조회할 때 최초에 Redis에서 조회를 하게 되는데 데이터가 없으므로 DB에서 가져와 Redis에 캐싱해 놓고 데이터를 반환해 응답해주고있다. (Cache Aside 방식)

![](https://velog.velcdn.com/images/jelog_131/post/0f5b20e9-90a9-449d-abeb-505e1a19349e/image.png)

스크린샷을 보면 최초에 쿼리가 수행되고 다음부터는 나오지않는 모습을 확인할 수 있다.




</code></pre>