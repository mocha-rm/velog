<h2 id="문제-발생">문제 발생</h2>
<p>Spring Boot 어플리케이션에서 JUnit과 Mockito를 사용해 테스트하던 중 UserAuthorizationUtil.getLoginUser()를 호출하는 코드에서 다음과 같은 예외가 발생했다.</p>
<blockquote>
<p>java.lang.IllegalArgumentException: 인증된 사용자가 없습니다.
    at com.shineidle.tripf.common.util.UserAuthorizationUtil.getUserDetails(UserAuthorizationUtil.java:19)
    at com.shineidle.tripf.common.util.UserAuthorizationUtil.getLoginUser(UserAuthorizationUtil.java:48)</p>
</blockquote>
<h2 id="원인-추론">원인 추론</h2>
<p><strong>1. SecurityContextHolder의 인증 정보 부족</strong>
<strong>UserAuthorizationUtil.getLoginUser()</strong> 내부에서는 <strong>SecurityContextHolder.getContext().getAuthentication()</strong> 을 호출해 현재 로그인한 사용자를 가져오는데, 테스트 환경에서는 <strong>SecurityContextHolder</strong>가 초기화되지 않이 null이 반환되었다.</p>
<p><strong>2. Spring Security의 Authentication 객체 미설정</strong>
Spring Security는 HTTP 요청을 처리할 때 <strong>SecurityConext를</strong> 자동으로 설정하지만, 단위 테스트 환경에서는 이를 직접 설정해야 한다. 그렇지 않으면 <strong>authentication == null || !authentication.isAuthenticated()</strong> 조건을 만족하여 <strong>IllegalArgumentException(&quot;인증된 사용자가 없습니다.&quot;)</strong> 예외가 발생한다.</p>
<h2 id="해결-방안">해결 방안</h2>
<h3 id="방법-1--securitycontextholder에-가짜-인증-정보-설정">방법 1 : SecurityContextHolder에 가짜 인증 정보 설정</h3>
<p>테스트 실행 전에 SecurityContextHolder에 가짜 인증 정보를 설정하여 UserAuthorizationUtil.getLoginUser()가 정상적으로 동작하도록 만든다</p>
<pre><code class="language-java">import org.junit.jupiter.api.BeforeEach;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import com.shineidle.tripf.common.security.UserDetailsImpl;

@BeforeEach
void setUp() {
    user = new User();
    user.setId(1L);
    user.setEmail(&quot;test@example.com&quot;);

    UserDetailsImpl userDetails = new UserDetailsImpl(user);
    UsernamePasswordAuthenticationToken auth =
        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());

    SecurityContext securityContext = SecurityContextHolder.createEmptyContext();
    securityContext.setAuthentication(auth);
    SecurityContextHolder.setContext(securityContext);
}</code></pre>
<p>이렇게 설정하면 <strong>SecurityContextHolder</strong>가 인증 정보를 갖게 되어, <strong>UserAuthorizationUtil.getLoginUser()</strong> 호출 시 정상적으로 User 객체를 반환한다.</p>
<h3 id="방법-2--reflection을-사용해-userauthorizationutil-내부-필드-강제-설정">방법 2 : Reflection을 사용해 UserAuthorizationUtil 내부 필드 강제 설정</h3>
<p><strong>UserAuthorizationUtil</strong>이 <strong>SecurityContextHolder</strong>에 의존하는 것이 아닌, 특정 객체를 참조하는 방식이라면 _<strong>ReflectionTestUtils</strong>_를 사용하여 내부 필드를 강제로 설정하는 방법도 있다.</p>
<pre><code class="language-java">import org.springframework.test.util.ReflectionTestUtils;

@BeforeEach
void setUp() {
    user = new User();
    user.setId(1L);

    UserDetailsImpl userDetails = new UserDetailsImpl(user);
    ReflectionTestUtils.setField(UserAuthorizationUtil.class, &quot;authenticatedUser&quot;, userDetails);
}</code></pre>
<p>이 방법은 <strong>SecurityContextHolder가</strong> 아닌 특정 필드를 참조하는 경우에만 유효하다.</p>
<h2 id="결과-확인">결과 확인</h2>
<p>위 방법을 적용한 후 다시 테스트를 실행한 결과 예외 발생 없이, 정상적으로 테스트 코드가 실행되었다.</p>
<ul>
<li>Spring Security가 관리하는 <strong>SecurityContext는</strong> 단위 테스트 환경에서 자동으로 설정되지 않으므로, 명시적으로 <strong>Mock</strong> 데이터를 주입하는 것이 중요.</li>
<li><strong>SecurityContextHolder를</strong> 활용하는 방식이 가장 일반적인 해결책이며, 특정 클래스 내부의 필드를 직접 수행해야 한다면 <strong>ReflectionTestUtils를</strong> 고려할 수 있다.</li>
</ul>