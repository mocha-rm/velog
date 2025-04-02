<h2 id="pvpersistent-volume와-pvcpersistent-volume-claim">PV(Persistent Volume)와 PVC(Persistent Volume Claim)</h2>
<p>쿠버네티스에서 Persistent Volume(PV)는 클러스터에서 관리하는 스토리지 리소스이고, Persistent Volume Claim(PVC)는 사용자가 해당 스토리지를 요청하는 방식이다.</p>
<ul>
<li>Persistent Volume(PV)<ul>
<li>관리자가 설정한 정적인 스토리지 리소스</li>
<li>다양한 스토리지 백엔드(NFS, AWS EBS, GCE Persistent Disk 등) 지원</li>
<li>수명 주기가 노드의 독립적</li>
</ul>
</li>
<li>Persistent Volume Claim(PVC)<ul>
<li>개발자가 필요한 스토리지를 요청하는 오브젝트</li>
<li>PV가 클레임에 매칭되면 바인딩되어 사용 가능</li>
<li>요청한 용량, 접근 모드(ReadWriteOnce, ReadOnlyMany 등)에 따라 적절한 PV가 할당됨</li>
</ul>
</li>
</ul>
<hr />
<h2 id="statefulset과-stateless-차이">StatefulSet과 Stateless 차이</h2>
<h3 id="stateful-vs-stateless">Stateful vs Stateless</h3>
<ul>
<li>Stateless(무상태 애플리케이션)<ul>
<li>특정 요청에 대한 상태를 저장하지 않음</li>
<li>여러 인스턴스를 동일한 방식으로 생성/삭제 가능 (ex: 웹서버, API 서버)</li>
<li>확장이 용이하며 로드 밸런서를 통해 부하 분산 가능</li>
</ul>
</li>
<li>Stateful(상태 저장 애플리케이션)<ul>
<li>특정 요청에 대한 상태를 유지해야 하는 애플리케이션 (ex: 데이터베이스, 메시지 큐)</li>
<li>각 인스턴스가 고유한 식별자와 데이터를 가지며, 특정 순서로 생성/삭제됨</li>
</ul>
</li>
</ul>
<h3 id="statefulset-이란">StatefulSet 이란?</h3>
<p>StatefulSet은 상태를 가지는 (Stateful) 애플리케이션을 관리하는 쿠버네티스 컨트롤러이다.</p>
<ul>
<li>각 Pod에 고유한 네트워크 ID(Stable Network Identity) 부여</li>
<li>Pod 재시작 또는 재배포 시에도 일관된 시토리지 유지 (PVC와 함께 사용)</li>
<li>Pod 번호 순서 보장 (ex: my-sql-0, my-sql-1)</li>
<li>주로 데이터베이스 (MySQL, PostgreSQL), Kafka, Redis 같은 상태 기반 서비스에서 사용</li>
</ul>
<h4 id="statefulset-기본-구성-예제">StatefulSet 기본 구성 예제</h4>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-db
spec:
  serviceName: &quot;my-db&quot;
  replicas: 3
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
      - name: my-db
        image: mysql:latest
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [&quot;ReadWriteOnce&quot;]
      resources:
        requests:
          storage: 10Gi</code></pre>
<hr />
<h2 id="batch-프로그램과-쿠버네티스-job">Batch 프로그램과 쿠버네티스 Job</h2>
<h3 id="batch-프로그램이란">Batch 프로그램이란?</h3>
<ul>
<li>일정한 작업을 한 번 또는 주기적으로 실행하는 프로그램</li>
<li>사용자 입력 없이 실행되며 대량의 데이터 처리에 적합</li>
<li>ex: 로그 분석, 백업, 데이터 변환, 배치 처리</li>
</ul>
<h3 id="쿠버네티스-job">쿠버네티스 Job</h3>
<p>Job은 일회성 또는 특정 횟수만큼 실행되는 작업을 관리하는 오브젝트이다.</p>
<ul>
<li>Pod가 성공적으로 완료될 때까지 실행됨</li>
<li>실패한 경우 재시도 가능</li>
<li>completions 및 parallelism 설정을 통해 여러 개의 작업을 병렬로 실행 가능</li>
</ul>
<h4 id="job-예제">Job 예제</h4>
<pre><code class="language-yaml">apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  template:
    spec:
      containers:
      - name: batch-job
        image: busybox
        command: [&quot;echo&quot;, &quot;Hello, Kubernetes Batch Job!&quot;]
      restartPolicy: Never
  backoffLimit: 4  # 실패 시 최대 4번 재시도</code></pre>
<h3 id="cronjob-스케줄링-job">CronJob (스케줄링 Job)</h3>
<ul>
<li>Job을 일정한 주기로 실행하는 오브젝트</li>
<li>Cron 표현식(* * * * *)을 사용하여 실행 시간 설정 가능</li>
<li>ex: 매일 자정에 백업 실행, 5분마다 로그 분석 실행</li>
</ul>
<h4 id="cronjob-예제">CronJob 예제</h4>
<pre><code class="language-yaml">apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: &quot;0 0 * * *&quot;  # 매일 자정
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: busybox
            command: [&quot;echo&quot;, &quot;Backing up data...&quot;]
          restartPolicy: OnFailure</code></pre>
<hr />
<h2 id="정리">정리</h2>
<ul>
<li>PV/PVC : 쿠버네티스에서 상태를 저장하는 방법 (스토리지)</li>
<li>StatefulSet : 상태를 유지하는 애플리케이션을 위한 관리 방식</li>
<li>Stateless vs Stateful : 상태 저장 여부에 따른 애플리케이션의 특성</li>
<li>Job : 일회성 작업을 수행하는 쿠버네티스 오브젝트</li>
<li>CronJob : 일정 주기로 실행되는 Job</li>
</ul>