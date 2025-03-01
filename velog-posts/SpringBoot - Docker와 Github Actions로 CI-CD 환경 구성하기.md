<p>스프링부트 프로젝트를 도커와 깃허브 액션을 통해 CI/CD를 구성했던 과정을 간단히 기록해보려고 한다.
먼저 사전에 준비물이 필요하다.</p>
<ul>
<li>EC2 인스턴스</li>
<li>RDS</li>
<li>Docker 설치, 계정 및 DockerHub에 레포지토리 만들기</li>
</ul>
<p>위에 언급된 내용들이 준비된 상태에서 진행하면 된다.</p>
<h2 id="도커-파일-작성">도커 파일 작성</h2>
<p>먼저 스프링부트 프로젝트 루트 디렉토리에 도커 파일을 생성하고 스크립트를 작성한다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/bd74cb58-eb28-416b-8ca5-c382ab622e87/image.png" /></p>
<blockquote>
<p>docker build . --file Dockerfile --tag 도커허브ID/프로젝트이름:태그
docker build . --file Dockerfile --tag jhpark0131/coboard:latest</p>
</blockquote>
<p>터미널에서 위 명령어를 실행해서 도커 이미지가 제대로 생성되는지 확인해본다.
or 도커 파일에 FROM 옆에 보이는 실행 버튼을 눌러서 테스트 해볼 수도 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/221d63f7-6f67-4436-96d5-e00eb78dd495/image.png" /></p>
<p>정상적으로 빌드가 되었다면 도커 데스크탑에서 Image가 생성된 모습을 확인할 수 있다.</p>
<h2 id="도커-accesstoken">도커 AccessToken</h2>
<p>EC2에서 Docker Hub에 접속하거나 Github Actions에서 Docker 이미지를 빌드한 후 Docker Hub에 push하거나, pull 받을 때 AccessToken이 필요하다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/d2b882b1-1401-4ebe-b172-30f73d7325ed/image.png" /></p>
<p>계정 설정의 Personal access tokens를 생성하고 복사해서 잘 보관해주자.</p>
<h2 id="github-actions에-환경-변수-등록">Github Actions에 환경 변수 등록</h2>
<p>이제 깃허브 액션을 적용할 레포지토리 설정으로 이동해서 환경 변수를 등록해 줘야한다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/d601d45c-edb6-4e5a-85f9-8a1a40ceee35/image.png" /></p>
<p>Secretes and variables -&gt; Actions를 확인해보면 </p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/411c647b-8327-41ee-a2fc-28f52abcbf54/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/be306fed-5b08-4d86-9da8-fe30a0515cf5/image.png" /></p>
<p>사진에 보이는 것과 같이 Secrets와 Variables 설정할 수 있다. DB와 EC2관련 환경변수는 secrets로 설정했고 variables로 도커 유저 이름과 도커 이미지 이름을 설정해 두었지만 모두 secrets로 설정해도 무방하다.</p>
<ul>
<li>DOCKERHUB_TOKEN : 도커 AccessToken</li>
<li>EC2_KEY : EC2에 접속할 때 사용하는 ssh key (id_rsa)의 내용을 복사해서 채워준다.</li>
<li>EC2_USER : EC2에 접속할 때 사용하는 유저 이름을 채워주면 된다. ex) ubuntu</li>
<li>EC2_HOST : EC2인스턴스의 퍼블릭 주소를 입력해준다.<ul>
<li>퍼블릭 IPv4 주소와 퍼블릭 IPv4 DNS 모두 외부에서 접근할 수 있게 해주므로, 어느 것을 사용해도 괜찮지만, 퍼블릭 IPv4 DNS를 사용하는 것이 일반적으로 더 안전하고, IP주소가 변경될 때 DNS 이름을 그대로 사용할 수 있다는 장점이 있다.</li>
</ul>
</li>
<li>DB_HOST : RDS 퍼블릭 엔드포인트</li>
<li>DB_USER : RDS 유저 이름</li>
<li>DB_PASSWORD : RDS 패스워드</li>
</ul>
<h2 id="github-actions-스크립트yml-작성">Github Actions 스크립트(yml) 작성</h2>
<p>프로젝트 루트 디렉토리에서 <strong>.github -&gt; workflows</strong> 패키지를 생성하고 안에다 yml 파일을 작성해준다. yml파일 이름은 아무렇게나 지어도 무방하지만 어떤 기능을 할 것인지 명확하게 해주는 쪽이 좋아보인다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/b2c1d7dc-3880-4826-b948-3956252ecdf1/image.png" /></p>
<pre><code class="language-yml">name: build to github

on:
  push:
    branches: [main, develop] # 해당 branch에 push 되었을 경우

jobs:
  github-build-and-push:
    runs-on: ubuntu-22.04

    # 실행 스텝 지정
    # https://github.com/marketplace/actions/build-with-gradle
    steps:
      - name: Checkout sources
        uses: actions/checkout@v4

      # java version 지정
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: 17

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4

      # Build
      - name: Build with Gradle
        run: ./gradlew clean build


      # https://github.com/marketplace/actions/build-and-push-docker-images
      # 로그인
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 관련 설적 적용
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Build 및 Push
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: . # 지정하지 않으면 기본적으로 현재 디렉토리가 빌드 컨텍스트로 사용됩니다.
          file: ./Dockerfile
          push: true
          tags: ${{ vars.DOCKERHUB_USERNAME }}/${{ vars.DOCKER_IMAGE_TAG_NAME }}:latest

  deploy-to-ec2:
    needs: github-build-and-push
    runs-on: ubuntu-22.04
    # https://github.com/marketplace/actions/ssh-remote-commands
    steps:
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            CONTAINER_ID=$(sudo docker ps -q --filter &quot;publish=8080-8080&quot;)

            if [ ! -z &quot;$CONTAINER_ID&quot; ]; then
              sudo docker stop $CONTAINER_ID
              sudo docker rm $CONTAINER_ID
            fi

            sudo docker pull ${{ vars.DOCKERHUB_USERNAME }}/${{ vars.DOCKER_IMAGE_TAG_NAME }}:latest
            sudo docker run -d -p 8080:8080 \
                -e DB_USER=${{secrets.DB_USER}} \
                -e DB_PASSWORD=${{secrets.DB_PASSWORD}} \
                -e DB_HOST=${{secrets.DB_HOST}} \
                ${{ vars.DOCKERHUB_USERNAME }}/${{ vars.DOCKER_IMAGE_TAG_NAME }}:latest</code></pre>
<p>yml파일의 내용을 살펴보면 main, develop 브랜치에 push를 하고 병합이 이뤄지게 되면
jobs에 있는 'github-build-and-push' 그리고 'deploy-to-ec2'가 순차적으로 실행되게 된다. 방금 위에서 설정해준 환경 변수들이 yml 파일에서 쓰이는 모습을 확인할 수 있다.</p>
<h2 id="결과-확인">결과 확인</h2>
<p>이제 개발한 내용을 commit -&gt; push 하고 main or develop 브랜치에서 merge하게 되면 Github Actions가 동작하게 된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/3f9f2639-9109-4198-b2a8-dd0f8631496d/image.png" /></p>
<p>레포지토리에서 Actions 탭을 눌러보면 진행 상황을 확인할 수 있고</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/517ef39c-f3a7-45be-ad5c-88d7815e7148/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/2562c205-3e3f-4d79-903d-f11ebcfce6f1/image.png" /></p>
<p>워크플로우를 눌러 안으로 들어가게 되면 상세한 로그들을 확인할 수 있다.</p>