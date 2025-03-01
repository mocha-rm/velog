<h1 id="스프링-시큐리티에서-인증-처리-과정">스프링 시큐리티에서 인증 처리 과정</h1>
<h3 id="1-사용자의-요청">1. 사용자의 요청</h3>
<p>사용자가 아이디와 비밀번호를 전송한다.</p>
<h3 id="2-usernamepasswordauthenticationfilter">2. UsernamePasswordAuthenticationFilter</h3>
<p>스프링 시큐리티의 기본 필터로, 사용자가 입력한 인증 정보를 Authentication 객체로 변환한다.</p>
<h3 id="3-authenticationmanager">3. AuthenticationManager</h3>
<p>인증 정보를 처리하고 실제 인증이 유효한지 검증하는 역할을 한다. 기본적으로 여러 인증 공급자(AuthenticationProvider)를 위임받아 처리한다.</p>
<h3 id="4-userdetailsservice">4. UserDetailsService</h3>
<p>사용자 정보를 조회하는 서비스, 일반적으로 데이터베이스에서 사용자의 아이디와 비밀번호를 조회하는데 사용된다.</p>
<h3 id="5-인증-성공-or-실패">5. 인증 성공 or 실패</h3>
<ul>
<li>인증 성공 : 사용자의 세션에 인증 정보를 저장</li>
<li>인증 실패 : 예외 메시지 출력</li>
</ul>