<p>RTMP는 &quot;Real-Time Messaging Protocol&quot;의 약자로, 오디오, 비디오, 데이터 스트리밍을 위해 <strong>Adobe Systems</strong>가 개발한 프로토콜이다. 주로 <strong>라이브 스트리밍</strong>과 <strong>VOD(Video on Demand)</strong> 서비스에서 사용된다.</p>
<h2 id="rtmp의-주요-특징">RTMP의 주요 특징</h2>
<ol>
<li><p><strong>저지연(낮은 Latency)</strong></p>
<ul>
<li>실시간 스트리밍을 위해 설계되어, 수 초 단위의 낮은 지연 시간으로 전송 가능</li>
</ul>
</li>
<li><p><strong>연결 기반 프로토콜</strong></p>
<ul>
<li><strong>TCP(Transmission Control Protocol)</strong> 위에서 동작하며, 신뢰성 있는 데이터 전송을 보장</li>
</ul>
</li>
<li><p><strong>다양한 미디어 형식 지원</strong></p>
<ul>
<li>오디오: AAC, MP3 등</li>
<li>비디오: H.264, VP6 등</li>
</ul>
</li>
<li><p><strong>Adaptive Bitrate Streaming</strong></p>
<ul>
<li>네트워크 상황에 따라 비트레이트를 조절해 끊김 없는 스트리밍 지원</li>
</ul>
</li>
</ol>
<h2 id="rtmp의-동작-방식">RTMP의 동작 방식</h2>
<ol>
<li><strong>Handshaking</strong>: 클라이언트와 서버 간 초기 연결을 설정</li>
<li><strong>Connection</strong>: 연결이 성립되면 미디어 데이터를 송수신</li>
<li><strong>Streaming</strong>: 오디오, 비디오 데이터를 패킷으로 나누어 전송</li>
</ol>
<h2 id="rtmp의-장점">RTMP의 장점</h2>
<ul>
<li><strong>저지연</strong>: 라이브 방송에서 즉각적인 피드백 가능</li>
<li><strong>확장성</strong>: 대규모 사용자 지원</li>
<li><strong>보안</strong>: RTMPS(SSL/TLS 기반 암호화)로 보안 강화</li>
</ul>
<h2 id="rtmp의-단점">RTMP의 단점</h2>
<ul>
<li><strong>HTML5 비호환성</strong>: 현대 브라우저에서 <strong>RTMP</strong>는 직접 지원되지 않음</li>
<li><strong>높은 서버 요구 사항</strong>: 지속적인 연결 유지로 인한 리소스 부담</li>
</ul>
<h2 id="rtmp와-대안-프로토콜-비교">RTMP와 대안 프로토콜 비교</h2>
<table>
<thead>
<tr>
<th>프로토콜</th>
<th>지연 시간</th>
<th>보안성</th>
<th>주요 사용 사례</th>
</tr>
</thead>
<tbody><tr>
<td><strong>RTMP</strong></td>
<td>낮음(2~5초)</td>
<td>RTMPS 지원</td>
<td>라이브 스트리밍</td>
</tr>
<tr>
<td><strong>HLS</strong></td>
<td>높음(10~30초)</td>
<td>기본 지원</td>
<td>VOD, HTML5 기반 스트리밍</td>
</tr>
<tr>
<td><strong>WebRTC</strong></td>
<td>매우 낮음(1초 이하)</td>
<td>기본 지원</td>
<td>양방향 영상 통화, 회의</td>
</tr>
</tbody></table>
<hr />
<p>오늘은 <strong>RTMP(Real-Time Messaging Protocol)</strong>에 대해 공부했다. RTMP는 <strong>저지연</strong>과 <strong>실시간 스트리밍</strong>에 특화된 프로토콜로, 현재 YouTube Live와 같은 주요 스트리밍 플랫폼에서 사용된다.</p>
<p>하지만 HTML5 비호환성과 서버 부하 문제로 최근에는 <strong>HLS</strong>와 <strong>WebRTC</strong>와 같은 대안 프로토콜도 많이 활용된다.</p>
<p><strong>배운 점:</strong></p>
<ol>
<li>RTMP는 <strong>TCP 기반</strong>으로 신뢰성 있는 데이터 전송을 보장한다.</li>
<li><strong>RTMPS</strong>를 통해 보안을 강화할 수 있다.</li>
<li>현대 브라우저에서의 비호환성 때문에 WebRTC나 HLS로의 전환이 필요할 수 있다.</li>
</ol>
<p>다음에는 <strong>RTMP를 활용한 직접 라이브 스트리밍 구현</strong>을 해봐야겠다!</p>