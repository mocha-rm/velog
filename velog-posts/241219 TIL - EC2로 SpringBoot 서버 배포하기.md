<p>EC2를 이용해 스프링 프로젝트를 배포하는 과정을 정리해본다.</p>
<h1 id="1-ec2-인스턴스-생성">1. EC2 인스턴스 생성</h1>
<p>먼저 AWS에 가입하고 EC2로 이동한 후에 '인스턴스 시작'을 클릭해서 생성 화면으로 넘어간다.
<img alt="" src="https://velog.velcdn.com/images/jelog_131/post/248244c1-72a2-4e6f-bc75-e401c6355728/image.png" /></p>
<p>이름을 지어주고 OS를 선택해주면 되는데 서버용으로 많이 사용되는 Ubuntu를 선택했다
아래 쪽에 어떤 스펙의 컴퓨터를 사용할지 고를 수 있는데 개발용으로 사용하는게 목적이라 프리 티어로도 충분하다고 생각한다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/a0bbef49-3ae6-4dac-97fb-c4d88dad4c73/image.png" /></p>
<p>ssh로 터미널에서 EC2에 접근하기 위해 키페어를 생성해준다.
키 페어는 한번 생성하면 두번 다시 다운로드 받을 수 없기 때문에 잘 보관하고 있어야 한다 ! </p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/0ea55826-c166-432b-be08-93c6cc7bf86e/image.png" /></p>
<p>보안 그룹을 생성해주고 ssh, https, http 트래픽 허용에 체크해준다.
밑에 나머지 값 들은 디폴트 세팅으로 해서 인스턴스를 생성해주었다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/9bc46fa6-254e-47ab-9469-b29115eb4b66/image.png" /></p>
<h1 id="2-ec2-접속하기">2. EC2 접속하기</h1>
<p>인스턴스 목록이 보이는 화면에서 인슨턴스 ID를 눌러 들어가게 되면 자세한 정보들이 나오는데 우측에 연결이라는 버튼을 누르게되면 어떠한 방법으로 접속할 건지 볼 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/1da3a09c-b3d4-448d-8de0-4355af71902d/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/d2ace421-98b4-4b64-bda8-7e815b6acd50/image.png" /></p>
<p>EC2 Instance Connect을 사용하여 연결을 선택하고 연결을 눌러주면</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/c249c627-ed3a-416c-ada3-500c8d4a7d77/image.png" /></p>
<p>이렇게 웹상에서 바로 접속할 수 있다.</p>
<p>발급 받은 키페어를 이용해 접속하려면 </p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/bb8e3b7c-5cb9-4f48-93bb-20eef5395db7/image.png" /></p>
<p>SSH 클라이언트를 선택하고</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/f21befb4-3025-4683-9424-eb3a6f61dde5/image.png" /></p>
<p>아래의 인스턴스 엑세스 방법을 보고 차례대로 진행해주면 된다.</p>
<hr />
<h1 id="3-ec2-세팅-및-서버-실행하기">3. EC2 세팅 및 서버 실행하기</h1>
<p>정상적으로 EC2에 접속되었다면 초기 세팅이 필요하다. 
먼저 Java와 git을 설치해주고 사용하고 있는 DB를 설치해준다.</p>
<p>그리고 정적 파일 배포에는 2가지 방식이 있는데,</p>
<ol>
<li>EC2에서 프로젝트 git clone 후 실행하기</li>
<li>로컬 PC에서 빌드하고 jar 파일만 EC2에 복사 후 실행하기</li>
</ol>
<p>일단은 1번 방법으로 진행했다. 깃을 설치했다면 깃헙에서 SSH KEY를 생성해야 한다.</p>
<blockquote>
<p>cd ~/.ssh
ssh-keygen -t rsa -C 깃허브계정(메일)</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/2c358b47-3c24-48d1-b4c7-95ebf6b00924/image.png" /></p>
<p>위 명령어를 통해 .ssh 디렉토리에 키페어를 생성하게 되고 id_rsa.pub 파일이 생성된다.</p>
<blockquote>
<p>cat id_rsa.pub</p>
</blockquote>
<p>cat 명령어로 id_rsa.pub 내용을 조회하고 복사해서 깃허브 ssh에 넣어줘야 한다.
깃허브로 이동해서 setting -&gt; SSH and GPG keys -&gt; new SSH key</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/7c52f16b-2d09-4f79-a26f-3411053783e3/image.png" /></p>
<p>이곳에 이름을 지어주고 Key에 내용을 붙여넣기 한다.</p>
<hr />
<h4 id="git-clone-하기">Git Clone 하기</h4>
<p>여기 까지 되었다면 이제 SSH를 이용해서 git clone을 받으면 된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/4cd608f7-79dd-45bc-a544-29b2cdef4d61/image.png" /></p>
<p>깃허브 레포지토리로 이동 후 Code -&gt; SSH 탭에 있는 주소를 복사해서 클론을 진행한다.</p>
<h4 id="빌드-후-서버-실행하기">빌드 후 서버 실행하기</h4>
<p>정상적으로 클론이 되었다면 프로젝트가 있는 경로로 이동한 후</p>
<blockquote>
<p>./gradlew build</p>
</blockquote>
<p>위 명령어를 통해 프로젝트를 빌드 한다.
빌드가 성공했다면 build -&gt; libs 디렉토리가 생성되고 폴더 안에 jar 파일이 생성된 것을 확인할 수 있다.</p>
<blockquote>
<p>nohup java -jar 파일이름.jar &amp;</p>
</blockquote>
<p>위 명령어로 생성된 jar 파일을 실행시켜준다. nohup 명령어 뒤에 &amp;를 붙이면 백그라운드에서 실행한다는 의미로 EC2 접속을 끊더라도 서버가 종료되지 않게된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/d22e16f0-6fb4-4288-b879-1743b8529c34/image.png" /></p>
<h4 id="서버-종료하기">서버 종료하기</h4>
<blockquote>
<p>jobs</p>
</blockquote>
<p>jobs 명령어로 현재 백그라운드에서 실행되고 있는 것들을 확인하고</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/11d6ef2d-0a59-4dce-bb7a-ecfcbb23a0ff/image.png" /></p>
<blockquote>
<p>fg %인덱스</p>
</blockquote>
<p>fg %(인덱스 번호) 로 해당 프로그램을 가져올 수 있다. 그 상태에서 control + c를 입력해서 종료 시켜줄 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/4569f9d9-c484-4a97-a78c-b20295877f4e/image.png" /></p>
<hr />
<h2 id="마무리">마무리</h2>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/664fb52c-d3b8-4132-b7ea-59d4311fa93d/image.png" /></p>
<p>정상적으로 API가 호출되는 모습을 확인할 수 있었다.
이렇게 EC2에서 Spring Boot 서버를 실행해 보았다 ! 고정 IP로 사용하기 위해서는 탄력적 IP를 적용해줘야 하고 추가 요금이 발생한다, 개발용 테스트용으로는 따로 설정해주지 않아도 충분할 것 같다.</p>