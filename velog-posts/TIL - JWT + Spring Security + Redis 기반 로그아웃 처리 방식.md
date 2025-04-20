<p>오늘은 Spring Boot에서 JWT 인증 방식을 사용하는 환경에서 <strong>Redis를 활용한 로그아웃 처리</strong> 및 <strong>Refresh Token 관리</strong>에 대해 정리했다. 구현하며 느꼈던 포인트들과 코드 중심으로 정리해본다.</p>
<h2 id="왜-redis를-사용하는가">왜 Redis를 사용하는가?</h2>
<p>JWT는 서버에 세션을 저장하지 않는 Stateless 방식이라 로그아웃 시 토큰을 무효화시키는 것이 쉽지 않다.
이를 해결하기 위해 <strong>Redis를 블랙리스트 저장소</strong>로 활용한다.</p>
<ul>
<li><strong>AccessToken 블랙리스트 저장</strong> : 로그아웃 시 해당 토큰을 Redis에 저장하여 재사용 방지</li>
<li><strong>Refresh Token 저장 및 검증</strong> : 클라이언트가 갱신 요청 시 Redis에 저장된 Refresh Token을 기준으로 검증</li>
</ul>
<h2 id="redis-설정">Redis 설정</h2>
<pre><code class="language-java">@Bean
public RedisTemplate&lt;String, Object&gt; redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate&lt;String, Object&gt; template = new RedisTemplate&lt;&gt;();
    template.setConnectionFactory(connectionFactory);
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(genericJackson2JsonRedisSerializer());
    return template;
}</code></pre>
<h2 id="jwtauthfilter-내부-블랙리스트-검증-코드">JwtAuthFilter 내부 블랙리스트 검증 코드</h2>
<pre><code class="language-java">private void authenticate(HttpServletRequest request) {
    String token = this.getTokenFromHeader(request);
    if (!StringUtils.hasText(token) || !jwtUtil.validateToken(token)) return;

    // 블랙리스트 여부 확인
    String isLogout = (String) redisTemplate.opsForValue().get(token);
    if (&quot;logout&quot;.equals(isLogout)) return;

    String email = jwtUtil.getUserEmail(token);
    UserDetails userDetails = userDetailsService.loadUserByUsername(email);

    UsernamePasswordAuthenticationToken authenticationToken =
        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());

    authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
}</code></pre>
<h2 id="로그아웃-서비스-구현">로그아웃 서비스 구현</h2>
<pre><code class="language-java">@Override
public void logout(String accessToken, Long expirationTime) {
    // 블랙리스트 등록
    redisTemplate.opsForValue().set(accessToken, &quot;logout&quot;, Duration.ofMillis(expirationTime));

    // Refresh Token 삭제
    redisTemplate.delete(&quot;RT:&quot; + getEmailFromToken(accessToken));
}</code></pre>
<h2 id="refresh-token-저장-예시">Refresh Token 저장 예시</h2>
<pre><code class="language-java">@Override
public void saveRefreshToken(String email, String refreshToken, Long ttl) {
    redisTemplate.opsForValue().set(&quot;RT:&quot; + email, refreshToken, Duration.ofMillis(ttl));
}</code></pre>
<h2 id="refresh-token-검증-및-재발급-로직-예시">Refresh Token 검증 및 재발급 로직 예시</h2>
<pre><code class="language-java">public String reissueAccessToken(String email, String providedRefreshToken) {
    String savedRefreshToken = (String) redisTemplate.opsForValue().get(&quot;RT:&quot; + email);
    if (!providedRefreshToken.equals(savedRefreshToken)) {
        throw new CustomException(TokenErrorCode.INVALID_REFRESH_TOKEN);
    }

    return jwtUtil.generateAccessToken(userRepository.findByEmail(email).get());
}</code></pre>
<hr />
<h4 id="유의할-점">유의할 점</h4>
<ul>
<li>토큰 만료 시간과 Redis TTL을 맞춰줘야 불필요한 메모리 낭비를 줄일 수 있다</li>
<li>Filter, JwtUtil, RedisConfig 사이 의존성도 잘 분리해둬야 유지보수가 쉬움</li>
<li>RedisTemplate를 여러 서비스에서 사용할 경우, 유틸 클래스로 추출해 사용하는 것도 고려할 수 있다</li>
<li>Refresh Token의 경우도 Redis에 저장되므로 탈취에 유의하고 HTTPS 등 보안 대책 필수</li>
</ul>
<h4 id="느낀-점">느낀 점</h4>
<p>처음에는 Stateless 인증 구조에서 로그아웃 구현이 어렵게 느껴졌지만, Redis를 활용한 블랙리스트 패턴과 Refresh Token 저장 방식으로 충분히 실용적인 방법이 가능하다는 걸 깨달았다. Spring Security와 잘 연동되도록 설계하는 것이 핵심이었다.</p>