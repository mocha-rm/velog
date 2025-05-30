<h3 id="문제-발생">문제 발생</h3>
<p>로컬에서는 정상 작동하던 크론잡이 EC2 서버에서는 정상적으로 실행되지 않았다.</p>
<h3 id="원인-분석">원인 분석</h3>
<p>크론잡(@Scheduled)은 <strong>서버 시간대(Timezone)를 기준으로</strong> 작동한다.
로컬은 'Asia/Seoul', EC2는 'UTC' 시간대를 사용하고 있었다. 즉, 크론 표현식 기준 시간이 달라져 실행되지 않은 것처럼 보인 것이다.</p>
<h3 id="해결-방법">해결 방법</h3>
<ol>
<li><p><strong>EC2 인스턴스의 시간대 설정</strong>
EC2 인스턴스에 접속하여 시간대 설정이 어떻게 되어있는지 확인해본다. ('<strong>timedatectl</strong>' 명령어로 가능)
<img alt="" src="https://velog.velcdn.com/images/jelog_131/post/50cf6a9f-705c-4d54-885c-7900e5fb6862/image.png" />
현재 'UTC' 시간대로 설정되어있는 모습을 확인할 수 있었다.
'<strong>sudo timedatectl set-timezone Asia/Seoul</strong>' 명령어로 시간대를 현지 지역과 시간에 맞게 변경해주었다.
<img alt="" src="https://velog.velcdn.com/images/jelog_131/post/a7c2fd4e-074d-4471-be00-312d02e69e32/image.png" /></p>
</li>
<li><p><strong>Docker 컨테이너의 시간대를 호스트와 동기화</strong>
docker-compose.yml 스크립트를 수정하여 컨테이너가 EC2의 시간대와 동일하게 작동하도록 설정하였다.</p>
<pre><code class="language-yaml">services:
app:
 ...
 volumes:
   - /etc/localtime:/etc/localtime:ro</code></pre>
</li>
<li><p><strong>크론잡에 시간대 명시</strong>
코드상에 명시적으로 타임존을 지정하므로서 코드만 봐도 유지보수 시 혼란을 줄일 수 있다.</p>
</li>
</ol>
<pre><code class="language-java">@Scheduled(cron = &quot;0 0 8 * * ?&quot;, zone = &quot;Asia/Seoul&quot;)
    public void sendDailyEmailToSubscribers() {
        log.info(&quot;일일 이메일 발송 작업 시작: {}&quot;, LocalDateTime.now());

        List&lt;Subscriber&gt; subscribers = subscribeService.getAllSubscribers();

        if (subscribers.isEmpty()) {
            log.info(&quot;구독자가 없습니다. 이메일 발송을 건너뜁니다.&quot;);
            return;
        }

        String subject = &quot;마이니치 니홍고 - 오늘의 일본어 학습&quot;;
        String content = contentService.generateDailyContent();

        int successCount = 0;
        int failCount = 0;

        for (Subscriber subscriber : subscribers) {
            boolean sent = emailService.sendEmail(subscriber.getEmail(), subject, content);

            if (sent) {
                successCount++;
            } else {
                failCount++;
            }
        }

        log.info(&quot;일일 이메일 발송 완료 - 성공: {}, 실패: {}&quot;, successCount, failCount);
    }</code></pre>
<p>결국 이번 문제는 코드상에 문제가 있다기 보다는 <strong>배포 환경의 시간대 설정</strong>이 문제였다.
크론잡을 사용할 때는 항상 <strong>시간대를 명시하거나 서버의 시간대</strong>를 확인하자</p>