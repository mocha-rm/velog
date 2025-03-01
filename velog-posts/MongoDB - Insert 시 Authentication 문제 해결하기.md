<p>프로젝트에 채팅 기능을 올리면서 MongoDB를 사용하게됬다. Docker 이미지를 받고 컨테이너를 실행시킨 후 IntelliJ에서 플러그인으로 제공되는 'MongoDB Browser'를 이용했다.</p>
<h2 id="intellij와-mongodb-연동">IntelliJ와 MongoDB 연동</h2>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/43baa2bc-63c5-448e-bb37-bedbe083f9ac/image.png" /></p>
<p>IntelliJ를 재실행 후 오른쪽 메뉴바를 보면 MongoDB Browser 아이콘이 생긴 모습을 볼 수 있다. '+' 버튼을 눌러 직접 연동을 해주어야한다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/9fe31778-0e8a-4a11-9aee-8dd58b2eff6e/image.png" /></p>
<p>구분을 해주기위해 Label 값을 입력해주고 User Database에는 사용할 DB 이름을 설정해줬다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/c3f6dd56-b357-4874-a433-25923a176a35/image.png" /></p>
<p>Authentication 탭으로 넘어와서 Docker 컨테이너를 실행할 때 환경변수로 지정해줬던 username과 password 입력하고 'Auth.databse'는 admin을 입력해주었다.</p>
<hr />
<h2 id="postman을-이용한-테스트">Postman을 이용한 테스트</h2>
<p>직접적인 채팅을 주고 받기 전에 먼저 채팅방을 만드는 API를 테스트하기 위해 Postman을 이용했다.</p>
<ul>
<li>ChatRoomController.java</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/d7ed7e6b-30f8-451b-bf2a-90d76611816c/image.png" /></p>
<ul>
<li>Postman</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/bc1ff04a-4e57-4f3d-8968-c37362547c73/image.png" /></p>
<blockquote>
<p>com.mongodb.MongoCommandException: Command failed with error 13 (Unauthorized): 'Command insert requires authentication' on server localhost:27017. The full response is {&quot;ok&quot;: 0.0, &quot;errmsg&quot;: &quot;Command insert requires authentication&quot;, &quot;code&quot;: 13, &quot;codeName&quot;: &quot;Unauthorized&quot;}</p>
</blockquote>
<p>API 호출했지만 500 Internal Server Error가 발생했고 콘솔 로그 내용을 확인해보니 Insert 요청에는 권한이 필요하다는 내용이었다.</p>
<hr />
<h2 id="문제-해결">문제 해결</h2>
<p>MongoDB를 연동할 때 Authentication 탭에 있는 'Auth.databse'에 분명 'admin'으로 지정을 해주었지만 추가적으로 application.yml에도 지정을 해주었다.</p>
<ul>
<li>기존</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/758214b4-59d4-4c4d-b56a-d6382e4492cd/image.png" /></p>
<ul>
<li>수정</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/57a07b44-1e4e-49c3-8289-5b76b139832e/image.png" /></p>
<p>uri안에 포함되어있던 host와 port 값을 각각 host, port에 입력해주었고, 새롭게 authentication-database라는 항목을 추가하여 admin을 넣어주었다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/1e7dd846-06b0-4023-a4c2-5eb1405bc92f/image.png" /></p>
<p>Postman API 호출 결과 정상적으로 작동하였고, MongoDB에도 Insert가 제대로 된 모습을 확인할 수 있다.</p>