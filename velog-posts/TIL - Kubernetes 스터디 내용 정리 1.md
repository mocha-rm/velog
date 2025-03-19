<h2 id="kubernetes란">Kubernetes란?</h2>
<p>Kubernetes는 컨테이너화된 애플리케이션을 배포하고 관리하는 오케스트레이션 도구로 자동화된 배포, 확장 및 운영을 지원한다.</p>
<h2 id="핵심-개념">핵심 개념</h2>
<h3 id="node">Node</h3>
<ul>
<li>Kubernetes 클러스터를 구성하는 개별 서버</li>
<li>Master Node : 클러스터를 제어하고 관리하는 역할</li>
<li>Worker Node : 실제 애플리케이션이 실행되는 노드로, Pod를 호스팅함</li>
</ul>
<h3 id="pod">Pod</h3>
<ul>
<li>Kubernetes에서 애플리케이션이 배포되는 최소 단위</li>
<li>하나 이상의 컨테이너를 포함할 수 있으며, 같은 네트워크 네임스페이스를 공유함</li>
</ul>
<h2 id="리소스-관리---requests--limits">리소스 관리 - Requests &amp; Limits</h2>
<p>Kubernetes는 컨테이너가 사용하는 CPU와 메모리를 관리하기 위해 requests와 limits 설정을 지원한다.</p>
<h3 id="requests">Requests</h3>
<ul>
<li>컨테이너가 최소한으로 보장받을 리소스</li>
<li>스케줄러는 Node의 가용 리소스를 고려하여 Pod를 배치함</li>
</ul>
<h3 id="limits">Limits</h3>
<ul>
<li>컨테이너가 최대 사용할 수 있는 리소스</li>
<li>설정된 한도를 초과하면 컨테이너는 제한을 받거나 종료될 수 있음</li>
</ul>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: sample-replicaset
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27.0
          resources:
            requests:
              cpu: '750m' #1Core = 1000m
              memory: '1Gi' # 1024Mi
            limits:
              cpu: '750m'
              memory: '1Gi'
          ports:
            - containerPort: 80
</code></pre>