<h2 id="pod-오토스케일링-hpa-horizontal-pod-autoscaler">Pod 오토스케일링: HPA (Horizontal Pod Autoscaler)</h2>
<p><strong>HPA</strong>는 애플리케이션의 리소스 사용량(CPU, 메모리 등)을 기준으로 <strong>Pod 수를 자동으로 조절하는 오브젝트이다.</strong>
일반적으로 Deployment, ReplicaSet, StatefulSet에 적용 가능하다.
리소스 사용량을 기반으로, 정의된 <strong>임계치를 넘기면 Pod의 수를 늘리고, 사용량이 줄어들면 다시 줄인다.</strong></p>
<h3 id="hpa-예제-코드">HPA 예제 코드</h3>
<pre><code class="language-yaml">apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: example-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50</code></pre>
<ul>
<li>kubectl top pod으로 리소스 사용량 확인 가능</li>
<li>metrics-server 설치가 필요하다 (클러스터에 리소스 사용량 데이터를 제공)</li>
</ul>
<hr />
<h2 id="다양한-배포-전략">다양한 배포 전략</h2>
<ul>
<li><strong>Rolling Update</strong><ul>
<li>쿠버네티스의 기본 배포 전략</li>
<li>점진적으로 새로운 Pod를 생성하고, 이전 버전을 하나씩 종료함</li>
<li>다운타임 없이 배포 가능</li>
</ul>
</li>
<li><strong>Blue/Green Deployment</strong><ul>
<li>기존 버전(Blue)과 새로운 버전(Green)을 동시에 유지하다가, 트래픽을 전환하는 방식</li>
<li>트래픽 전환이 완전히 이뤄진 후, 기존 버전을 제거</li>
<li>롤백이 쉽고 안정성이 높다</li>
<li>쿠버네티스에서는 서비스 객체를 이용해 label selector를 변경하여 트래픽을 전환</li>
</ul>
</li>
<li><strong>Canary Deployment</strong><ul>
<li>새로운 버전을 일부 사용자에게만 점진적으로 배포한 뒤, 이상이 없으면 전체 배포한다</li>
<li>ex) 전체 트래픽의 10%만 Canary 버전에 전달 -&gt; 이상이 없다면 점차 확대한다</li>
<li>Deployment를 복수로 관리하거나, Service 객체를 통해 레이블 조정 + 트래픽 분산을 조절함</li>
</ul>
</li>
</ul>