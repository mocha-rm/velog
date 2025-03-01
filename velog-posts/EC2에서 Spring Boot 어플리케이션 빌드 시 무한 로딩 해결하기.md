<h2 id="문제-발생">문제 발생</h2>
<p>EC2에서 스프링 서버를 실행시키기 위해 git clone을 받고 해당 디렉토리에 들어가</p>
<blockquote>
<p>./gradlew build</p>
</blockquote>
<p>명령어를 입력해 빌드를 하던 중 더 이상 진행되지 않고 멈춰 있길래 연결을 종료하고 다시 웹에서 EC2에 접속했는데 무한 로딩이 진행되면서 EC2 콘솔에 접속되지 않는 일이 발생했다 ..ㅠ </p>
<p>급하게 구글링으로 정보를 찾아보니 현재 프리티어로 사용 중이기 때문에 메모리가 부족해서 그렇다는 내용을 찾을 수 있었다.</p>
<h2 id="해결-방안">해결 방안</h2>
<h4 id="1-리눅스-스왑-메모리-사용">1. 리눅스 스왑 메모리 사용</h4>
<p>스왑 메모리란 실제 메모리가 가득 차고 더 많은 메모리가 필요할 때 디스크 공간을 대체해서 사용하는 것이다. RAM 대신에 HDD를 가상 메모리로 사용한다.
실제 메모리보다 하드디스크를 사용하기 때문에 속도 면에서는 떨어진다.</p>
<ul>
<li>dd 명령어를 통해 swap 메모리를 할당한다.<blockquote>
<p>sudo dd if=/dev/zero of=/swapfile bs=128M count=16</p>
</blockquote>
</li>
</ul>
<p>128Mb 씩 16개의 공간을 만든다 즉, 2GB 정도 차지하는 것이다.</p>
<ul>
<li><p>스왑 파일에 대한 읽기 및 쓰기 권한을 업데이트 한다.</p>
<blockquote>
<p>sudo chmod 600 /swapfile</p>
</blockquote>
</li>
<li><p>Linux 스왑 영역을 설정한다.</p>
<blockquote>
<p>sudo mkswap /swapfile</p>
</blockquote>
</li>
<li><p>스왑 공간에 스왑 파일을 추가하여 스왑 파일을 즉시 사용할 수 있도록 만든다.</p>
<blockquote>
<p>sudo swapon /swapfile</p>
</blockquote>
</li>
<li><p>절차가 성공하였는지 확인한다.</p>
<blockquote>
<p>sudo swapon -s</p>
</blockquote>
</li>
<li><p>/etc/fstab 파일을 편집하여 부팅 시 스왑 파일을 활성화한다.</p>
<blockquote>
<p>sudo vi /etc/fstab</p>
</blockquote>
</li>
</ul>
<p>마지막에 아래 내용을 새로 추가하고 저장해준다.</p>
<blockquote>
<p>/swapfile swap swap defaults 0 0</p>
</blockquote>
<p>free 명령어를 통해 메모리 상태를 체크해본다.</p>
<ul>
<li>전체적인 실행 절차
<img alt="" src="https://velog.velcdn.com/images/jelog_131/post/fa9dd961-348e-4d3a-8b4a-08bc6095e4f8/image.png" /></li>
</ul>
<p>스왑 메모리를 적용하고 다시 빌드를 해보니 잘 작동하였고 무사히 서버를 실행시킬 수 있었다.</p>
<h4 id="또-다른-방법">또 다른 방법</h4>
<p>프리티어의 EC2의 경우 메모리가 부족하기 때문에 로컬 환경에서 (작업하는 PC) 프로젝트를 빌드하고 jar 파일만 넘겨주는 방법을 해볼 수도 있겠다.</p>
<p>출처 :
<a href="https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-memory-swap-file/">https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-memory-swap-file/</a></p>
<p><a href="https://junuuu.tistory.com/346">https://junuuu.tistory.com/346</a></p>