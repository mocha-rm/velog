<h2 id="service--pod의-네트워크-접근을-위한-추상화">Service : Pod의 네트워크 접근을 위한 추상화</h2>
<p>kubernetes에서 Pod는 동적으로 생성되고 사라지므로, 특정 Pod의 IP 주소는 고정되지 않는다. 따라서 안정적인 네트워크 통신을 위해 Service가 필요하다.</p>
<h3 id="service의-주요-타입">Service의 주요 타입</h3>
<ol>
<li>ClusterIP (기본값)<ul>
<li>클러스터 내부에서만 접근 가능한 서비스</li>
<li>kubectl expose deployment my-app --type=ClusterIP --port=80</li>
</ul>
</li>
<li>NodePort<ul>
<li>각 Node의 고정된 포트를 개방하여 외부에서 접근 가능</li>
<li>kubectl expose deployment my-app --type=NodePort --port=80</li>
</ul>
</li>
<li>LoadBalancer<ul>
<li>클라우드 제공자의 로드밸런서를 사용하여 외부 접근을 지원</li>
<li>kubectl expose deployment my-app --type=LoadBalancer --port=80</li>
</ul>
</li>
<li>ExternalName<ul>
<li>외부 서비스 도메인과 연결</li>
<li>kubectl apply -f external-service.yaml</li>
</ul>
</li>
</ol>
<hr />
<h2 id="ingress--외부-트래픽을-클러스터-내부로-라우팅">Ingress : 외부 트래픽을 클러스터 내부로 라우팅</h2>
<p>Ingress는 kubernetes에서 HTTP(S) 트래픽을 관리하는 리소스로, 로드밸런서보다 유연한 라우팅 기능을 제공한다.</p>
<h3 id="ingress-주요-개념">Ingress 주요 개념</h3>
<ul>
<li>Path-based Routing : 도메인 + 경로 기반으로 트래픽을 특정 서비스로 전달</li>
<li>Host-based Routing : 여러 도메인을 하나의 Ingress에서 처리 가능</li>
<li>TLS 지원 : HTTPS 트래픽 관리 가능</li>
</ul>
<h3 id="ingress-예제-yaml">Ingress 예제 yaml</h3>
<pre><code class="language-yaml">apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: my-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80</code></pre>
<h4 id="ingress-controller-필요">Ingress Controller 필요</h4>
<p>기본적으로 kubernetes에는 Ingress 기능만 있고, 실제 트래픽을 처리하려면 <strong>Ingress Controller(Nginx, Traefik 등)</strong>를 설치해야 한다.</p>
<hr />
<h2 id="deployment--애플리케이션-배포-및-관리">Deployment : 애플리케이션 배포 및 관리</h2>
<p>Deployment는 Pod를 생성, 업데이트, 롤백, 스케일링할 수 있도록 관리하는 리소스이다.</p>
<h3 id="deployment의-주요-기능">Deployment의 주요 기능</h3>
<ul>
<li>ReplicaSet을 통해 원하는 개수의 Pod 유지</li>
<li>새로운 버전 배포 시 Rolling Update 또는 Recreate 전략 제공</li>
<li>특정 버전으로 되돌리는 Rollback 기능 지원</li>
</ul>
<h3 id="deployment-예제-yaml">Deployment 예제 yaml</h3>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:1.0
        ports:
        - containerPort: 80</code></pre>
<ul>
<li>replicas: 3 -&gt; Pod 3개 유지</li>
<li>image: my-app:1.0 -&gt; 특정 버전 이미지 사용</li>
<li>Rolling Update로 중단 없이 새로운 버전 배포 가능</li>
</ul>
<h4 id="kubectl-명령어-정리">kubectl 명령어 정리</h4>
<ul>
<li>kubectl apply -f deployment.yaml -&gt; Deployment 생성</li>
<li>kubectl rollout status deployment my-app -&gt; 배포 상태 확인</li>
<li>kubectl rollout undo deployment my-app -&gt; 이전 버전으로 롤백</li>
</ul>