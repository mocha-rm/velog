<h2 id="문제-상황">문제 상황</h2>
<p>Spring Boot + JWT 인증 구조에서 로그인 시 refreshToken을 HttpOnly 쿠키에 담아 응답하도록 설정했다.</p>
<pre><code class="language-java">ResponseCookie refreshTokenCookie = ResponseCookie.from(&quot;refreshToken&quot;, refreshToken)
    .httpOnly(true)
    .secure(true)
    .path(&quot;/api/auth/refresh&quot;)
    .maxAge(Duration.ofDays(14))
    .sameSite(&quot;None&quot;)
    .build();

return ResponseEntity.ok()
    .header(HttpHeaders.SET_COOKIE, refreshTokenCookie.toString())
    .body(...);</code></pre>
<p>그러나 로컬에서 <a href="http://localhost%EB%A1%9C">http://localhost로</a> 서버를 실행했을 때, 브라우저에 쿠키가 저장되지 않았다.</p>
<h2 id="원인-분석">원인 분석</h2>
<p>쿠키 설정에서 아래 두 옵션이 문제였다.</p>
<ul>
<li>secure=true : HTTPS에서만 쿠키 전송을 허용</li>
<li>sameSite=&quot;None&quot; : 크로스 도메인 요청에서 쿠키 전송 허용. 단, secure=true 조건 필수</li>
</ul>
<p>따라서 로컬에서는 쿠키 자체가 전송되지 않는다. -&gt; 브라우저 개발자 도구에서도 보이지 않음</p>
<h2 id="해결-방법">해결 방법</h2>
<p>환경에 따라 쿠키 설정을 분기 처리하도록 수정하였다.</p>
<pre><code class="language-java">boolean isLocal = true; // 개발/운영 환경에 따라 설정

ResponseCookie refreshTokenCookie = ResponseCookie.from(&quot;refreshToken&quot;, refreshToken)
    .httpOnly(true)
    .secure(!isLocal)                          // 운영에서는 secure=true
    .path(&quot;/api/auth/refresh&quot;)
    .sameSite(isLocal ? &quot;Lax&quot; : &quot;None&quot;)        // 로컬에서는 Lax로 설정
    .maxAge(Duration.ofDays(14))
    .build();</code></pre>
<ul>
<li>로컬(localhost)<ul>
<li>secure=false, sameSite=Lax</li>
<li>쿠키 정상 전송됨 (브라우저 Application &gt; Cookies 탭에서 확인 가능)</li>
</ul>
</li>
<li>운영(https)<ul>
<li>secure=true, sameSite=None</li>
<li>크로스 사이트에서도 쿠키 전송 가능 (보안 유지)</li>
</ul>
</li>
</ul>
<h4 id="보안상-중요한-포인트">보안상 중요한 포인트</h4>
<ul>
<li>httpOnly=true 덕분에 JS에서는 document.cookie로 접근이 불가하다. -&gt; XSS 방어</li>
<li>브라우저 콘솔에서 쿠키가 안 보이는 건 정상 동작이다.</li>
</ul>