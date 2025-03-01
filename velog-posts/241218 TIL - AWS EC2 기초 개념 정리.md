<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/72b87a63-10bf-49bb-84f1-fd1a695e289f/image.png" /></p>
<p>오늘은 AWS EC2에 대해 공부한 내용을 정리해보려한다.
AWS EC2는 클라우드 환경에서 컴퓨팅 파워를 제공하는 서비스로, 사용자는 물리적 서버를 구매하거나 관리하지 않고도 가상 서버를 쉽게 생성하고 관리할 수 있다.</p>
<h1 id="1-ec2elastic-compute-cloud">1. EC2(Elastic Compute Cloud)</h1>
<ul>
<li>AWS의 핵심 서비스로, 클라우드 환경에서 확장 가능한 가상 서버를 제공한다.</li>
<li>필요한 시점에 서버를 생성하고, 사용하지 않을 때는 종료하여 비용을 절약할 수 있다.</li>
<li>다양한 인스턴스 유형이 제공되며, 적절한 인스턴스를 선택해 사용하면 된다.</li>
</ul>
<h1 id="2-elastic-load-balancer-elb">2. Elastic Load Balancer (ELB)</h1>
<ul>
<li><p>여러 EC2 인스턴스에 트래픽을 자동으로 분산하는 서비스이다.</p>
</li>
<li><p>주요 역할</p>
<ul>
<li>서버 과부하 방지</li>
<li>가용성 및 확장성 향상</li>
</ul>
</li>
<li><p>유형</p>
<ul>
<li>Application Load Balancer : HTTP/HTTPS 트래픽에 최적화</li>
<li>Network Load Balancer : 초저지연 트래픽 처리</li>
<li>Gateway Load Balancer : 네트워크 보안과 모니터링</li>
</ul>
</li>
</ul>
<h1 id="3-elastic-block-store-ebs">3. Elastic Block Store (EBS)</h1>
<ul>
<li>EC2 인스턴스를 위한 스토리지 서비스로, 디스크 볼륨을 제공한다.</li>
<li>EC2 인스턴스를 중지하거나 종료하더라도 데이터가 저장된다 (백업)</li>
<li>주요 특징 :<ul>
<li>여러 볼륨 크기 및 성능 옵션 제공</li>
<li>스냅샷을 통해 데이터 백업 가능</li>
</ul>
</li>
</ul>
<h1 id="4-ami-amazon-machine-image">4. AMI (Amazon Machine Image)</h1>
<ul>
<li>EC2 인스턴스를 생성하기 위한 템플릿이다.</li>
<li>AMI에는 운영체제, 애플리케이션, 설정 등이 포함된다.</li>
<li>필요한 소프트웨어를 미리 설정한 AMI를 사용하면 EC2 인스턴스를 빠르게 배포할 수 있다.</li>
</ul>
<h1 id="5-보안-그룹-security-group">5. 보안 그룹 (Security Group)</h1>
<ul>
<li>EC2 인스턴스에 대한 네트워크 트래픽을 제어하는 방화벽 역할을 한다.</li>
<li>주요 특징 :<ul>
<li>인바운드 및 아웃바운드 트래픽 규칙 설정 가능</li>
<li>특정 IP나 포트만 허용하여 보안을 강화</li>
</ul>
</li>
</ul>
<h1 id="6-탄력적-ip-elastic-ip">6. 탄력적 IP (Elastic IP)</h1>
<ul>
<li>고정된 퍼블릭 IP 주소를 EC2 인스턴스에 연결할 수 있는 기능.</li>
<li>EC2 인스턴스를 재시작하거나 종료해도 IP 주소를 유지해야 하는 경우 유용하다.</li>
<li>*<em>사용하지 않는 탄력적 IP는 추가 비용이 발생할 수 있으니 주의하자 ! *</em></li>
</ul>