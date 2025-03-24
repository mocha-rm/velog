<h2 id="nodegroup">NodeGroup</h2>
<ul>
<li>노드 그룹은 쿠버네티스 클러스터에서 노드들을 그룹화하여 관리하는 개념이다.</li>
<li>주로 클라우드 환경(AWS, GCP, Azure)에서 노드 풀(Node Pool)형태로 제공된다.</li>
<li>동일한 설정을 가진 여러 개의 노드를 그룹으로 묶어 자동 확장 및 배포 전략을 효율적으로 관리할 수 있다.</li>
</ul>
<h3 id="활용">활용</h3>
<ul>
<li>AWS EKS : Managed Node Group을 사용하여 특정 인스턴스 타입, AMI, Auto Scaling 정책을 적용 가능</li>
<li>GKE : Node Pool을 사용하여 서로 다른 VM 사양의 노드를 운영 가능</li>
<li>On-Premise : 직접 노드들을 그룹화하여 특정 워크로드를 할당</li>
</ul>
<hr />
<h2 id="namespace">Namespace</h2>
<ul>
<li>쿠버네티스에서 리소스를 논리적으로 분리하는 데 사용되는 가상 클러스터 개념</li>
<li>동일한 클러스터 내에서도 여러 팀 또는 프로젝트를 격리하여 운영할 수 있음</li>
<li>기본적으로 default, kube-system, kube-public 네임스페이스가 제공됨</li>
</ul>
<h3 id="활용-1">활용</h3>
<ul>
<li>서로 다른 프로젝트 또는 팀을 위한 네임스페이스 분리 (dev, staging, prod)</li>
<li>네임스페이스 기반 리소스 할당 및 권한 제어</li>
<li>모니터링 및 네트워크 정책 적용 시 유용</li>
</ul>
<h4 id="명령어-예시">명령어 예시</h4>
<pre><code class="language-text"># 네임스페이스 목록 조회
kubectl get namespaces

# 새로운 네임스페이스 생성
kubectl create namespace my-namespace

# 특정 네임스페이스에서 리소스 조회
kubectl get pods -n my-namespace</code></pre>
<hr />
<h2 id="configmap">ConfigMap</h2>
<ul>
<li>애플리케이션이 사용하는 환경 설정 데이터를 Key-Value 형태로 저장하는 리소스</li>
<li>일반적으로 환경 변수, 설정 파일 등을 저장하는 용도로 활용됨</li>
<li>Secret과 달리 민감하지 않은 데이터를 저장한다.</li>
</ul>
<h3 id="활용-2">활용</h3>
<ul>
<li>애플리케이션의 환경 변수 설정</li>
<li>ConfigMap을 마운트하여 설정 파일을 컨테이너에 주입</li>
<li>여러 파드에서 공통 설정을 공유</li>
</ul>
<h4 id="예시">예시</h4>
<pre><code class="language-yaml"># ConfigMap 생성
kubectl create configmap app-config --from-literal=APP_ENV=production

# ConfigMap 목록 조회
kubectl get configmap

# ConfigMap을 사용하는 Pod 예제
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo
spec:
  containers:
  - name: app
    image: my-app
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV</code></pre>
<hr />
<h2 id="secret">Secret</h2>
<ul>
<li>민감한 데이터를 저장하고 관리하는 리소스</li>
<li>기본적으로 Base64 인코딩되어 저장되지만, 보안 강화를 위해 외부 KMS(Key Management Service)와 연동 가능</li>
</ul>
<h3 id="활용-3">활용</h3>
<ul>
<li>데이터베이스 접속 정보 저장</li>
<li>API 키 또는 인증서 관리</li>
<li>볼륨 마운트 또는 환경 변수로 주입하여 사용</li>
</ul>
<h4 id="예시-1">예시</h4>
<pre><code class="language-yaml"># Secret 생성 (Base64 인코딩된 데이터 포함)
echo -n &quot;mypassword&quot; | base64
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=mypassword

# Secret 목록 조회
kubectl get secrets

# Secret을 사용하는 Pod 예제
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: app
    image: my-app
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password</code></pre>