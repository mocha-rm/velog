<h3 id="1-cors-문제">1. CORS 문제</h3>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/fe1710be-2a14-46ec-bbb3-f46d6950b5ce/image.png" /></p>
<p>주석을 해제하면 setAllowedOrigins(&quot;*&quot;) 설정이 적용되면서 특정 브라우저 환경에서 CORS 오류가 발생할 수 있다. 특히 SockJS는 기본적으로 다양한 전송 방식을 사용하므로, 일부 전송 방식에서는 setAllowedOrigins(&quot;*&quot;) 설정이 올바르게 동작하지 않을 수 있다.</p>
<h4 id="해결-방법">해결 방법</h4>
<ul>
<li>setAllowedOrigins(&quot;*&quot;)대신 setAllowedOriginPatterns(&quot;*&quot;)를 사용한다.</li>
<li>보안을 위해 허용할 도메인을 명확히 지정하는 것이 좋다<pre><code class="language-java">registry.addEndpoint(&quot;/ws-stomp&quot;)
      .setAllowedOriginPatterns(&quot;https://example.com&quot;);</code></pre>
</li>
</ul>
<h3 id="2-sockjs와-jwt-핸드셰이크-문제">2. SockJS와 JWT 핸드셰이크 문제</h3>
<p>JwtHandshakeInterceptor가 WebSocket 핸드셰이크 과정에서 JWT를 검사하는데, SockJS가 XHR Polling이나 iframe을 사용할 경우 요청 헤더에 토큰을 포함하지 못하는 문제가 발생할 수 있다.</p>
<h4 id="해결-방법-1">해결 방법</h4>
<ul>
<li>토큰을 Authorization 헤더가 아닌 쿠키로 전달하는 방식으로 변경한다.</li>
<li>JwtHandshakeInterceptor에서 쿠키를 읽어 토큰을 검증하도록 수정한다.</li>
</ul>
<pre><code class="language-java">public class JwtHandshakeInterceptor implements HandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, 
                                   WebSocketHandler wsHandler, Map&lt;String, Object&gt; attributes) {
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
            Cookie[] cookies = servletRequest.getServletRequest().getCookies();
            if (cookies != null) {
                for (Cookie cookie : cookies) {
                    if (&quot;jwt&quot;.equals(cookie.getName())) {
                        String token = cookie.getValue();
                        if (validateToken(token)) {
                            attributes.put(&quot;jwt&quot;, token);
                            return true;
                        }
                    }
                }
            }
        }
        return false;
    }
}</code></pre>
<h3 id="3-sockjs-polling과-서버-세션-문제">3. SockJS Polling과 서버 세션 문제</h3>
<p>SockJS는 폴백 방식 (XHR, Long Polling 등)을 사용할 때 서버에 HttpSession이 생성될 수 있다. 분산 서버 환경에서는 세션을 공유할 수 없으므로, JWT 기반 인증을 사용할 때 예기치 않은 세션 문제가 발생할 수 있다.</p>
<h4 id="해결-방법-2">해결 방법</h4>
<ul>
<li><p>Spring Boot에서 HttpSession을 강제로 사용하지 않도록 설정 -&gt; 시큐리티 필터 설정</p>
</li>
<li><p>spring.session.store-type=none 설정을 적용하여 세션을 비활성화한다.</p>
</li>
<li><p>JwtHandshakeInterceptor에서 HttpSession을 사용하지 않도록 수정</p>
<ul>
<li><p>securityFilterChain
<img alt="" src="https://velog.velcdn.com/images/jelog_131/post/5d34f553-9afa-46fd-956b-cc2cfd1ed410/image.png" /></p>
</li>
<li><p>application.properties</p>
<pre><code class="language-yaml">spring.session.store-type=none
server.servlet.session.timeout=0</code></pre>
</li>
</ul>
</li>
</ul>
<hr />
<p>웹소켓과 STOMP 환경에서 JWT를 사용하는 경우, CORS 문제, JWT 핸드셰이크 문제, SockJS의 세션 생성 문제 등이 발생할 수 있다. 이를 해결하기 위해 올바른 CORS 설정을 적용하고, JWT를 헤더가 아닌 다른 방식으로 전달하며, 세션을 비활성화하는 방식으로 설정을 최적화해야 한다.</p>