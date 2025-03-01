<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/0ca54c63-d480-4824-a43a-12800fefd5df/image.png" /></p>
<p>오늘은 Spring Boot에서 AWS S3에 이미지를 업로드하고 삭제하는 방법을 정리해보려고 한다. 먼저 AWS에서 해줘야 할 것들이 있다.</p>
<h1 id="aws-s3-버킷-생성">AWS S3 버킷 생성</h1>
<p>버킷 만들기를 클릭해준다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/2ffe0cd0-c61e-4809-9811-8f8cccfd9fe2/image.png" /></p>
<p>버킷 이름을 지어주고 객체 소유권은 ACL 활성화됨으로 설정해주어야 Spring Boot 어플리케이션과 통신이 가능하다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/1dc71def-b0d6-4a3b-b814-feb3fa964097/image.png" /></p>
<p>퍼블릭 액세스 차단 설정도 디폴트로 '차단' 되어 있는데 이 부분도 체크 해제해준다.
그 아래 설정들은 디폴트로 하고 '버킷 만들기' 버튼을 눌러 버킷을 생성해준다.</p>
<h1 id="버킷-권한-설정">버킷 권한 설정</h1>
<p>생성된 버킷으로 들어가면 권한 탭을 볼 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/9fa644aa-6ecb-44d0-8231-735caa38968c/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/744ac4d7-c2ca-4c37-80c6-c918feccd6cc/image.png" /></p>
<p>아랫쪽으로 조금 내리다 보면 버킷 정책이 보인다. 편집 버튼을 누르고</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/3757b382-524b-4a88-a4ef-547a05456d1d/image.png" /></p>
<p>우측 상단에 보이는 '정책 생성기'로 정책을 생성하고 나오는 JSON을 붙여넣어도 되고, S3 버킷 정책은 구글링해도 바로 나올 만큼 검색 결과가 많아서, 찾아서 복사해 와도 괜찮지만 일단은 '정책 생성기'로 진행해봤다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/bb61ae09-0055-4bb4-94c9-a6331c03dc16/image.png" /></p>
<p>위 쪽에 정책 타입을 'S3 Bucket Policy'로 선택해준다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/7133ac00-3275-4923-af31-e7bd3e699a17/image.png" /></p>
<p>Principal에는 모두 허용한다는 의미로 '*'를 입력해주고 Actions에서 GetObject 옵션을 추가해 주었다. 다른 권한이 필요하다면 찾아보고 추가해주면 된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/12de5655-0763-4fd2-945b-13748ab28549/image.png" /></p>
<p>그리고 아래에 ARN에는 만든 버킷의 ARN을 넣어주면 된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/1a64364a-7e02-4cd9-8b21-4c88955f2c92/image.png" /></p>
<p>여기까지 하고 Step3의 'Generate Policy'를 누르게 되면 JSON을 받을 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/199a7912-0376-404d-bfc7-d1d4b963a433/image.png" /></p>
<p>이 부분을 복사하여 좀 전의 버킷 정책에 붙여넣는다. 이렇게 하면 AWS S3의 설정은 끝이났다. 아직 스프링 부트 프로젝트에서 S3를 연동하고 코드를 작성해줘야 할 부분이 남아있다.</p>
<hr />
<h1 id="spring-boot-설정">Spring Boot 설정</h1>
<p>build.gradle에 AWS 관련된 의존성을 추가해줘야 한다.</p>
<blockquote>
<p>implementation 'org.springframework.cloud:spring-cloud-starter-aws:2.2.6.RELEASE'</p>
</blockquote>
<p>다음은 application.yml or application.properties 파일 설정이다.</p>
<pre><code class="language-yaml">loud:
  aws:
    credentials: # IAM으로 생성한 시크릿키 정보를 입력한다.
      access-key: 
      secret-key: 
    S3:
      bucket: # bucket 이름을 설정한다.
    region:
      static: ap-northeast-2 # bucket이 위치한 AWS 리전을 설정한다.
    stack:
      auto: false # 자동 스택 생성 기능 사용여부를 설정한다. (자동 스택 생성: 애플리케이션이 실행, 배포될 때 인프라 리소스를 자동으로 생성하고 설정하는 것)</code></pre>
<p>여기서 access-key와 secret-key는 절대로 외부에 공개되면 안된다. 환경 변수로 설정해서 가져오거나 .gitignore로 해당 파일을 제외하고 깃에 올려야 한다.</p>
<p>만약 EC2를 사용하고 있다면
AWS에서 권장하는 방법은 환경 변수를 직접 설정하지 않고 IAM Role을 사용하는 것이다.</p>
<ul>
<li>EC2 인스턴스에 필요한 최소 권한(AmazonS3FullAccess 등)을 가진 IAM Role을 생성</li>
<li>EC2에 IAM Role 연결</li>
<li>SpringBoot 설정</li>
</ul>
<p>AWS SDK는 EC2에서 실행 중인 경우 자동으로 IAM Role의 자격 증명을 사용한다.</p>
<pre><code class="language-java">@Bean
public S3Client s3Client() {
    return S3Client.builder()
            .region(Region.of(&quot;your-region&quot;)) // 리전을 설정하세요
            .build(); // IAM Role 자동 사용
}</code></pre>
<p>EC2가 아닌 로컬환경이라면 AWS CLI를 설치한 후 자격 증명 파일을 설정해주면 된다.</p>
<ul>
<li>AWS CLI 설치</li>
<li>AWS 자격 증명 파일 생성<blockquote>
<p>aws configure</p>
</blockquote>
</li>
</ul>
<p>프롬프트에 따라 다음 정보를 입력한다.</p>
<ul>
<li>AWS Access Key ID:엑세스키</li>
<li>AWS Secret Access Key:시크릿키</li>
<li>Default region name:지역</li>
</ul>
<p>자격 증명 정보는 ~/.aws/credentials 파일에 저장된다.
Spring Boot는 Default Credentials Provider Chain을 사용하므로 환경에 따라 자동으로 자격 증명을 선택한다.</p>
<ul>
<li>EC2 환경 : EC2 Instance Profile을 자동으로 사용.</li>
<li>로컬 환경 : 환경 변수 또는 ~/.aws/credentials 파일을 자동으로 사용.</li>
</ul>
<p>이렇게 설정해주면 EC2, 로컬 모두 코드는 위의 코드와 동일하게 사용할 수 있다.</p>
<p>참고 : <a href="https://velog.io/@chrkb1569/AWS-S3-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-IAM-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%A4%EC%A0%95">https://velog.io/@chrkb1569/AWS-S3-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-IAM-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%A4%EC%A0%95</a></p>
<h2 id="코드-작성">코드 작성</h2>
<p>설정이 끝났다면 S3에 이미지를 업로드하고 삭제할 수 있도록 코드를 작성해주면 된다.
먼저 @Configuration을 통해 AmazonS3를 빈으로 등록해줘야 한다.</p>
<pre><code class="language-java">package com.jhpark.awss3test.config;

import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AwsS3Config {

    @Bean
    public AmazonS3 amazonS3() {
        BasicAWSCredentials awsCreds = new BasicAWSCredentials(&quot;my-accessKey&quot;, &quot;my-secretKey&quot;); // 엑세스키, 시크릿키
        return AmazonS3ClientBuilder.standard()
                .withRegion(&quot;ap-northeast-2&quot;)
                .withCredentials(new AWSStaticCredentialsProvider(awsCreds))
                .build();
    }
}</code></pre>
<ul>
<li><p>Controller</p>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/api/s3&quot;)
@RequiredArgsConstructor
public class S3Controller {

  private final S3ImageService s3ImageService;

  @PostMapping(&quot;/upload&quot;)
  public ResponseEntity&lt;String&gt; uploadImage(@RequestPart(value = &quot;image&quot;) MultipartFile image) {
      String imageUrl = s3ImageService.uploadImage(image);
      return ResponseEntity.ok(imageUrl);
  }

  @DeleteMapping(&quot;/delete&quot;)
  public ResponseEntity&lt;Void&gt; deleteImage(@RequestParam String imageUrl) {
      s3ImageService.deleteImage(imageUrl);
      return ResponseEntity.ok().build();
  }
}</code></pre>
</li>
<li><p>Service</p>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class S3ImageService {

  private final AmazonS3 amazonS3;
  private final String bucketName = &quot;bucketName&quot;; //버킷 이름

  public String uploadImage(MultipartFile image) {
      if (image.isEmpty()) {
          throw new IllegalArgumentException(&quot;File is empty&quot;);
      }

      String originalFilename = image.getOriginalFilename();
      String extension = originalFilename.substring(originalFilename.lastIndexOf(&quot;.&quot;) + 1);
      String fileName = &quot;image/&quot; + UUID.randomUUID().toString() + &quot;.&quot; + extension;

      try {
          byte[] bytes = toByteArray(image.getInputStream());
          ObjectMetadata metadata = new ObjectMetadata();
          metadata.setContentType(image.getContentType());
          metadata.setContentLength(bytes.length);

          amazonS3.putObject(new PutObjectRequest(bucketName, fileName, new ByteArrayInputStream(bytes), metadata)
                  .withCannedAcl(CannedAccessControlList.PublicRead));
      } catch (IOException e) {
          throw new RuntimeException(&quot;Failed to upload image&quot;, e);
      }

      return amazonS3.getUrl(bucketName, fileName).toString();
  }

  public void deleteImage(String imageUrl) {
      String key = imageUrl.substring(imageUrl.indexOf(&quot;image/&quot;));
      amazonS3.deleteObject(new DeleteObjectRequest(bucketName, key));
  }

  private byte[] toByteArray(InputStream input) throws IOException {
      ByteArrayOutputStream buffer = new ByteArrayOutputStream();
      byte[] data = new byte[1024];
      int nRead;
      while ((nRead = input.read(data, 0, data.length)) != -1) {
          buffer.write(data, 0, nRead);
      }
      return buffer.toByteArray();
  }
}</code></pre>
</li>
</ul>