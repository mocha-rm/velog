<h1 id="docker-compose-기반-spring-boot-백엔드-railway-배포-완전-정복-redismongodbmysql-포함">Docker Compose 기반 Spring Boot 백엔드, Railway 배포 완전 정복 (Redis/MongoDB/MySQL 포함)</h1>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/d65dccdb-c0c1-4e58-881f-06c42d9035ee/image.png" /></p>
<h2 id="들어가며">들어가며</h2>
<p>최근 사이드 프로젝트로 Spring Boot 기반의 RESTful API 서버를 개발하고 있었습니다. 로컬 환경에서는 Docker Compose를 활용해 Redis, MongoDB, MySQL을 함께 운영하며 개발을 진행했는데, 배포 단계에서 고민이 많았습니다.</p>
<p>AWS EC2나 GCP Compute Engine을 사용하자니 설정할 것이 너무 많고, Heroku는 2022년 11월부터 무료 플랜이 사라져서 대안을 찾고 있었죠. 그러던 중 개발자 커뮤니티에서 <strong>Railway</strong>라는 서비스를 알게 되었고, 직접 마이그레이션해본 경험을 상세히 공유하고자 합니다.</p>
<h2 id="기존-환경-분석">기존 환경 분석</h2>
<h3 id="로컬-개발-환경">로컬 개발 환경</h3>
<ul>
<li><strong>Framework</strong>: Spring Boot 3.1.5</li>
<li><strong>Java Version</strong>: OpenJDK 17</li>
<li><strong>Build Tool</strong>: Gradle 8.4</li>
<li><strong>Database</strong>: MySQL 8.0 (주 데이터), MongoDB 6.0 (로그 데이터)</li>
<li><strong>Cache</strong>: Redis 7.0</li>
<li><strong>Container</strong>: Docker Compose</li>
</ul>
<h3 id="docker-compose-구성">Docker Compose 구성</h3>
<pre><code class="language-yaml">version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - &quot;8080:8080&quot;
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - MYSQL_HOST=mysql
      - MYSQL_PORT=3306
      - MYSQL_DATABASE=myapp
      - MYSQL_USERNAME=app_user
      - MYSQL_PASSWORD=secure_password
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - MONGO_HOST=mongo
      - MONGO_PORT=27017
      - MONGO_DATABASE=myapp_logs
    depends_on:
      - mysql
      - redis
      - mongo
    networks:
      - app-network

  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_DATABASE=myapp
      - MYSQL_USER=app_user
      - MYSQL_PASSWORD=secure_password
      - MYSQL_ROOT_PASSWORD=root_password
    ports:
      - &quot;3306:3306&quot;
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - &quot;6379:6379&quot;
    command: redis-server --requirepass redis_password
    volumes:
      - redis_data:/data
    networks:
      - app-network

  mongo:
    image: mongo:6
    environment:
      - MONGO_INITDB_ROOT_USERNAME=mongo_user
      - MONGO_INITDB_ROOT_PASSWORD=mongo_password
      - MONGO_INITDB_DATABASE=myapp_logs
    ports:
      - &quot;27017:27017&quot;
    volumes:
      - mongo_data:/data/db
    networks:
      - app-network

volumes:
  mysql_data:
  redis_data:
  mongo_data:

networks:
  app-network:
    driver: bridge</code></pre>
<h3 id="dockerfile-구성">Dockerfile 구성</h3>
<pre><code class="language-dockerfile">FROM openjdk:17-jdk-slim

WORKDIR /app

COPY build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;/app/app.jar&quot;]</code></pre>
<h3 id="주요-기능">주요 기능</h3>
<ul>
<li><strong>인증/인가</strong>: JWT 기반 인증 시스템</li>
<li><strong>파일 업로드</strong>: MultipartFile 처리 (이미지, 문서)</li>
<li><strong>실시간 알림</strong>: WebSocket 연결</li>
<li><strong>API 문서화</strong>: Swagger/OpenAPI 3.0</li>
<li><strong>로깅</strong>: 구조화된 로그를 MongoDB에 저장</li>
<li><strong>캐싱</strong>: Redis를 활용한 세션 관리 및 데이터 캐싱</li>
</ul>
<h2 id="railway-마이그레이션-과정">Railway 마이그레이션 과정</h2>
<h3 id="1단계-railway-프로젝트-초기화">1단계: Railway 프로젝트 초기화</h3>
<p>Railway 대시보드에서 새 프로젝트를 생성하는 과정은 놀라울 정도로 간단했습니다.</p>
<ol>
<li><strong>GitHub 연동</strong>: Railway에서 &quot;Deploy from GitHub repo&quot; 선택</li>
<li><strong>Repository 선택</strong>: 배포할 레포지토리를 선택하면 자동으로 Dockerfile을 인식</li>
<li><strong>자동 빌드</strong>: 첫 번째 배포가 자동으로 시작됨</li>
</ol>
<pre><code class="language-bash"># Railway CLI 설치 (선택사항)
npm install -g @railway/cli

# 로컬에서 Railway 프로젝트 연결
railway login
railway link [프로젝트-ID]</code></pre>
<h3 id="2단계-데이터베이스-서비스-추가">2단계: 데이터베이스 서비스 추가</h3>
<p>Railway의 가장 큰 장점 중 하나는 데이터베이스를 별도로 관리할 필요가 없다는 것입니다.</p>
<h4 id="2-1-mysql-서비스-추가">2-1. MySQL 서비스 추가</h4>
<pre><code>Railway Dashboard → Add Service → Database → MySQL</code></pre><p>Railway에서 제공하는 MySQL 인스턴스 정보:</p>
<ul>
<li><strong>Host</strong>: <code>containers-us-west-xxx.railway.app</code></li>
<li><strong>Port</strong>: <code>6543</code></li>
<li><strong>Database</strong>: <code>railway</code></li>
<li><strong>Username</strong>: <code>root</code> </li>
<li><strong>Password</strong>: 자동 생성된 32자리 패스워드</li>
</ul>
<h4 id="2-2-redis-서비스-추가">2-2. Redis 서비스 추가</h4>
<pre><code>Railway Dashboard → Add Service → Database → Redis</code></pre><p>Redis 연결 정보:</p>
<ul>
<li><strong>Host</strong>: <code>redis-xxx.railway.app</code></li>
<li><strong>Port</strong>: <code>6379</code></li>
<li><strong>Password</strong>: 자동 생성</li>
</ul>
<h4 id="2-3-mongodb-서비스-추가">2-3. MongoDB 서비스 추가</h4>
<pre><code>Railway Dashboard → Add Service → Database → MongoDB</code></pre><p>MongoDB 연결 정보:</p>
<ul>
<li><strong>Connection URI</strong>: <code>mongodb://mongo:xxx@containers-us-west-xxx.railway.app:6034/railway</code></li>
</ul>
<h3 id="3단계-환경변수-구성">3단계: 환경변수 구성</h3>
<p>Railway는 각 데이터베이스 서비스마다 연결 정보를 환경변수로 자동 제공합니다. 이를 Spring Boot의 <code>application.yml</code>에 맞게 매핑해야 합니다.</p>
<h4 id="railway-제공-환경변수">Railway 제공 환경변수</h4>
<pre><code class="language-env"># MySQL
MYSQL_URL=mysql://root:xxx@containers-us-west-xxx.railway.app:6543/railway
MYSQLHOST=containers-us-west-xxx.railway.app
MYSQLPORT=6543
MYSQLDATABASE=railway
MYSQLUSER=root
MYSQLPASSWORD=xxx

# Redis
REDIS_URL=redis://:xxx@redis-xxx.railway.app:6379
REDISHOST=redis-xxx.railway.app
REDISPORT=6379
REDISPASSWORD=xxx

# MongoDB  
MONGO_URL=mongodb://mongo:xxx@containers-us-west-xxx.railway.app:6034/railway
MONGOHOST=containers-us-west-xxx.railway.app
MONGOPORT=6034
MONGOUSER=mongo
MONGOPASSWORD=xxx
MONGODATABASE=railway</code></pre>
<h4 id="applicationyml-수정">application.yml 수정</h4>
<pre><code class="language-yaml">spring:
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:prod}

  # MySQL 설정
  datasource:
    url: jdbc:mysql://${MYSQLHOST:localhost}:${MYSQLPORT:3306}/${MYSQLDATABASE:myapp}?useSSL=false&amp;serverTimezone=UTC&amp;allowPublicKeyRetrieval=true
    username: ${MYSQLUSER:root}
    password: ${MYSQLPASSWORD:password}
    driver-class-name: com.mysql.cj.jdbc.Driver

  # JPA 설정
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true

  # MongoDB 설정
  data:
    mongodb:
      host: ${MONGOHOST:localhost}
      port: ${MONGOPORT:27017}
      database: ${MONGODATABASE:myapp_logs}
      username: ${MONGOUSER:}
      password: ${MONGOPASSWORD:}
      authentication-database: admin

  # Redis 설정
  data:
    redis:
      host: ${REDISHOST:localhost}
      port: ${REDISPORT:6379}
      password: ${REDISPASSWORD:}
      timeout: 60000
      jedis:
        pool:
          max-active: 8
          max-wait: -1
          max-idle: 8
          min-idle: 0

# 서버 설정
server:
  port: ${PORT:8080}

# JWT 설정
jwt:
  secret: ${JWT_SECRET:your-secret-key}
  expiration: ${JWT_EXPIRATION:3600000}

# 파일 업로드 설정
file:
  upload:
    path: ${FILE_UPLOAD_PATH:/tmp/uploads}
    max-size: ${FILE_MAX_SIZE:10485760}</code></pre>
<h3 id="4단계-railway-variables-설정">4단계: Railway Variables 설정</h3>
<p>Railway 대시보드에서 Variables 탭으로 이동해 다음 환경변수들을 추가했습니다.</p>
<pre><code class="language-env"># Spring 프로파일
SPRING_PROFILES_ACTIVE=prod

# JWT 설정
JWT_SECRET=your-super-secure-jwt-secret-key-here
JWT_EXPIRATION=3600000

# 파일 업로드 설정
FILE_UPLOAD_PATH=/app/uploads
FILE_MAX_SIZE=10485760

# 로깅 레벨
LOGGING_LEVEL_ROOT=INFO
LOGGING_LEVEL_COM_MYAPP=DEBUG

# Railway 포트 (자동 설정됨)
PORT=${{PORT}}</code></pre>
<h3 id="5단계-배포-및-테스트">5단계: 배포 및 테스트</h3>
<p>코드를 GitHub에 push하면 Railway가 자동으로 배포를 시작합니다.</p>
<pre><code class="language-bash">git add .
git commit -m &quot;Configure Railway deployment&quot;
git push origin main</code></pre>
<p>배포 과정은 Railway 대시보드의 Deployments 탭에서 실시간으로 확인할 수 있습니다.</p>
<ol>
<li><strong>Build Phase</strong>: Dockerfile 기반 이미지 빌드</li>
<li><strong>Deploy Phase</strong>: 컨테이너 실행 및 health check</li>
<li><strong>Live</strong>: 서비스 활성화 완료</li>
</ol>
<p>배포가 완료되면 Railway에서 제공하는 도메인 (예: <code>https://myapp-production-xxxx.up.railway.app</code>)으로 접근할 수 있습니다.</p>
<h2 id="성능-및-모니터링">성능 및 모니터링</h2>
<h3 id="railway-제공-메트릭">Railway 제공 메트릭</h3>
<p>Railway 대시보드에서 다음과 같은 메트릭을 실시간으로 확인할 수 있습니다.</p>
<ul>
<li><strong>CPU 사용량</strong>: 평균 15-20% (유휴 상태 기준)</li>
<li><strong>메모리 사용량</strong>: 약 300MB (Spring Boot JVM 기본 설정)</li>
<li><strong>네트워크 I/O</strong>: 인바운드/아웃바운드 트래픽</li>
<li><strong>응답 시간</strong>: 평균 200ms 이하</li>
</ul>
<h3 id="로그-모니터링">로그 모니터링</h3>
<p>Railway는 실시간 로그 스트리밍을 제공합니다.</p>
<pre><code class="language-bash"># Railway CLI로 로그 확인
railway logs --follow

# 특정 서비스 로그만 확인
railway logs --service mysql --follow</code></pre>
<h3 id="health-check-구성">Health Check 구성</h3>
<p>Spring Boot Actuator를 활용해 health check 엔드포인트를 구성했습니다.</p>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/actuator&quot;)
public class HealthController {

    @Autowired
    private RedisTemplate&lt;String, String&gt; redisTemplate;

    @Autowired
    private DataSource dataSource;

    @GetMapping(&quot;/health&quot;)
    public ResponseEntity&lt;Map&lt;String, String&gt;&gt; health() {
        Map&lt;String, String&gt; status = new HashMap&lt;&gt;();

        try {
            // MySQL 연결 확인
            try (Connection connection = dataSource.getConnection()) {
                status.put(&quot;mysql&quot;, &quot;UP&quot;);
            }

            // Redis 연결 확인
            redisTemplate.opsForValue().set(&quot;health-check&quot;, &quot;OK&quot;);
            status.put(&quot;redis&quot;, &quot;UP&quot;);

            status.put(&quot;status&quot;, &quot;UP&quot;);
            return ResponseEntity.ok(status);

        } catch (Exception e) {
            status.put(&quot;status&quot;, &quot;DOWN&quot;);
            status.put(&quot;error&quot;, e.getMessage());
            return ResponseEntity.status(503).body(status);
        }
    }
}</code></pre>
<h2 id="비용-분석">비용 분석</h2>
<h3 id="railway-요금-체계">Railway 요금 체계</h3>
<p>Railway는 사용량 기반 과금 모델을 사용합니다.</p>
<ul>
<li><strong>Compute</strong>: $0.000463/vCPU/minute</li>
<li><strong>Memory</strong>: $0.000231/GB/minute</li>
<li><strong>Egress</strong>: $0.10/GB</li>
<li><strong>Storage</strong>: 데이터베이스별 별도 과금</li>
</ul>
<h3 id="실제-사용-비용-월간">실제 사용 비용 (월간)</h3>
<ul>
<li><strong>Spring Boot App</strong>: CPU 0.5, Memory 512MB → 약 $8/월</li>
<li><strong>MySQL</strong>: 약 $5/월</li>
<li><strong>Redis</strong>: 약 $3/월</li>
<li><strong>MongoDB</strong>: 약 $4/월</li>
<li><strong>Total</strong>: 약 $20/월</li>
</ul>
<p>$20 무료 크레딧으로 약 1개월간 무료 사용이 가능합니다.</p>
<h3 id="기존-솔루션과-비교">기존 솔루션과 비교</h3>
<ul>
<li><strong>AWS EC2 t3.micro</strong>: $8.5/월 + RDS $15/월 + ElastiCache $13/월 = $36.5/월</li>
<li><strong>Heroku</strong>: Dyno $7/월 + PostgreSQL $9/월 + Redis $15/월 = $31/월</li>
<li><strong>Railway</strong>: 약 $20/월 (올인원)</li>
</ul>
<h2 id="최적화-및-팁">최적화 및 팁</h2>
<h3 id="1-빌드-시간-최적화">1. 빌드 시간 최적화</h3>
<pre><code class="language-dockerfile"># Multi-stage build로 빌드 시간 단축
FROM gradle:7.6-jdk17 AS build
COPY --chown=gradle:gradle . /home/gradle/src
WORKDIR /home/gradle/src
RUN gradle build --no-daemon

FROM openjdk:17-jdk-slim
COPY --from=build /home/gradle/src/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;/app.jar&quot;]</code></pre>
<h3 id="2-환경별-배포-설정">2. 환경별 배포 설정</h3>
<p>Railway에서는 브랜치별로 다른 환경을 구성할 수 있습니다.</p>
<ul>
<li><strong>main</strong> 브랜치 → Production 환경</li>
<li><strong>develop</strong> 브랜치 → Staging 환경</li>
</ul>
<h3 id="3-자동-ssl-및-도메인">3. 자동 SSL 및 도메인</h3>
<p>Railway는 자동으로 HTTPS를 적용하고, 커스텀 도메인도 쉽게 연결할 수 있습니다.</p>
<pre><code>Settings → Environment → Custom Domain → Add Domain</code></pre><h3 id="4-백업-설정">4. 백업 설정</h3>
<p>각 데이터베이스는 자동 백업이 활성화되어 있지만, 중요한 데이터는 별도 백업 전략을 수립하는 것이 좋습니다.</p>
<h2 id="🚨-주의사항-및-한계점">🚨 주의사항 및 한계점</h2>
<h3 id="장점">장점</h3>
<ul>
<li><strong>간편한 배포</strong>: Dockerfile만 있으면 즉시 배포 가능</li>
<li><strong>통합 관리</strong>: DB까지 한 곳에서 관리</li>
<li><strong>자동 스케일링</strong>: 트래픽에 따른 자동 확장</li>
<li><strong>실시간 모니터링</strong>: 로그, 메트릭 실시간 확인</li>
<li><strong>합리적인 가격</strong>: 소규모 프로젝트에 적합</li>
</ul>
<h3 id="한계점">한계점</h3>
<ul>
<li><strong>지역 제한</strong>: 아직 아시아 리전 없음 (레이턴시 약 150-200ms)</li>
<li><strong>커스터마이징 제약</strong>: 고도의 인프라 커스터마이징이 어려움</li>
<li><strong>대용량 트래픽</strong>: 대규모 서비스에는 부적합할 수 있음</li>
<li><strong>한국어 지원</strong>: 공식 한국어 문서 부재</li>
</ul>
<h3 id="추천-대상">추천 대상</h3>
<ul>
<li>사이드 프로젝트 및 MVP 개발</li>
<li>스타트업 초기 단계</li>
<li>개인 포트폴리오 프로젝트</li>
<li>프로토타입 개발 및 테스트</li>
</ul>
<h2 id="향후-계획">향후 계획</h2>
<h3 id="1-cicd-파이프라인-구성">1. CI/CD 파이프라인 구성</h3>
<p>GitHub Actions와 Railway를 연동한 자동 배포 파이프라인을 구축할 예정입니다.</p>
<pre><code class="language-yaml"># .github/workflows/deploy.yml
name: Deploy to Railway

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install Railway CLI
        run: npm install -g @railway/cli
      - name: Deploy to Railway
        run: railway deploy
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}</code></pre>
<h3 id="2-모니터링-강화">2. 모니터링 강화</h3>
<p>Prometheus + Grafana를 통한 커스텀 메트릭 수집과 New Relic 연동을 검토하고 있습니다.</p>
<h3 id="3-로드-테스트">3. 로드 테스트</h3>
<p>Apache JMeter를 활용해 Railway 환경에서의 성능 한계를 측정하고, 최적화 포인트를 찾을 계획입니다.</p>
<h2 id="결론">결론</h2>
<p>Docker Compose에서 Railway로의 마이그레이션은 예상보다 훨씬 수월했습니다. 특히 데이터베이스 관리의 복잡성이 크게 줄어들었고, 배포 과정도 자동화되어 개발에만 집중할 수 있게 되었습니다.</p>
<p>Railway는 다음과 같은 경우에 특히 추천합니다.</p>
<ol>
<li><strong>빠른 MVP 배포</strong>가 필요한 경우</li>
<li><strong>인프라 관리</strong>보다 <strong>개발에 집중</strong>하고 싶은 경우  </li>
<li><strong>합리적인 비용</strong>으로 풀스택 환경을 구축하고 싶은 경우</li>
</ol>
<p>물론 대규모 트래픽을 처리해야 하거나, 특수한 인프라 요구사항이 있다면 AWS나 GCP를 고려하는 것이 좋습니다. 하지만 개인 프로젝트나 스타트업 초기 단계라면 Railway가 매우 실용적인 선택지라고 생각합니다.</p>
<hr />
<h2 id="참고-자료">참고 자료</h2>
<ul>
<li><a href="https://docs.railway.app/">Railway 공식 문서</a></li>
<li><a href="https://spring.io/projects/spring-boot">Spring Boot 공식 문서</a></li>
<li><a href="https://docs.docker.com/compose/">Docker Compose 공식 문서</a></li>
</ul>
<p><strong>💡 Railway $20 크레딧 받기</strong>: <a href="https://railway.com?referralCode=qZFbvo">여기서 가입</a></p>