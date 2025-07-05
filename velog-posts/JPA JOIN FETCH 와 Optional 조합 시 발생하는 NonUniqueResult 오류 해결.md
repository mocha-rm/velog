<h2 id="문제-상황">문제 상황</h2>
<p>Spring Boot 프로젝트에서 날짜별 콘텐츠를 조회하는 기능을 구현하던 중, 다음과 같은 오류가 발생했다.</p>
<pre><code class="language-text">Query did not return a unique result: 2 results were returned</code></pre>
<p>처음에는 동일한 날짜에 여러 개의 레코드가 존재하는 것으로 생각했다. 하지만 MySQL에서 직접 쿼리를 실행해보니 예상과 달랐다.</p>
<h3 id="문제가-된-코드">문제가 된 코드</h3>
<h4 id="repository">Repository</h4>
<pre><code class="language-java">@Query(&quot;SELECT e FROM EmailContent e JOIN FETCH e.theme WHERE e.createdAt BETWEEN ?1 AND ?2&quot;)
Optional&lt;EmailContent&gt; findByCreatedDateBetween(LocalDateTime startDate, LocalDateTime endDate);</code></pre>
<h4 id="controller">Controller</h4>
<pre><code class="language-java">LocalDate targetDate = LocalDate.parse(date, DateTimeFormatter.ofPattern(&quot;yyyyMMdd&quot;));
LocalDateTime startOfDay = targetDate.atStartOfDay();        // 2025-07-05T00:00:00
LocalDateTime endOfDay = targetDate.atTime(23, 59, 59);     // 2025-07-05T23:59:59

Optional&lt;EmailContent&gt; content = emailContentRepository.findByCreatedDateBetween(startOfDay, endOfDay);</code></pre>
<h3 id="mysql-직접-확인-결과">MySQL 직접 확인 결과</h3>
<pre><code class="language-sql">-- 날짜별 레코드 수 확인
SELECT DATE(created_at) as date, COUNT(*) as count 
FROM email_content 
GROUP BY DATE(created_at) 
HAVING count &gt; 1;
-- 결과: Empty set

-- 해당 날짜의 데이터 확인
SELECT count(1) 
FROM email_content e 
JOIN content_theme t ON t.id = e.theme_id 
WHERE e.created_at BETWEEN '2025-07-05 00:00:00' AND '2025-07-05 23:59:59';
-- 결과: 1 (정상)</code></pre>
<p>MySQL에서는 1개의 결과만 나오는데, 왜 JPA에서는 2개 결과가 반환된다고 하지?</p>
<h2 id="원인-분석">원인 분석</h2>
<h3 id="jpa-join-fetch의-특성">JPA JOIN FETCH의 특성</h3>
<p>문제는 JPA의 <strong>JOIN FETCH와 Optional 조합</strong>에서 발생했다.</p>
<ul>
<li><strong>JOIN FETCH</strong>는 연관된 엔티티를 즉시 로딩하기 위한 JPA 최적화 기법</li>
<li>하지만 <strong>Optional과</strong> 함께 사용할 때, JPA 구현체 (Hibernate)의 내부 처리 과정에서 중복 결과로 인식하는 경우가 있음</li>
<li><strong>실제 DB에서는 1개 행 이지만, JPA 레벨에서는 2개로 인식</strong>하는 상황</li>
</ul>
<h3 id="hibernate-로그-분석">Hibernate 로그 분석</h3>
<pre><code class="language-sql">Hibernate: 
    select
        ec1_0.id,
        ec1_0.created_at,
        ec1_0.detailed_content,
        ec1_0.html_content,
        ec1_0.sent,
        ec1_0.sent_at,
        t1_0.id,
        t1_0.jlptlevel,
        t1_0.topic,
        t1_0.used,
        t1_0.used_at 
    from
        email_content ec1_0 
    join
        content_theme t1_0 
            on t1_0.id=ec1_0.theme_id 
    where
        ec1_0.created_at between ? and ?</code></pre>
<p>SQL 자체는 정상적이지만, JPA가 결과를 Optional로 변환하는 과정에서 문제가 발생한다.</p>
<h2 id="해결-방법">해결 방법</h2>
<h3 id="1-distinct-추가">1. DISTINCT 추가</h3>
<pre><code class="language-java">@Query(&quot;SELECT DISTINCT e FROM EmailContent e JOIN FETCH e.theme WHERE e.createdAt BETWEEN ?1 AND ?2&quot;)
Optional&lt;EmailContent&gt; findByCreatedDateBetween(LocalDateTime startDate, LocalDateTime endDate);</code></pre>
<h4 id="왜-distinct가-해결책">왜 DISTINCT가 해결책?</h4>
<ul>
<li>JPA에서 JOIN FETCH를 사용할 때 권장되는 패턴</li>
<li>중복 결과 제거로 Optional 안전하게 동작</li>
</ul>
<h3 id="2-반환-타입-변경">2. 반환 타입 변경</h3>
<pre><code class="language-java">@Query(&quot;SELECT e FROM EmailContent e JOIN FETCH e.theme WHERE e.createdAt BETWEEN ?1 AND ?2&quot;)
List&lt;EmailContent&gt; findByCreatedDateBetween(LocalDateTime startDate, LocalDateTime endDate);

// Controller에서 사용
List&lt;EmailContent&gt; contents = emailContentRepository.findByCreatedDateBetween(startOfDay, endOfDay);
Optional&lt;EmailContent&gt; content = contents.isEmpty() ? Optional.empty() : Optional.of(contents.get(0));</code></pre>
<h3 id="3-join-fetch-제거">3. JOIN FETCH 제거</h3>
<pre><code class="language-java">Optional&lt;EmailContent&gt; findByCreatedAtBetween(LocalDateTime startDate, LocalDateTime endDate);</code></pre>
<p>연관된 엔티티는 지연 로딩으로 처리하고, 필요시 별도 조회</p>
<hr />
<h2 id="til">TIL</h2>
<ol>
<li><strong>JPA와 실제 DB 동작의 차이</strong> : MySQL에서 정상 동작하는 쿼리라도 JPA 레벨에서는 다른 결과가 나올 수 있다.</li>
<li><strong>JOIN FETCH + Optional 조합 주의</strong> : 이 조합은 NonUniqueResult 오류를 발생시킬 수 있으므로 DISTINCT를 함께 사용하는 것이 안전하다.</li>
<li><strong>디버깅 접근법</strong> :<ul>
<li>먼저 DB에서 직접 쿼리 실행</li>
<li>Hibernate 로그 확인</li>
<li>JPA 내부 동작 원리 이해</li>
</ul>
</li>
<li><strong>Best Practice</strong> : JPA에서 JOIN FETCH를 사용할 때는 항상 DISTINCT를 고려하자.</li>
</ol>
<p>단순해 보이는 조회 기능이지만, JPA의 내부 동작을 이해하지 못하면 예상치 못한 오류에 직면할 수 있다.
이번 경험을 통해 <strong>ORM의 추상화 뒤에 숨겨진 복잡성</strong>을 다시 한번 깨달았다.</p>
<ul>
<li>참고 자료<ul>
<li><a href="https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#hql-explicit-join-fetch">Hibernate Documemtation - JOIN FETCH</a></li>
<li><a href="https://docs.oracle.com/javaee/7/tutorial/persistence-querylanguage.htm">JPA Query 최적화 가이드</a></li>
</ul>
</li>
</ul>