<h2 id="kafka란">Kafka란?</h2>
<p>Apache Kafka는 <strong>분산 스트리밍 플랫폼</strong>으로 대용량의 데이터를 빠르고 안정적으로 처리할 수 있는 메시지 브로커 시스템이다. 기본적인 구성 요소는 다음과 같다.</p>
<ul>
<li>Producer : 메시지를 발행하는 주체</li>
<li>Broker : 메시지를 저장하고 관리하는 Kafka 서버</li>
<li>Topic : 메시지를 분류하는 단위</li>
<li>Consumer : 메시지를 구독하고 처리하는 주체</li>
</ul>
<p>Kafka는 일반적인 메시지 큐 시스템보다 <strong>높은 처리량, 내구성, 확정성</strong>을 제공하여 마이크로서비스 간의 통신이나 실시간 데이터 처리에 매우 유용하다.</p>
<h2 id="왜-kafka를-채팅-시스템에-도입">왜 Kafka를 채팅 시스템에 도입?</h2>
<p>기존의 WebSocket 기반 채팅 시스템에서는 <strong>WebSocket 세션 간 직접 메시지를 전달</strong>하는 방식으로 구성하는 경우가 많다. 이 경우 다음과 같은 문제점이 발생할 수 있다.</p>
<ul>
<li><strong>다중 서버 환경에서 세션 공유의 어려움</strong></li>
<li><strong>실시간 처리 외의 기능(메시지 저장, 분석, 알림 발송 등) 확장이 어려움</strong></li>
</ul>
<p>Kafka를 도입하면 메시지를 중간에 Kafka로 보내고 Consumer에서 받아 WebSocket을 통해 다시 전달하는 방식으로 아키텍처가 바뀐다. 이를 통해 다음과 같은 이점을 얻을 수 있다.</p>
<ul>
<li><strong>메시지 발행과 소비를 분리하여 확장성 향상</strong></li>
<li><strong>다양한 서비스가 동일 메시지를 구독 가능 (알림, 저장, 분석)</strong></li>
<li><strong>서버 간 세션 공유가 불필요 (stateless)</strong></li>
</ul>
<h2 id="구현-구조-요약">구현 구조 요약</h2>
<ul>
<li>클라이언트는 WebSocket으로 연결하며 token과 receiverId를 쿼리 파라미터로 보낸다.</li>
<li>서버는 JWT로 인증된 후 세션을 관리하며 과거 메시지를 불러온다.</li>
<li>클라이언트에서 메시지를 보내면 서버를 이를 Kafka Producer를 통해 전송한다.</li>
<li>Kafka Consumer가 메시지를 수신하고 수신자에게 해당 메시지를 WebSocket으로 전송한다.</li>
</ul>
<h4 id="websocket-handler-예제-코드">WebSocket Handler 예제 코드</h4>
<pre><code class="language-java">@Override
protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
    String senderId = (String) session.getAttributes().get(&quot;userId&quot;);
    String receiverId = (String) session.getAttributes().get(&quot;receiverId&quot;);

    ChatMessageRequestDto requestDto = new ChatMessageRequestDto(message.getPayload());
    ChatMessageResponseDto savedMessage = chatMessageService.saveMessage(
        Long.parseLong(receiverId),
        Long.parseLong(senderId),
        requestDto
    );

    String messageJson = objectMapper.writeValueAsString(savedMessage);
    kafkaProducerService.sendMessage(messageJson); // Kafka로 메시지 전송
}</code></pre>
<h3 id="문제-상황과-해결">문제 상황과 해결</h3>
<h4 id="문제-1--채팅방에-관계없는-메시지가-전달됨">문제 1 : 채팅방에 관계없는 메시지가 전달됨</h4>
<ul>
<li>유저 1과 채팅 중인데 유저 2가 보낸 메시지가 실시간으로 나타남</li>
<li>원인 : 메시지를 수신할 때 receiverId만 비교하고 roomId에 대한 필터링이 없음</li>
<li>해결 : WebSocketSession에 roomId를 저장하고, Kafka Consumer에서 메시지의 roomId와 세션의 roomId를 비교해 일치할 때만 전송하도록 수정</li>
</ul>
<h4 id="문제-2--메시지가-중복-전송됨">문제 2 : 메시지가 중복 전송됨</h4>
<ul>
<li>Kafka에서 메시지를 수신할 때 같은 메시지가 2번 표시되는 현상 발생</li>
<li>원인 : 메시지를 전송하는 로직이 이중으로 실행되거나, WebSocket 세션이 여러 개 생성된 경우</li>
<li>해결 : SessionManager를 통해 중복 세션을 방지하고, 메시지 전송 전에 중복 여부 확인</li>
</ul>