<p><a href="https://velog.io/@jelog_131/%ED%8A%B8%EB%9F%AC%EB%B8%94-%EC%8A%88%ED%8C%85-QueryDSL-NULL-%EA%B0%92-%EB%8C%80%EC%B2%98%ED%95%98%EA%B8%B0">트러블 슈팅 - QueryDSL NULL 값 대처하기</a></p>
<p>저번 트러블 슈팅에서 이어지는 내용이라 위쪽에 링크를 첨부했다.</p>
<h1 id="문제-발생">문제 발생</h1>
<p>BooleanBuilder를 이용해 파라미터에 null 값이 들어와도 정상적으로 예약 목록을 조회하도록 구현했지만, 파라미터 2개가 모두 없다면 전체 목록을 조회해오는데 이 부분에서 N+1 문제가 발생했다.</p>
<pre><code class="language-java">@Override
    public List&lt;Reservation&gt; searchReservations(Long userId, Long itemId) {
        BooleanBuilder predicate = new BooleanBuilder();

        if (userId != null) {
            predicate.and(reservation.user.id.eq(userId));
        }

        if (itemId != null) {
            predicate.and(reservation.item.id.eq(itemId));
        }

        return jpaQueryFactory.selectFrom(reservation)
                .join(reservation.user, user).fetchJoin()
                .join(reservation.item, item).fetchJoin()
                .where(predicate)
                .fetch();
    }</code></pre>
<h1 id="원인-추론">원인 추론</h1>
<p>파라미터가 모두 null일 때, Hibernate가 추가적으로 user 테이블을 조회하는 이유는 fetchJoin을 사용했더라도 실제로 데이터를 필요로 하면 Lazy Loading이 발생하기 때문이다. 
(연관관계를 설정할 때 fetchType을 Lazy로 설정했기 때문)
이 문제를 해결하려면 파라미터가 둘 다 null인 경우에 대해 별도로 처리를 해줘야 한다.</p>
<h1 id="해결-방법">해결 방법</h1>
<p>Reservation 엔티티에서 필요한 필드만 반환하도록 DTO를 사용했다. 
이 방법은 모든 데이터를 가져오는 대신, 특정 필드만 조회하여 추가적인 Lazy Loading을 방지한다.</p>
<pre><code class="language-java">@Override
    public List&lt;ReservationResponseDto&gt; searchReservations(Long userId, Long itemId) {
        BooleanBuilder predicate = new BooleanBuilder();

        if (userId != null) {
            predicate.and(reservation.user.id.eq(userId));
        }
        if (itemId != null) {
            predicate.and(reservation.item.id.eq(itemId));
        }


        return jpaQueryFactory
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
                .where(predicate)
                .fetch();
    }</code></pre>
<h1 id="결과-확인">결과 확인</h1>
<ul>
<li>파라미터 2개 모두 값이 있는 경우</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/ab5623bb-6bf1-48dd-9a3b-d50f7d51ef4c/image.png" /></p>
<ul>
<li>쿼리</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/851a2e7e-5938-4050-8dea-15e613d3111b/image.png" /></p>
<hr />
<ul>
<li>파라미터 1개만 값이 있는 경우</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/5f6681d2-1838-4b8c-a9c2-62e394c93e56/image.png" /></p>
<ul>
<li>쿼리</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/e72afa55-620b-49ab-93ab-21c2fb3d10e8/image.png" /></p>
<hr />
<ul>
<li>파라미터 2개다 null값인 경우</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/710339e8-7b91-4460-aef7-a83a490c43a2/image.png" /></p>
<ul>
<li>쿼리</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/3d07168c-55ce-48ff-b4f0-a0b7af29d8a0/image.png" /></p>
<p>N+1 문제 없이 모든 케이스에서 정상적으로 작동하는 모습을 확인할 수 있었다 ! </p>