<h1 id="스프링부트-테스트의-종류">스프링부트 테스트의 종류</h1>
<p>스프링부트 테스트는 주로 다음과 같이 나뉜다.</p>
<ul>
<li>단위 테스트(Unit Test) : 개별 클래스나 메서드의 동작을 검증 (Mock 객체를 자주 사용)</li>
<li>통합 테스트(Integration Test) : 애플리케이션의 여러 계층 (Service, Repository 등)이 상호작용하는 동작을 검증 -&gt; 실제 환경과 유사하게 실행</li>
</ul>
<h1 id="mock-객체란">Mock 객체란?</h1>
<p>Mock 객체는 테스트 환경에서 실제 객체를 대신하여 동작하는 <strong>가짜 객체</strong>이다. 원하는 동작을 시뮬레이션하거나 특정 결과를 반환하도록 설정할 수 있어, 테스트를 <strong>단순화하고 실행 속도를 빠르게</strong> 한다.</p>
<p><strong>장점</strong></p>
<ul>
<li>외부 의존성을 제거해 테스트가 독립적으로 실행된다.</li>
<li>네트워크나 데이터베이스 접근 없이 테스트가 가능하다.</li>
<li>실패 상황이나 예외를 쉽게 시뮬레이션 할 수 있다.</li>
</ul>
<p><strong>Mock 객체를 사용하는 경우</strong></p>
<ol>
<li>외부 API와의 연동 테스트
외부 API가 느리거나 비싸게 동작할 수 있으므로, Mock객체로 빠르고 안전한 테스트 가능</li>
<li>데이터베이스를 사용하지 않아도 되는 경우
단순 서비스 로직 테스트</li>
<li>실패 시나리오 테스트
네트워크 오류, API 타임아웃 등을 시뮬레이션</li>
</ol>
<p><em>Mock 객체를 사용한 테스트 예제 :</em></p>
<pre><code class="language-java">@ExtendWith(MockitoExtension.class)
class WeatherServiceTest {

    @Mock
    private WeatherApiClient weatherApiClient;

    @InjectMocks
    private WeatherService weatherService;

    @Test
    void getWeather_returnsMockedData() {
        when(weatherApiClient.fetchWeather(&quot;Seoul&quot;))
            .thenReturn(new WeatherResponse(&quot;Sunny&quot;, 25));

        WeatherResponse response = weatherService.getWeather(&quot;Seoul&quot;);

        assertEquals(&quot;Sunny&quot;, response.getCondition());
        assertEquals(25, response.getTemperature());
    }
}</code></pre>
<h1 id="실제-로직-테스트">실제 로직 테스트</h1>
<p>실제 로직 테스트는 Mock 객체를 사용하지 않고, <strong>실제 데이터베이스나 외부 API</strong>를 호출하여 동작을 검증한다. 주로 통합 테스트에서 활용되며, Mock 테스트가 제공하지 못하는 <strong>상호작용과 데이터 흐름</strong>을 확인할 수 있다.</p>
<p>장점</p>
<ul>
<li>애플리케이션 전체적인 동작을 검증</li>
<li>의존성 간에 상호작용 및 데이터 흐름을 확인할 수 있다.</li>
<li>실제 환경과 유사한 조건에서 테스트할 수 있다.</li>
</ul>
<p>실제 테스트가 필요한 경우</p>
<ol>
<li>서비스와 레포지토리 간 상호작용 검증 -&gt; 데이터베이스 CRUD 테스트</li>
<li>외부 API 호출 테스트 -&gt; Mock이 아닌 실제 API를 호출하여 데이터의 정확성 확인</li>
<li>전체적인 시스템 동작 확인
컨트롤러 -&gt; 서비스 -&gt; 레포지토리 흐름이 올바르게 작동하는지 검증</li>
</ol>
<p><em>실제 로직을 사용한 테스트 예제 :</em></p>
<pre><code class="language-java">@SpringBootTest
class WeatherServiceIntegrationTest {

    @Autowired
    private WeatherService weatherService;

    @Test
    void getWeather_returnsActualData() {
        WeatherResponse response = weatherService.getWeather(&quot;Seoul&quot;);

        assertNotNull(response);
        assertTrue(response.getTemperature() &gt; -50); // 온도 범위 예시
    }
}</code></pre>
<h2 id="mock-테스트와-실제-테스트의-조합">Mock 테스트와 실제 테스트의 조합</h2>
<p>Mock으로 단위 테스트를 작성해 개별 로직을 빠르게 검증한다.
통합 테스트에서는 Mock 없이 실제 리소스를 사용해 전체적인 동작을 확인한다.
외부 API 테스트는 Mock으로 기본적인 동작을 확인하고, 필요 시 실제 API를 호출하는 테스트를 추가한다.</p>
<p>결론적으로, <strong>Mock 테스트는 빠르고 예측 가능한 테스트</strong>에 유리하고 <strong>실제 테스트는 데이터 흐름과 전체적인 시스템 동작을 검증</strong>하는데 필수적이다.</p>