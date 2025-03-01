<h1 id="oauth2-소셜-로그인-흐름-요약">OAuth2 소셜 로그인 흐름 요약</h1>
<ol>
<li>사용자 인증 요청</li>
</ol>
<ul>
<li>사용자가 소셜 로그인을 요청하면, 클라이언트는 소셜 플랫폼으로 리다이렉션한다.</li>
</ul>
<ol start="2">
<li>권한 승인 및 인증 정보 반환</li>
</ol>
<ul>
<li>사용자가 소셜 플랫폼에서 권한을 승인하면, 인증 코드 또는 토큰이 반환된다.</li>
</ul>
<ol start="3">
<li>서버 인증 처리</li>
</ol>
<ul>
<li>반환된 정보를 바탕으로 서버는 사용자의 인증을 완료하고, 애플리케이션 내부의 인정 정보를 생성한다.</li>
</ul>
<ol start="4">
<li>토큰 발급 및 응답</li>
</ol>
<ul>
<li>인증이 완료되면 서버는 어플리케이션 내부에서 사용할 엑세스 토큰과 리프레시 토큰을 발급한다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/93f9a9fc-e626-4114-824c-72b787da7d65/image.png" /></p>
<hr />
<h1 id="소셜-로그인-과정">소셜 로그인 과정</h1>
<h3 id="httpcookieoauth2authorizationrequestrepository">HttpCookieOAuth2AuthorizationRequestRepository</h3>
<pre><code class="language-java">package com.shineidle.tripf.oauth2;

import com.shineidle.tripf.oauth2.util.CookieUtils;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.oauth2.client.web.AuthorizationRequestRepository;
import org.springframework.security.oauth2.core.endpoint.OAuth2AuthorizationRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.util.WebUtils;

@Slf4j
@Component
@RequiredArgsConstructor
public class HttpCookieOAuth2AuthorizationRequestRepository implements AuthorizationRequestRepository&lt;OAuth2AuthorizationRequest&gt; {
    public static final String OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME = &quot;oauth2_auth_request&quot;;
    private static final int COOKIE_EXPIRE_SECONDS = 18000;

    @Override
    public OAuth2AuthorizationRequest loadAuthorizationRequest(HttpServletRequest request) {
        Cookie cookie = WebUtils.getCookie(request, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME);
        return CookieUtils.deserialize(cookie, OAuth2AuthorizationRequest.class);
    }

    @Override
    public void saveAuthorizationRequest(OAuth2AuthorizationRequest authorizationRequest, HttpServletRequest request, HttpServletResponse response) {
        if (authorizationRequest == null) {
            removeAuthorizationRequestCookies(request, response);
            return;
        }

        CookieUtils.addCookie(response, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME, CookieUtils.serialize(authorizationRequest), COOKIE_EXPIRE_SECONDS);
    }

    @Override
    public OAuth2AuthorizationRequest removeAuthorizationRequest(HttpServletRequest request, HttpServletResponse response) {
        return this.loadAuthorizationRequest(request);
    }

    public void removeAuthorizationRequestCookies(HttpServletRequest request, HttpServletResponse response) {
        CookieUtils.deleteCookie(request, response, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME);
    }
}</code></pre>
<p>이 클래스는 OAuth2 인증 요청에 대한 쿠키를 관리한다.</p>
<ul>
<li>주요 메서드:<ul>
<li>saveAuthorizationRequest
소셜 로그인 과정의 초기 단계에서 호출된다.
인증 요청 정보를 쿠키에 저장</li>
<li>loadAuthorizationRequest
인증 요청 정보를 쿠키에서 읽어오는 역할을 한다.</li>
<li>removeAuthorizationRequest
인증 완료 후, 인증 요청 관련 쿠키를 삭제한다.</li>
</ul>
<hr />
</li>
</ul>
<h3 id="customoauth2userservice">CustomOAuth2UserService</h3>
<pre><code class="language-java">package com.shineidle.tripf.oauth2.service;

import com.shineidle.tripf.oauth2.exception.OAuth2AuthenticationProcessingException;
import com.shineidle.tripf.oauth2.user.OAuth2Provider;
import com.shineidle.tripf.oauth2.user.OAuth2UserInfo;
import com.shineidle.tripf.oauth2.user.OAuth2UserInfoFactory;
import com.shineidle.tripf.user.entity.User;
import com.shineidle.tripf.user.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.util.Optional;

@Slf4j
@Service
@RequiredArgsConstructor
public class CustomOAuth2UserService extends DefaultOAuth2UserService {
    private final UserRepository userRepository;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest oAuth2UserRequest) throws OAuth2AuthenticationException {
        OAuth2User oAuth2User = super.loadUser(oAuth2UserRequest);
        return processOAuth2User(oAuth2UserRequest, oAuth2User);
    }

    private OAuth2User processOAuth2User(OAuth2UserRequest userRequest, OAuth2User oAuth2User) {
        String registrationId = userRequest.getClientRegistration().getRegistrationId(); // Google, Kakao, Naver
        String accessToken = userRequest.getAccessToken().getTokenValue();

        log.info(&quot;소셜 로그인 플랫폼: {}&quot;, registrationId);
        log.info(&quot;소셜 로그인 엑세스 토큰: {}&quot;, accessToken);

        // OAuth2UserInfo 객체 생성
        OAuth2UserInfo oAuth2UserInfo = OAuth2UserInfoFactory.getOAuth2UserInfo(
                registrationId,
                accessToken,
                oAuth2User.getAttributes()
        );

        // 이메일 검증
        if (!StringUtils.hasText(oAuth2UserInfo.getEmail()) &amp;&amp; !OAuth2Provider.KAKAO.getRegistrationId().equals(registrationId)) {
            throw new OAuth2AuthenticationProcessingException(&quot;이메일을 찾을 수 없습니다.&quot;);
        }

        // 유저 정보 저장 또는 업데이트
        saveOrUpdate(oAuth2UserInfo);

        // OAuth2UserPrincipal 반환
        return new OAuth2UserPrincipal(oAuth2UserInfo);
    }

    private void saveOrUpdate(OAuth2UserInfo oAuth2UserInfo) {
        Optional&lt;User&gt; existUser = userRepository.findByEmail(oAuth2UserInfo.getEmail());

        if (existUser.isEmpty() &amp;&amp; OAuth2Provider.KAKAO.equals(oAuth2UserInfo.getProvider())) {
            existUser = userRepository.findByProviderId(oAuth2UserInfo.getId());
        }

        if (existUser.isPresent()) {
            // 기존 유저 정보 업데이트
            User user = existUser.get();
            user.update(oAuth2UserInfo);
        } else {
            User newUser;

            if (OAuth2Provider.KAKAO.equals(oAuth2UserInfo.getProvider())) {
                newUser = new User(null,
                        oAuth2UserInfo.getNickname(),
                        oAuth2UserInfo.getProvider().getRegistrationId(),
                        oAuth2UserInfo.getId()
                );
            } else {
                newUser = new User(oAuth2UserInfo.getEmail(),
                        oAuth2UserInfo.getName(),
                        oAuth2UserInfo.getProvider().getRegistrationId(),
                        oAuth2UserInfo.getId()
                        );
            }
            // 새로운 유저 생성
            userRepository.save(newUser);
        }
    }
}</code></pre>
<p>이 클래스는 소셜 로그인 후 사용자 정보를 처리한다.</p>
<ul>
<li><p>주요 메서드:</p>
<ul>
<li>loadUser
소셜 플랫폼에서 반환한 사용자 정보를 로드</li>
<li>processOAuth2User
반환된 사용자 정보를 기반으로 어플리케이션에 맞는 사용자 객체를 생성하거나 업데이트
기존 사용자 정보가 있을 경우 업데이트, 없을 경우 새로 저장</li>
<li>saveOrUpdate
사용자의 이메일, 이름 등 소셜 계정 정보를 데이터베이스에 저장하거나 갱신</li>
</ul>
<hr />
</li>
</ul>
<h3 id="oauth2authenticationsuccesshandler">OAuth2AuthenticationSuccessHandler</h3>
<pre><code class="language-java">package com.shineidle.tripf.oauth2.handler;

import com.shineidle.tripf.common.util.JwtProvider;
import com.shineidle.tripf.common.util.TokenType;
import com.shineidle.tripf.oauth2.HttpCookieOAuth2AuthorizationRequestRepository;
import com.shineidle.tripf.oauth2.service.OAuth2UserPrincipal;
import com.shineidle.tripf.oauth2.user.OAuth2Provider;
import com.shineidle.tripf.oauth2.util.CookieUtils;
import com.shineidle.tripf.user.entity.RefreshToken;
import com.shineidle.tripf.user.entity.User;
import com.shineidle.tripf.user.repository.UserRepository;
import com.shineidle.tripf.user.service.RefreshTokenService;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.web.util.UriComponentsBuilder;

import java.io.IOException;

@Slf4j
@Component
@RequiredArgsConstructor
public class OAuth2AuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {
    public static final String REDIRECT_URL = &quot;/&quot;;
    private final HttpCookieOAuth2AuthorizationRequestRepository httpCookieOAuth2AuthorizationRequestRepository;
    private final UserRepository userRepository;
    private final JwtProvider jwtProvider;
    private final RefreshTokenService refreshTokenService;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException {
        OAuth2UserPrincipal oAuth2UserPrincipal = getOAuth2UserPrincipal(authentication);

        User user = oAuth2UserPrincipal.getUserInfo().getProvider().equals(OAuth2Provider.KAKAO) ?
                userRepository.findByProviderId(oAuth2UserPrincipal.getUserInfo().getId()).orElseThrow(() -&gt;
                        new ResponseStatusException(HttpStatus.NOT_FOUND)) :
                userRepository.findByEmail(oAuth2UserPrincipal.getName()).orElseThrow(() -&gt;
                        new ResponseStatusException(HttpStatus.NOT_FOUND));

        String accessToken = jwtProvider.generateToken(authentication, true, TokenType.ACCESS);
        RefreshToken refreshToken = refreshTokenService.generateToken(user.getId(), authentication, true);

        // 쿠키에 사용
        addRefreshTokenToCookie(request, response, refreshToken);

        //Path 뒤에 accessToken 붙이기
        String targetUrl = getTargetUrl(accessToken);

        clearAuthenticationAttributes(request, response);

        // 리다이렉트
        getRedirectStrategy().sendRedirect(request, response, targetUrl);
    }

    protected String getTargetUrl(String accessToken) {
        return UriComponentsBuilder.fromUriString(REDIRECT_URL)
                .queryParam(&quot;accessToken&quot;, accessToken)
                .build().toUriString();
    }

    private OAuth2UserPrincipal getOAuth2UserPrincipal(Authentication authentication) {
        Object principal = authentication.getPrincipal();

        if (principal instanceof OAuth2UserPrincipal) {
            return (OAuth2UserPrincipal) principal;
        }

        return null;
    }

    private void addRefreshTokenToCookie(HttpServletRequest request, HttpServletResponse response, RefreshToken refreshToken) {
        int cookieMaxAge = jwtProvider.getRefreshExpiryMillis().intValue();
        CookieUtils.deleteCookie(request, response, &quot;refresh_token&quot;);
        CookieUtils.addCookie(response, &quot;refresh_token&quot;, refreshToken.getToken(), cookieMaxAge);
    }

    protected void clearAuthenticationAttributes(HttpServletRequest request, HttpServletResponse response) {
        super.clearAuthenticationAttributes(request);
        httpCookieOAuth2AuthorizationRequestRepository.removeAuthorizationRequestCookies(request, response);
    }
}</code></pre>
<p>소셜 로그인 성공 후 최종적으로 호출되는 클래스</p>
<ul>
<li><p>주요 메서드:</p>
<ul>
<li>onAuthenticationSuccess
인증이 성공적으로 이루어진 경우 호출
어플리케이션 내부의 액세스 토큰과 리프레시 토큰을 발급하고, 적절한 리다이렉션 URL을 설정</li>
<li>getTargetUrl
리다이렉션할 URL을 결정
Path뒤에 accessToken을 표시</li>
</ul>
<hr />
<h2 id="상세-흐름">상세 흐름</h2>
</li>
</ul>
<ol>
<li><p>사용자 요청 -&gt; 인증 요청 저장
사용자가 소셜 로그인 버튼을 클릭하면 서버는 HttpCookieOAuth2AuthorizationRequestRepository의 saveAuthorizationRequest 메서드를 호출하여 인증 요청 정보를 쿠키에 저장한다.</p>
</li>
<li><p>소셜 플랫폼 인증 -&gt; 사용자 정보 로드
소셜 플랫폼 인증이 완료되면, CustomOAuth2UserService의 loadUser가 호출되어 사용자의 정보를 로드한다.
processOAuth2User 메서드에서 어플리케이션에 맞는 사용자 정보를 생성하거나 업데이트한다.</p>
</li>
<li><p>로그인 성공 -&gt; 토큰 발급
인증이 성공하면, OAuth2AuthenticationSuccessHandler의 onAuthenticationSuccess가 호출된다.
이 메서드에서 액세스 토큰과 리프레시 토큰을 생성하고, 최종 리다이렉션 URL을 결정한다.</p>
</li>
<li><p>리다이렉션
getTargetUrl 메서드는 리다이렉션 Path와 함께 이어서 accessToken을 표시한다.</p>
</li>
</ol>