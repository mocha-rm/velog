<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/094a0739-0e81-4e6b-bda4-b1d8942a5f5a/image.png" /></p>
<p>최근에 Mini PC를 구매해서 직접 Ubuntu를 설치하고, 기존에 Railway에서 운영하던 서비스를 Mini PC로 마이그레이션하는 작업을 진행했다.
처음엔 단순히 “클라우드 비용을 아껴보자”는 생각이었는데, 실제로 직접 서버를 구성해보니 예상보다 훨씬 많은 걸 배우고 느낄 수 있었다.</p>
<hr />
<h2 id="1-실제-서버-환경-구성--클라우드가-자동으로-해주던-것들">1. 실제 서버 환경 구성 — 클라우드가 자동으로 해주던 것들</h2>
<p>Railway 같은 클라우드 서비스에서는 배포 버튼 몇 번만 누르면 모든 게 자동으로 된다.
하지만 Mini PC 환경에서는 기본적인 네트워크와 서버 설정을 모두 직접 해야 한다.</p>
<h4 id="포트포워딩-설정">포트포워딩 설정</h4>
<p>라우터에서 외부 요청이 Mini PC까지 올 수 있도록 포트포워딩을 수동으로 설정했다.
예를 들어, HTTP(80), HTTPS(443), API용 8080 포트를 열어서 내 내부 IP로 포워딩:</p>
<pre><code class="language-bash"># 예시 (라우터 설정 화면에서)
외부포트: 80  → 내부 IP: 192.168.0.20  내부포트: 80  (TCP)
외부포트: 443 → 내부 IP: 192.168.0.20  내부포트: 443 (TCP)
외부포트: 8080 → 내부 IP: 192.168.0.20 내부포트: 8080 (TCP)</code></pre>
<p>라우터마다 UI가 조금씩 다르긴 하지만, 원리는 동일하다.</p>
<hr />
<h3 id="ufw-방화벽-설정">UFW 방화벽 설정</h3>
<p>Ubuntu 기본 방화벽인 UFW(Uncomplicated Firewall)를 활용해서 불필요한 포트는 차단하고, 필요한 포트만 열었다.</p>
<pre><code class="language-bash"># 방화벽 활성화
sudo ufw enable

# 기본 정책: 모든 외부 접근 차단
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 필요한 포트만 열기
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 8080/tcp  # API 서버</code></pre>
<p>이 과정을 거치니, “클라우드에서 왜 기본적으로 보안 설정이 잘 돼 있는지” 체감할 수 있었다.
직접 하면 생각보다 귀찮고, 설정 실수 하나로 바로 접속이 막히거나 공격 대상이 되기도 한다.</p>
<hr />
<h3 id="도메인--ssl-인증서-적용">도메인 &amp; SSL 인증서 적용</h3>
<p>무료 SSL 인증서를 발급해주는 Let’s Encrypt와 Nginx를 활용해서 HTTPS를 적용했다.</p>
<pre><code class="language-bash">sudo apt update
sudo apt install nginx certbot python3-certbot-nginx

# Nginx 서버블록 설정 (예: /etc/nginx/sites-available/myservice)
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:8080;
    }
}

# 심볼릭 링크로 활성화
sudo ln -s /etc/nginx/sites-available/myservice /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# SSL 인증서 발급
sudo certbot --nginx -d example.com</code></pre>
<p>이제 브라우저에서 <a href="https://example.com%EC%9C%BC%EB%A1%9C">https://example.com으로</a> 접속하면 인증서가 적용된 안전한 서비스에 접근할 수 있게 됐다.</p>
<hr />
<h3 id="서비스-자동-실행-설정-systemd">서비스 자동 실행 설정 (systemd)</h3>
<p>서버가 재부팅되거나 장애가 발생했을 때 자동으로 백엔드 애플리케이션이 재시작되도록 systemd 설정도 추가했다.</p>
<pre><code class="language-bash">sudo nano /etc/systemd/system/myservice.service</code></pre>
<pre><code class="language-ini">[Unit]
Description=My Spring Boot Service
After=network.target

[Service]
User=ubuntu
ExecStart=/usr/bin/java -jar /home/ubuntu/app/myservice.jar
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target</code></pre>
<pre><code class="language-bash">sudo systemctl daemon-reload
sudo systemctl enable myservice
sudo systemctl start myservice</code></pre>
<p>이제 서버가 꺼졌다 켜져도 자동으로 애플리케이션이 실행된다.</p>
<hr />
<h2 id="2-클라우드가-그만큼의-비용을-받는-이유를-실감하다">2. 클라우드가 “그만큼의 비용”을 받는 이유를 실감하다</h2>
<p>Railway, Vercel, AWS, GCP 등은 단순히 서버만 빌려주는 게 아니다.
실제로 해보면, 그들이 인프라를 얼마나 자동화하고 안정적으로 제공하는지가 피부로 느껴진다.
    • 자동 백업 / 복구
    • 장애 대비 고가용성 구성 (HA)
    • 로드밸런싱 및 트래픽 분산
    • 모니터링, 로그 수집, 알림 시스템</p>
<p>이런 걸 혼자서 Mini PC 환경에 구성하려면 생각보다 훨씬 많은 시간, 경험, 그리고 리소스가 필요하다.
그래서 규모가 커지면 결국 클라우드 서비스로 이전하는 게 정답이라는 걸 깨달았다.</p>
<hr />
<h2 id="3-mini-pc의-역할--학습용-소규모-서비스에-딱이다">3. Mini PC의 역할 — 학습용, 소규모 서비스에 딱이다</h2>
<p>그렇다고 Mini PC가 의미 없는 건 아니다. 오히려 공부용, 실험용으로는 정말 훌륭하다.
    •    개인 블로그나 소규모 API 서버 운영 가능
    •    Docker, Nginx, CI/CD 등 실습에 최적
    •    인프라 전반을 스스로 설정해보며 실력을 끌어올릴 수 있음
    •    비용 부담 없이 자유롭게 테스트 가능</p>
<p>실제로 이번 마이그레이션 경험은 단순히 서버 이전이 아니라, 인프라의 기본 원리를 체득하는 값진 시간이었다.</p>
<hr />
<h2 id="마무리">마무리</h2>
<p>👉 정리하자면,
    • 공부/소규모 서비스에는 Mini PC로도 충분하다.
    • 실서비스/대규모 트래픽은 클라우드의 안정성과 자동화가 필요하다.
    • 무엇보다, 실제로 직접 설정해보는 경험이 개발자로서의 역량을 크게 키운다.</p>
<p>이번 경험을 통해 클라우드의 편리함이 단순한 “편의” 수준이 아니라는 걸 확실히 느꼈고, 동시에 Mini PC 환경에서 자유롭게 실험해볼 수 있는 즐거움도 컸다.
앞으로도 이런 식의 실험을 계속해 나갈 생각이다. 💪</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/687b3137-a120-46dd-b754-a85f8e05d16d/image.png" /></p>