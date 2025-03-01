<h2 id="문제-발생">문제 발생</h2>
<p>Spring Boot 프로젝트에서 OAuth2 로그인 구현 중, 어플리케이션이 실행되지 않고 순환 참조 문제가 발생했다.</p>
<pre><code class="language-plain">The dependencies of some of the beans in the application context form a cycle:

┌─────┐
|  webConfig defined in file [/path/to/project/WebConfig.class]
↑     ↓
|  OAuth2AuthenticationSuccessHandler defined in file [/path/to/project/OAuth2AuthenticationSuccessHandler.class]
↑     ↓
|  OAuth2UserUnlinkManager defined in file [/path/to/project/OAuth2UserUnlinkManager.class]
↑     ↓
|  googleOAuth2UserUnlink defined in file [/path/to/project/GoogleOAuth2UserUnlink.class]
└─────┘</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/793f4061-1bd7-4911-adf5-9549a4a4cc62/image.png" /></p>
<p>Spring Boot 2.6 이상에서는 순환 참조가 기본적으로 금지되어 있어 어플리케이션이 시작되지 않았다.</p>
<h2 id="원인-추론">원인 추론</h2>
<p>콘솔 화면을 확인해보면 친절하게도 어떻게 순환 참조가 발생하고 있는지 확인 할 수 있다.</p>
<ul>
<li>의존성 순환 구조 :
여러 클래스 간 상호 의존성이 순환 구조를 형성했다.<ul>
<li>WebConfing -&gt; OAuth2AuthenticationSuccessHandler</li>
<li>OAuth2AuthenticationSuccessHandler -&gt; OAuth2UserUnlinkManager</li>
<li>OAuth2UserUnlinkManager -&gt; GoogleOAuth2UserUnlink</li>
<li>GoogleOAuth2UserUnlink -&gt; WebConfig</li>
</ul>
</li>
</ul>
<p>각 클래스가 서로의 빈(Bean)을 주입받는 과정에서 순환 참조가 발생했다.</p>
<ul>
<li>RestTemplateBuilder의 반복 사용 문제:
프로젝트에서 여러 구성 요소가 동일한 방식으로 RestTemplateBuilder를 직접 주입받아 순환 참조 문제를 악화시켰다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/a9309640-3e28-4414-b93d-28634552ef03/image.png" /></p>
<p>WebConfig 에서 RestTemplate를 생성자로 주입하고 있고,</p>
<details>
OAuth2UserUnlink

<p><img alt="Image 1" src="https://velog.velcdn.com/images/jelog_131/post/5c0680a0-eb4d-4f37-9a81-7eb2b7b3b5ca/image.png" /></p>
<p><img alt="Image 2" src="https://velog.velcdn.com/images/jelog_131/post/a7096373-c9cf-4a20-a690-a881dea24a06/image.png" /></p>
<p><img alt="Image 3" src="https://velog.velcdn.com/images/jelog_131/post/8185b401-32db-45c8-ad85-1385239ff85a/image.png" /></p>
</details>

<p>각각의 OAuth2UserUnlink 클래스에서 동일한 방식으로 주입.</p>
<h2 id="해결-방법">해결 방법</h2>
<ol>
<li>팩토리 클래스 사용</li>
</ol>
<ul>
<li>RestTemplateBuilder를 여러 곳에서 직접 사용하지 않도록, 팩토리 클래스를 도입하여 RestTemplate 객체 생성을 통합</li>
</ul>
<pre><code class="language-java">@Component
public class RestTemplateFactory {
    private final RestTemplateBuilder restTemplateBuilder;

    public RestTemplateFactory(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplateBuilder = restTemplateBuilder;
    }

    public RestTemplate create() {
        return restTemplateBuilder.build();
    }
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/456f4813-d3a0-4037-838d-410b37123ce0/image.png" /></p>
<p>이후 필요한 곳에서 RestTemplate를 직접 생성하는 대신 RestTemplateFactory의 create() 메서드를 호출하도록 수정</p>
<pre><code class="language-java">@Service
public class SomeService {
    private final RestTemplate restTemplate;

    public SomeService(RestTemplateFactory restTemplateFactory) {
        this.restTemplate = restTemplateFactory.create();
    }
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/7e21b0e4-6473-43f2-95f7-bbc4cb8b7bf3/image.png" /></p>
<ol start="2">
<li>구조 분리</li>
</ol>
<ul>
<li>순환 참조를 끊기 위해 의존성을 재설계<ul>
<li>GoogleOAuth2UserUnlink -&gt; WebConfig로의 직접 참조를 제거</li>
<li>의존성을 최소화하고 필요한 경우 인터페이스를 도입하여 간접적으로 주입하도록 변경</li>
</ul>
</li>
</ul>
<ol start="3">
<li>@Lazy 적용</li>
</ol>
<ul>
<li><p>일부 순환 참조가 불가피한 경우, 지연 로드(@Lazy)를 사용하여 의존성을 해결</p>
<pre><code class="language-java">@Service
public class OAuth2AuthenticationSuccessHandler {
  private final OAuth2UserUnlinkManager unlinkManager;

  public OAuth2AuthenticationSuccessHandler(@Lazy OAuth2UserUnlinkManager unlinkManager) {
      this.unlinkManager = unlinkManager;
  }
}</code></pre>
</li>
</ul>
<ol start="4">
<li>Spring 설정 변경 (임시)</li>
</ol>
<ul>
<li>문제를 디버깅하는 동안 application.properties에 설정을 추가하여 순환 참조 허용<pre><code class="language-yaml">spring.main.allow-circular-references=true</code></pre>
</li>
</ul>
<h2 id="결과-확인">결과 확인</h2>
<p>RestTemplateFactory를 도입함으로써 RestTemplateBuilder의 반복 사용을 제거하고, 순환 참조 문제를 해결할 수 있었다. 
의존성 설계의 중요성과 해결 방안에 대한 좋은 학습 기회가 되었다.</p>