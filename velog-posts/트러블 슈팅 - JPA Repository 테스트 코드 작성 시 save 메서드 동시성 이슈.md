<h1 id="문제-발생">문제 발생</h1>
<p>UserRepository에 대한 테스트 클래스를 만들고 findByEmail() 이라는 메서드에 대한 테스트 코드를 작성했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/e1504627-2925-4304-9a58-c8bf181e77bb/image.png" /></p>
<p>미리 setUp에서 MockUser를 만들고 JPA에서 기본적으로 제공하는 repository.save() 메서드를 이용해 데이터베이스에 저장하고, findByEmail()로 유저를 잘 찾아오는지 assertThat을 이용해 검증한다. 
하지만 오류가 발생했다..</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/7bc8b763-687f-4baa-97d3-7583c3b611cd/image.png" /></p>
<blockquote>
<p>Caused by: org.hibernate.StaleObjectStateException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect): [com.example.demo.entity.User#1]</p>
</blockquote>
<p>오류의 내용을 추려서 검색해보니 StaleObjectStateException으로 , Hibernate에서 Optimistic Locking이 활성화된 상태에서 발생하는 오류라고 나왔다.</p>
<h1 id="원인-추론">원인 추론</h1>
<p>&quot;Row was updated or deleted by another transaction&quot;
한 트랜잭션이 데이터베이스에서 특정 행을 조회 후 변경 또는 삭제하려고 할 때, 그 동안 다른 트랜잭션이 같은 행을 업데이트 하거나 삭제기 때문에 충돌이 발생했다는 것이다.</p>
<p>다시 콘솔에 출력된 내용을 보니 친절하게 어디가 문제인지 알려준다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/985a32a3-0d23-45ed-883f-a775b450b214/image.png" /></p>
<p>25번째 줄에는 MockUser를 save하는 코드가 작성되어 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/ce1976b7-eb05-4a58-b924-19acfe7d435f/image.png" /></p>
<p>흠.. 뭐가 문제일까 생각하다가 User 클래스를 다시 한번 확인했고,</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/84a3fdd0-44b3-4823-804d-19fbab05a9c3/image.png" /></p>
<p>ID 생성전략이 GenerationType.IDENTITY 라는 걸 확인했다. save 메서드가 호출될 때 자동으로 Id값을 1씩 증가시키면서 데이터베이스에 저장하는데, 테스트 코드에서 MockUser를 만들 때 넣어줬던 임의 Id값이 문제가 될 수도 있겠다고 생각했다.</p>
<h1 id="해결-방법">해결 방법</h1>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/3a6ee16d-550c-4e2e-b3ee-fddc9e83613f/image.png" /></p>
<p>기존에 있던 코드를 제거하고 Id 값을 뺀 나머지 값만 초기화 시키는 생성자로 MockUser를 만들어주었다.</p>
<h1 id="결과-확인">결과 확인</h1>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/dc6a4603-898f-48be-aff2-eb517620f7b1/image.png" /></p>
<p>테스트가 성공적으로 진행된 모습과 함께 쿼리도 기대했던 대로 잘 나오는 모습을 확인 할 수 있었다 ! </p>