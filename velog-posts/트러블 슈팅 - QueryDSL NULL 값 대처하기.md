<p>QueryDSL을 이용해서 다른 테이블과 조인을 하고 파라미터로 특정 값을 받아와 where 구문으로 필터링하여 결과를 조회하는 코드를 작성했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/a9baecf9-1cd7-4e71-be82-6ef921af8f8c/image.png" /></p>
<h2 id="문제-발생">문제 발생</h2>
<p>파라미터로 받아오는 userId, itemId가 모두 값이 있을 경우는 정상적으로 예약 리스트를 받아올수 있었지만, 둘 중 하나라도 값이 null이거나 둘 다 null이라면 예외를 출력하고 있다.</p>
<ul>
<li>파라미터의 값이 모두 있는 경우</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/066953a6-4e28-484a-b3a1-e1dd6b96f53f/image.png" /></p>
<ul>
<li>파라미터 1개만 값이 있거나 둘 다 없는 경우</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/238c4ec8-4c7e-44e6-8515-bcb6c5f99f66/image.png" /></p>
<h2 id="원인-추론">원인 추론</h2>
<p>파라미터로 전달 받는 userId와 itemId가 null이므로 디폴트로 임의의 값을 넣어주면 괜찮지 않을까 생각했다.하지만 userId와 itemId를 임의로 넣어주게되면 다른 데이터를 불러오게 될 수도 있으니 null체크를 해서 예외처리를 해주고 다시 실행해보았다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/0572bc92-7ae0-4e16-ada9-ad6113bc7e75/image.png" /></p>
<p>예약 목록을 조회할 수 없다고 예외를 출력해주고 있지만, 원래 구현하려 했던 기능은 파라미터가 없으면 필터링 없이 모든 목록이 조회되도록 하는 것이기 때문에 다른 방법을 찾아 보기로했다.</p>
<h2 id="해결-방안">해결 방안</h2>
<h3 id="booleanbuilder">BooleanBuilder</h3>
<p>BooleanBuilder는 QueryDSL에서 동적 쿼리를 생성할 때 유용한 도구로, <strong>여러 조건을 동적으로 추가하거나 제거</strong>할 수 있도록 해준다.
빈 상태로 생성 가능하며, 이후 조건을 추가하면서 AND 또는 OR로 결합할 수 있다. 
조건이 null인 경우 해당 조건을 추가하지 않도록 쉽게 처리할 수 있고 복잡한 조건문을 깔끔하게 관리할 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/a4496dbf-bcba-41aa-ae86-95a53da7d2d6/image.png" /></p>
<p>BooleanBuilder를 초기화 해주고 userId와 itemId가 null 값이 아닐 때만 비교하도록 하였다. 이렇게 코드를 작성하면 파라미터 중 1개가 null 값이면 null이 아닌 파라미터 값만 넣어서 필터링을 하고 둘 다 null 값일 때는 필터링 없이 모든 예약 목록을 조회해서 가져올 수 있다. 
(BooleanBuilder를 사용하면 <strong>조건이 하나도 추가되지 않은 경우</strong> where절이 비어있게 된다. 이 경우 QueryDSL은 where절을 생략하고, <strong>필터링 없이 모든 데이터를 조회</strong>한다)</p>
<h4 id="장점">장점</h4>
<ul>
<li>조건을 null 체크 없이 안전하게 추가할 수 있다.</li>
<li>여러 조건을 누적시키며 동적으로 where절을 생성할 수 있다.</li>
</ul>
<h4 id="단점">단점</h4>
<ul>
<li>단순 조건을 추가할 때는 코드가 길어질 수 있다.</li>
</ul>
<h3 id="booleanexpression">BooleanExpression</h3>
<p>지금과 같은 경우를 해결할 수 있는 또 다른 방법으로 BooleanExpression을 활용할 수 있다.
BooleanExpression은 QueryDSL의 주요 조건 표현식 중 하나로, 단일 논리 조건을 표현하거나 다른 표현식과 결합할 수 있다. Predicate의 하위 타입으로, BooleanBuilder보다 구체적인 조건 작성에 사용된다.</p>
<pre><code class="language-java">// 기본 조건
BooleanExpression userCondition = userId != null ? reservation.user.id.eq(userId) : null;
BooleanExpression itemCondition = itemId != null ? reservation.item.id.eq(itemId) : null;

// 조건 결합
BooleanExpression finalCondition = userCondition != null &amp;&amp; itemCondition != null
        ? userCondition.and(itemCondition)
        : (userCondition != null ? userCondition : itemCondition);

// 동적 쿼리 적용
List&lt;ReservationResponseDto&gt; results = jpaQueryFactory
        .select(Projections.constructor(
                ReservationResponseDto.class,
                reservation.id,
                reservation.user.nickname,
                reservation.item.name,
                reservation.status,
                reservation.startAt,
                reservation.endAt
        ))
        .from(reservation)
        .join(reservation.user, user)
        .join(reservation.item, item)
        .where(finalCondition) // 결합된 조건 사용
        .fetch();</code></pre>
<h4 id="장점-1">장점</h4>
<ul>
<li>조건이 명확한 경우, 코드가 간결하고 가독성이 높다.</li>
<li>불필요한 객체 생성 없이 조건을 다룰 수 있다.</li>
</ul>
<h4 id="단점-1">단점</h4>
<ul>
<li>여러 조건을 동적으로 추가해야 하는 경우 복잡해질 수 있다.</li>
<li>null 처리가 필요하다. (조건을 누락하면 예외 발생 가능)</li>
</ul>
<h2 id="결과-확인">결과 확인</h2>
<ul>
<li>파라미터 중 1개만 null인 경우</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/483f4ea0-5006-4eda-8fb2-7ba109df5ca7/image.png" /></p>
<ul>
<li>파라미터 둘 다 null인 경우</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/1f259fa8-7aab-4b23-bdee-4d04f9309dba/image.png" /></p>
<p>모든 케이스에 정상적으로 데이터가 조회되는 모습을 확인할 수 있다.</p>