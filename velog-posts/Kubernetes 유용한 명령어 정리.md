<table>
<thead>
<tr>
<th><strong>명령어</strong></th>
<th><strong>설명</strong></th>
</tr>
</thead>
<tbody><tr>
<td><code>aws eks --region &lt;region&gt; update-kubeconfig --name &lt;cluster-name&gt;</code></td>
<td>EKS 클러스터와 로컬 kubectl을 연결</td>
</tr>
<tr>
<td><code>kubectl get nodes</code></td>
<td>클러스터의 노드 상태 확인</td>
</tr>
<tr>
<td><code>kubectl get pods --all-namespaces</code></td>
<td>모든 네임스페이스의 파드 상태 확인</td>
</tr>
<tr>
<td><code>kubectl describe pod &lt;pod-name&gt; -n &lt;namespace&gt;</code></td>
<td>특정 파드의 상태 세부 정보 확인</td>
</tr>
<tr>
<td><code>kubectl logs -f &lt;pod-name&gt; -n &lt;namespace&gt;</code></td>
<td>특정 파드의 실시간 로그 확인</td>
</tr>
<tr>
<td><code>kubectl get deployments -n &lt;namespace&gt;</code></td>
<td>특정 네임스페이스의 디플로이먼트 상태 확인</td>
</tr>
<tr>
<td><code>kubectl get svc -n &lt;namespace&gt;</code></td>
<td>특정 네임스페이스의 서비스 상태 확인</td>
</tr>
<tr>
<td><code>kubectl top nodes</code></td>
<td>노드 리소스 사용량 확인</td>
</tr>
<tr>
<td><code>kubectl top pods -n &lt;namespace&gt;</code></td>
<td>파드 리소스 사용량 확인</td>
</tr>
<tr>
<td><code>kubectl run &lt;pod-name&gt; --image=&lt;image-name&gt; -n &lt;namespace&gt;</code></td>
<td>새로운 파드 실행</td>
</tr>
<tr>
<td><code>kubectl set image deployment/&lt;deployment-name&gt; &lt;container-name&gt;=&lt;new-image&gt; -n &lt;namespace&gt;</code></td>
<td>배포된 애플리케이션 이미지 업데이트</td>
</tr>
<tr>
<td><code>kubectl rollout undo deployment/&lt;deployment-name&gt; -n &lt;namespace&gt;</code></td>
<td>배포 롤백</td>
</tr>
<tr>
<td><code>kubectl delete pod &lt;pod-name&gt; -n &lt;namespace&gt;</code></td>
<td>파드 삭제</td>
</tr>
<tr>
<td><code>kubectl delete svc &lt;service-name&gt; -n &lt;namespace&gt;</code></td>
<td>서비스 삭제</td>
</tr>
<tr>
<td><code>kubectl create namespace &lt;namespace-name&gt;</code></td>
<td>새로운 네임스페이스 생성</td>
</tr>
<tr>
<td><code>kubectl get all -n &lt;namespace&gt;</code></td>
<td>특정 네임스페이스에 대한 모든 리소스 상태 확인</td>
</tr>
<tr>
<td><code>kubectl get events -n &lt;namespace&gt;</code></td>
<td>클러스터의 이벤트 확인</td>
</tr>
<tr>
<td><code>source &lt;(kubectl completion bash)</code></td>
<td>kubectl 명령어 자동완성 설정</td>
</tr>
<tr>
<td>`curl <a href="https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3">https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3</a></td>
<td>bash`</td>
</tr>
<tr>
<td><code>helm install &lt;release-name&gt; &lt;chart-name&gt; -n &lt;namespace&gt;</code></td>
<td>Helm 차트를 이용해 애플리케이션 배포</td>
</tr>
<tr>
<td><code>aws eks --region &lt;region&gt; describe-cluster --name &lt;cluster-name&gt; --query &quot;cluster.identity.oidc.issuer&quot;</code></td>
<td>EKS에서 IAM 역할 연결</td>
</tr>
<tr>
<td><code>kubectl get endpoints -n &lt;namespace&gt;</code></td>
<td>서비스 디스커버리 확인</td>
</tr>
<tr>
<td><code>kubectl get svc &lt;service-name&gt; -n &lt;namespace&gt;</code></td>
<td>서비스의 외부 IP 확인</td>
</tr>
<tr>
<td><code>kubectl apply -f &lt;file-name&gt;.yaml</code></td>
<td>YAML 파일로 리소스를 생성하거나 업데이트</td>
</tr>
<tr>
<td><code>kubectl apply -f &lt;dir-path&gt;/</code></td>
<td>여러 YAML 파일을 한 번에 적용</td>
</tr>
<tr>
<td><code>kubectl get -f &lt;file-name&gt;.yaml</code></td>
<td>YAML 파일로 정의된 리소스의 상태 확인</td>
</tr>
<tr>
<td><code>kubectl delete -f &lt;file-name&gt;.yaml</code></td>
<td>YAML 파일로 정의된 리소스 삭제</td>
</tr>
<tr>
<td><code>kubectl apply -f &lt;file-name&gt;.yaml</code></td>
<td>수정된 YAML 파일을 클러스터에 반영</td>
</tr>
<tr>
<td><code>kubectl get &lt;resource-type&gt; &lt;resource-name&gt; -n &lt;namespace&gt; -o yaml</code></td>
<td>배포된 리소스를 YAML 형식으로 출력</td>
</tr>
<tr>
<td><code>kubectl create -f &lt;file-name&gt;.yaml -o yaml</code></td>
<td>리소스 생성 후 YAML 형식으로 출력</td>
</tr>
<tr>
<td><code>kubectl rollout status deployment/&lt;deployment-name&gt; -n &lt;namespace&gt;</code></td>
<td>배포된 리소스의 상태 확인</td>
</tr>
<tr>
<td><code>kubectl apply -f &lt;scaled-deployment&gt;.yaml</code></td>
<td>리소스의 replica 수를 변경</td>
</tr>
<tr>
<td><code>kubectl logs -f &lt;pod-name&gt; -n &lt;namespace&gt;</code></td>
<td>배포된 파드의 로그 확인</td>
</tr>
<tr>
<td><code>kubectl describe -f &lt;file-name&gt;.yaml</code></td>
<td>YAML 파일로 정의된 리소스의 세부 정보 확인</td>
</tr>
<tr>
<td><code>kubectl diff -f &lt;file-name&gt;.yaml</code></td>
<td>두 YAML 파일 간의 차이점 확인</td>
</tr>
<tr>
<td><code>kubectl apply -f &lt;file-name&gt;.yaml --dry-run=client</code></td>
<td>YAML 파일의 유효성 검증 (실제로 리소스를 생성하지 않음)</td>
</tr>
</tbody></table>