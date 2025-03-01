<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/8788bdbe-a127-4310-8ec0-86d453f46f69/image.png" /></p>
<p>UserController에 Logout 메서드를 만들고 Postman에서 API를 호출했다.</p>
<h2 id="문제-발생">문제 발생</h2>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/41ab8c8b-f264-4514-aa9b-62a363f1c222/image.png" /></p>
<p>먼저 로그인 API를 호출해서 토큰을 받고, 로그아웃 API의 Authorization을 Bearer Token으로 설정해준 후 API를 호출하게 되면 다음과 같은 결과가 나온다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/c43bf7b6-61b8-4fa1-baee-0541e4f1826d/image.png" /></p>
<p>&quot;error&quot;: &quot;Method Not Allowed&quot;,
&quot;message&quot;: &quot;Method 'GET' is not supported.&quot;,
&quot;path&quot;: &quot;/login&quot;</p>
<p>라는 응답이 나왔는데, 먼저 path가 /login으로 되어있는 모습을 보고 의아했다. 분명 /logout이라는 엔드포인트를 입력했고 API를 호출했기 때문이다..</p>
<h2 id="원인-추론">원인 추론</h2>
<p>혹시 캐시 때문에 그럴수도 있겠다는 생각에 Postman 캐시를 초기화시키고 다시 시도 해봤지만 결과는 동일했다.
또 다른 원인으로는 Spring Security의 기본 설정 문제일 수도 있다. Spring Security는 <strong>기본적으로 /logout 엔드포인트를 제공하며, HTTP GET 요청으로 처리하려고 한다.</strong> 커스텀 /logout을 만들 경우, Spring Security의 기본 설정과 충돌이 날 수도 있다.</p>
<h2 id="문제-해결">문제 해결</h2>
<ol>
<li><p>경로를 변경 하기</p>
<pre><code class="language-java">@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
 @Override
 protected void configure(HttpSecurity http) throws Exception {
     http
         .logout()
             .logoutUrl(&quot;/custom-logout&quot;) // 기본 경로를 변경
             .invalidateHttpSession(true)
             .deleteCookies(&quot;JSESSIONID&quot;)
             .logoutSuccessHandler((request, response, authentication) -&gt; {
                 response.setStatus(HttpServletResponse.SC_OK);
                 response.getWriter().write(&quot;로그아웃 완료&quot;);
             })
         .and()
         .authorizeRequests()
             .anyRequest().authenticated();
 }
}</code></pre>
</li>
<li><p>/logout을 POST 요청으로만 허용하기</p>
<pre><code class="language-java">@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
 http.cors(AbstractHttpConfigurer::disable)
     .csrf(AbstractHttpConfigurer::disable)
     .authorizeHttpRequests(auth -&gt; auth
             .requestMatchers(WHITE_LIST).permitAll()
             .requestMatchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
             .dispatcherTypeMatchers(DispatcherType.FORWARD, DispatcherType.INCLUDE, DispatcherType.ERROR).permitAll()
             .requestMatchers(&quot;/admin/**&quot;).hasAuthority(&quot;AUTH_ADMIN&quot;)
             .anyRequest().authenticated()
     )
     .exceptionHandling(handler -&gt; handler
             .authenticationEntryPoint(authenticationEntryPoint)
             .accessDeniedHandler(accessDeniedHandler))
     .sessionManagement(session -&gt; session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
     .authenticationProvider(authenticationProvider)
     .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
     .logout(logout -&gt; logout
             .logoutUrl(&quot;/logout&quot;) // 로그아웃 URL 설정
             .logoutRequestMatcher(new AntPathRequestMatcher(&quot;/logout&quot;, &quot;POST&quot;)) // POST 요청만 허용
             .invalidateHttpSession(true) // 세션 무효화
             .deleteCookies(&quot;JSESSIONID&quot;) // 쿠키 삭제
             .logoutSuccessHandler((request, response, authentication) -&gt; {
                 response.setStatus(HttpServletResponse.SC_OK);
                 response.getWriter().write(&quot;로그아웃 완료&quot;);
             })
     );

 return http.build();
}</code></pre>
</li>
</ol>
<ul>
<li>&quot;/logout&quot; 엔드포인트를 유지 하면서도 기본으로 설정되어있던 GET 요청을 -&gt; POST 요청으로 변경하게 된다.</li>
</ul>
<p>하지만 프론트엔드가 붙게 되면서 기존 엔드포인트 앞에 모두 &quot;/api&quot;를 붙여주기로 약속했기에, 간단하게 엔드포인트를 수정하는 방법으로 해결이 되었다.</p>
<h2 id="결과-확인">결과 확인</h2>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/bc6eb06e-1bd3-402a-baf5-468fd7beeec9/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/9311bb88-9954-4e53-aed6-15a31f8e16f5/image.png" /></p>
<p>정상적으로 로그아웃이 진행된 모습을 확인 할 수 있다.</p>
<h3 id="추가-고려-사항">추가 고려 사항</h3>
<p>지금 로그아웃을 처리하는 방식은 SecurityContext에서 인증 객체를 제거하여 현재 세션에 대한 인증 상태를 삭제한다. 하지만 JWT 자체는 여전히 유효하기 때문에 클라이언트가 토큰을 그대로 가지고 있다면, 다른 요청을 보낼 때 서버는 이를 유효한 토큰으로 간주할 수 있다.
때문에, <strong>JWT 기반 시스템에서 조금 더 완전한 로그아웃 처리를 하기 위해 Refresh Token, 블랙리스트 사용(Redis) 등을 고려해볼 필요가 있을 것 같다.</strong></p>