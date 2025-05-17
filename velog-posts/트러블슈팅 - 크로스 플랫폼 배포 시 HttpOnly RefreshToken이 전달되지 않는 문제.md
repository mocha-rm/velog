<h2 id="프로젝트-개요">프로젝트 개요</h2>
<p>현재 GigSync라는 인디밴드 커뮤니티 웹 어플리케이션을 개발하고 있다.</p>
<ul>
<li>프론트엔드 : Vite + React (localhost:5173)</li>
<li>백엔드 : Spring Boot (localhost:8080)</li>
<li>인증 방식 : JWT + Refresh Token (HttpOnly Cookie)</li>
</ul>
<p>로컬 환경에서는 아무 문제 없이 RefreshToken이 쿠키로 잘 전달되고 인증이 매끄럽게 동작했다.
그러나 EC2에 프론트와 백엔드를 각각 배포하면서 문제가 발생했다.</p>
<h3 id="발생한-문제--로그인-직후-로그아웃됨">발생한 문제 : 로그인 직후 로그아웃됨</h3>
<p>프론트 EC2에서는 로그인 요청을 정상적으로 보내고, 백엔드 EC2에서는 RefreshToken을 Set-Cookie 헤더에 담아 응답했음에도 불구하고, 쿠키가 클라이언트에 저장되지 않았다.
결과적으로 인증 상태가 유지되지 않아 로그인 직후 자동 로그아웃되는 현상이 발생했다.</p>
<h3 id="원인분석--크로스-도메인--보안-정책">원인분석 : 크로스 도메인 + 보안 정책</h3>
<ol>
<li><p><strong>Same-Origin Policy와 크로스 도메인</strong>
브라우저는 보안상 <strong>출처(origin)</strong>가 다른 경우 쿠키를 자동으로 저장하거나 전송하지 않는다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>프론트</th>
<th>백엔드</th>
<th>Same-Origin인가?</th>
</tr>
</thead>
<tbody><tr>
<td>로컬</td>
<td><a href="http://localhost:5173">http://localhost:5173</a></td>
<td><a href="http://localhost:8080">http://localhost:8080</a></td>
<td>X (포트 다름)</td>
</tr>
<tr>
<td>EC2 배포</td>
<td><a href="http://FRONT_IP">http://FRONT_IP</a></td>
<td><a href="http://BACK_IP">http://BACK_IP</a></td>
<td>X (도메인 다름)</td>
</tr>
</tbody></table>
<ul>
<li><strong>포트가 다르거나 도메인이 다르면 Same-Origin이 아니다.</strong></li>
</ul>
</li>
<li><p><strong>Set-Cookie의 보안 속성</strong>
백엔드가 보낸 쿠키가 브라우저에 저장되려면 아래 조건을 만족해야 한다.</p>
<table>
<thead>
<tr>
<th>속성</th>
<th>설명</th>
<th>테스트 환경에서 주의할 점</th>
</tr>
</thead>
<tbody><tr>
<td>HttpOnly</td>
<td>자바스크립트로 접근 불가</td>
<td>로그인 보안 향상</td>
</tr>
<tr>
<td>Secure</td>
<td>HTTPS에서만 동작</td>
<td><strong>로컬이나 HTTP EC2에서는 작동 안 함</strong></td>
</tr>
<tr>
<td>SameSite=None</td>
<td>크로스 사이트 요청 허용</td>
<td><code>Secure</code>가 반드시 같이 설정되어야 함</td>
</tr>
</tbody></table>
<p>즉, SameSite=None; Secure 조합은 HTTPS가 아니면 무효하다.</p>
</li>
</ol>
<h3 id="해결-방법">해결 방법</h3>
<h4 id="테스트-환경에서의-임시-해결프론트--백엔드를-단일-ec2에-배포">테스트 환경에서의 임시 해결(프론트 + 백엔드를 단일 EC2에 배포)</h4>
<ul>
<li>NGINX 또는 Caddy 등으로 <strong>Reverse Proxy를</strong> 구성하여 하나의 도메인 또는 포트로 동작하게 한다.</li>
<li>로컬처럼 동일 Origin이 되므로 쿠키가 정상 동작함</li>
</ul>
<h4 id="실제-서비스-운영-시-해결-방안">실제 서비스 운영 시 해결 방안</h4>
<ol>
<li><strong>도메인 구입 및 HTTPS 적용</strong><ul>
<li><strong>SSL 인증서</strong>로 HTTPS 적용</li>
<li>Secure 속성이 붙은 쿠키도 정상 작동</li>
</ul>
</li>
<li><strong>백엔드 응답 헤더 설정</strong></li>
</ol>
<pre><code class="language-java">ResponseCookie.from(&quot;refreshToken&quot;, token)
    .httpOnly(true)
    .secure(true) // HTTPS 환경에서만 작동
    .sameSite(&quot;None&quot;) // 크로스 도메인 허용
    .path(&quot;/&quot;)
    .build();</code></pre>
<ol start="3">
<li><strong>프론트엔드 요청 시 설정</strong></li>
</ol>
<pre><code class="language-ts">axios.post('https://api.gigsync.com/login', data, {
  withCredentials: true // 반드시 설정
});</code></pre>
<h3 id="쿠키-전달-문제-점검-체크리스트">쿠키 전달 문제 점검 체크리스트</h3>
<table>
<thead>
<tr>
<th>체크 항목</th>
<th>확인 내용</th>
</tr>
</thead>
<tbody><tr>
<td>withCredentials 설정 여부</td>
<td>프론트 axios/fetch에서 반드시 포함</td>
</tr>
<tr>
<td>Set-Cookie에 SameSite 포함 여부</td>
<td>백엔드 쿠키 설정 확인</td>
</tr>
<tr>
<td>HTTPS 환경 여부</td>
<td>Secure 쿠키는 HTTPS에서만 유효</td>
</tr>
<tr>
<td>도메인/서브도메인 설정</td>
<td>크로스 도메인 여부에 따라 SameSite 고려</td>
</tr>
</tbody></table>
<h2 id="끝으로">끝으로..</h2>
<p>테스트 환경에서는 동일 EC2에 프론트와 백엔드를 배치하여 문제를 해결했지만, 서비스 <strong>확장성</strong>과 <strong>보안성</strong>을 고려하면 도메인을 이용한 HTTPS 기반의 분리 배포가 필수이다. HttpOnly Cookie 기반 인증은 보안상 유리하지만, <strong>크로스 도메인 환경에서의 제약</strong>을 반드시 이해하고 맞춰줘야 한다는 걸 알 수 있었다.</p>