<p>동시성 문제를 해결하기 위해 데이터베이스에서 <strong>락(Lock)</strong>을 사용하는 방법은 크게 <strong>낙관적 락(Optimistic Lock)</strong>과 <strong>비관적 락(Pessimistic Lock)</strong> 두 가지로 나눠진다. 각각의 방식은 데이터 충돌을 처리하는 방식이 다르며, 사용하는 상황에 따라 성능과 효율성이 크게 달라질 수 있다.</p>
<h2 id="낙관적-락-optimistic-lock">낙관적 락 (Optimistic Lock)</h2>
<p>낙관적 락은 기본적으로 '충돌이 발생할 확률은 낮다' 라고 낙관하는 방식이다. 즉, 데이터 수정 시 충돌을 미리 방지하지 않고, 수정이 끝난 후 충돌 여부를 검사하는 방식이다. 데이터의 버전(version)을 통해 트랜잭션이 충돌하는지 여부를 체크한다.</p>
<h4 id="동작-원리">동작 원리</h4>
<ol>
<li>데이터를 조회할 때 <strong>버전 번호</strong>를 함께 가져온다.</li>
<li>데이터를 수정하려면 DB에서 가져온 <strong>버전 번호</strong>를 확인하고, <strong>변경된 버전 번호</strong>와 일치하는지 비교한다.</li>
<li>버전 번호가 일치하면 <strong>수정</strong>을 진행하고, 다르면 <strong>충돌</strong>이 발생한 것으로 간주하여 예외를 던진다.</li>
</ol>
<p>ex) 게시판의 조회수 증가, 사용자 정보 수정 등 <strong>동시에 수정이 거의 발생하지 않는 경우</strong></p>
<hr />
<h2 id="비관적-락-pessimistic-lock">비관적 락 (Pessimistic Lock)</h2>
<p>비관적 락은 '충돌이 발생할 확률이 높다' 라고 비관하고, 데이터를 수정하기 전에 잠금을 걸어 다른 트랜잭션이 접근하지 못하도록 하는 방식이다. 즉, 데이터를 조회하는 순간부터 잠금 상태로 유지하여 다른 트랜잭션의 접근을 차단한다.</p>
<h4 id="동작-원리-1">동작 원리</h4>
<ol>
<li>데이터를 조회하면서 <strong>즉시 잠금</strong>을 걸고, 다른 트랜잭션은 이 데이터를 수정할 수 없다.</li>
<li>수정이 완료될 때까지 다른 트랜잭션은 해당 데이터를 읽거나 수정할 수 없다.</li>
<li>작업이 끝나면 <strong>잠금을 해제</strong>한다.</li>
</ol>
<p>ex) 재고 관리 시스템, 은행 계좌 이체 등 <strong>동시에 데이터 수정이 자주 발생하는 경우</strong></p>
<h3 id="낙관적-락-vs-비관적-락">낙관적 락 vs 비관적 락</h3>
<table>
<thead>
<tr>
<th>특징</th>
<th>낙관적 락 (Optimistic Lock)</th>
<th>비관적 락 (Pessimistic Lock)</th>
</tr>
</thead>
<tbody><tr>
<td>접근 방식</td>
<td>충돌이 없을 거라고 가정하고 진행</td>
<td>충돌 가능성이 높으므로 미리 잠금</td>
</tr>
<tr>
<td>동작 원리</td>
<td>버전 필드 비교로 충돌 감지</td>
<td>데이터 조회 시 바로 DB에 잠금 설정</td>
</tr>
<tr>
<td>성능</td>
<td>빠름 (락을 걸지 않음)</td>
<td>느림 (락을 걸고 해제까지 대기)</td>
</tr>
<tr>
<td>충돌 발생 시</td>
<td>예외(OptimisticLockException) 발생</td>
<td>대기(Blocking) 후 처리</td>
</tr>
<tr>
<td>사용 예시</td>
<td>읽기 중심, 충돌이 드문 시스템 (게시판 조회수)</td>
<td>쓰기 중심, 충돌 위험 높은 시스템 (재고, 결제)</td>
</tr>
<tr>
<td>구현 방식 (JPA)</td>
<td><code>@Version</code></td>
<td><code>@Lock(LockModeType.PESSIMISTIC_WRITE)</code></td>
</tr>
</tbody></table>
<hr />
<ul>
<li><strong>낙관적 락</strong>은 충돌이 드물고 성능이 중요한 경우에 유리하며, <strong>비관적 락</strong>은 데이터 정합성이 중요한 상황에서 유리하다.</li>
<li>시스템의 <strong>동시성</strong>과 <strong>충돌 가능성</strong>에 따라 적절한 락 방식을 선택하는 것이 중요 !!</li>
</ul>