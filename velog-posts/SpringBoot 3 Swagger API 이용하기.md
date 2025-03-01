<p>API를 테스트 하기 위해 Postman을 이용 중 이었는데 팀원들과 개발을 진행하면서 서로 만든 API를 좀 더 쉽게 통합하고 테스트 시간을 단축하고 싶어서 Swagger를 한번 적용해보는 과정을 정리해봤다.</p>
<h2 id="문제-발생">문제 발생</h2>
<p>Swagger를 사용하기 위해 먼저 build.gradle에 관련 의존성을 추가해 줘야한다.</p>
<blockquote>
<p>implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/1bc834a2-55da-44fc-8315-ef7c7abff844/image.png" /></p>
<p>그리고 Configuration을 작성하여 Bean으로 등록시켜준다.
어플리케이션을 실행 시키고 <a href="http://localhost:8080/swagger-ui/index.html">http://localhost:8080/swagger-ui/index.html</a> 로 접속을 시도했는데...</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/61ae8c3d-5fd6-429e-9059-9e18a444bb78/image.png" /></p>
<p>'Faild to load API definition.' 이라는 문구와 함께 정상적으로 접속이 되지 않은 모습을 볼 수 있다 ..</p>
<h2 id="원인-추론">원인 추론</h2>
<p>먼저 콘솔 로그를 확인해 봤다. -&gt; '로그인을 해주세요' 라는 문구가 출력 
로그인 필터에서 해당 도메인을 필터링 하지 않도록 수정이 필요하다고 생각했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/0a39f142-8596-431e-bbfe-e9dc51c32948/image.png" /></p>
<h2 id="재시도-1-실패">재시도 1 (실패)</h2>
<p>WHITE_LIST에 Swagger 관련 도메인을 넣어주었고 다시 시도해봤지만 로그인이 필요하다는 예외 문구는 사라졌을 뿐, 이번에도 원하는 결과는 얻지 못했다.</p>
<h2 id="재시도-2-성공">재시도 2 (성공)</h2>
<p>블로그와 Swagger 공식 Docs를 확인해보니 Swagger UI를 적용하는 방법은 별로 어렵지 않지만, 버전 충돌이 쉽게 발생할 수 있으므로 이 부분은 유의해야한다고 한다.</p>
<p>현재 사용 중인 SpringBoot 3.4.1 버전과 호환되는 Swagger 의존성을 찾지 못하였고 결국 서로 호환되는 버전인 SpringBoot 3.2.4 그리고 springdoc 2.4.0 버전을 임포트 시켰다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/c3b3edc1-0f5f-4983-9b52-f6d5e47c54b2/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/ed5984f6-9416-4d6b-8ce8-c5329efd35e0/image.png" /></p>
<h2 id="결과-확인">결과 확인</h2>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/15867919-ef09-41cc-9dae-f09cf9750c70/image.png" /></p>
<p>정상적으로 SwaggerUI에 접속이 된 모습이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/a8032154-1ef5-464d-8867-91493d96e02c/image.png" /></p>
<p>SpringBoot에 작성된 코드를 기반으로 API를 불러오고 바로 테스트도 할 수 있으니 편리할 것 같다. 하지만 비지니스 코드에 Swagger 관련된 어노테이션등 몇 가지 추가를 해줘야 해서 자칫 코드가 지저분해 질 수 있다. 이 부분에 대해서는 좀 더 생각해보고 더 나은 방법이 있는지 고민해 봐야겠다.</p>