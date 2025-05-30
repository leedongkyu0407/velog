<h1 id="☁️-presigned-url-이번엔-aws-sdk-v2로-도입한-이유">☁️ Presigned URL, 이번엔 AWS SDK v2로 도입한 이유</h1>
<h2 id="🎯-사용자-캐릭터-공모전">🎯 사용자 캐릭터 공모전</h2>
<p><a href="https://www.solve-nyang.com">솔브냥</a>에서는 유저가 직접 만든 캐릭터 이미지를 제출하고, 다른 유저들이 하루에 한 번 투표할 수 있는 <strong>참여형 공모전 기능</strong>을 만들었습니다.</p>
<p>그 중 &quot;이미지 제출&quot;은 현재 서비스 관점에서는 부하가 크진 않았지만, <strong>앞으로의 확장성과 구조 안정성을 고려해</strong> AWS S3 Presigned URL 방식을 도입하기로 했습니다.</p>
<hr />
<h3 id="❓-presigned-url이란">❓ Presigned URL이란?</h3>
<p>Presigned URL은 <strong>AWS S3의 일시적인 업로드/다운로드 권한이 포함된 URL</strong>입니다. 서버가 S3 객체에 대한 접근 권한을 미리 생성해서 클라이언트에 전달하고, 클라이언트는 해당 URL로 <strong>S3에 직접 파일을 업로드</strong>할 수 있습니다.</p>
<h4 id="🔄-presigned-url-vs-multipartform-data-방식">🔄 Presigned URL vs Multipart/form-data 방식</h4>
<table>
  <thead>
    <tr>
      <th>항목</th>
      <th>Multipart/form-data 방식</th>
      <th>Presigned URL 방식</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><strong>업로드 흐름</strong></td>
      <td style="border: 1px solid #ddd; padding: 8px;">클라이언트 → 서버 → S3 or DB(저장소)</td>
      <td style="border: 1px solid #ddd; padding: 8px;">클라이언트 → S3 (서버는 URL만 생성)</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><strong>서버 부하</strong></td>
      <td style="border: 1px solid #ddd; padding: 8px;">높음 (파일 수신 및 전송 처리)</td>
      <td style="border: 1px solid #ddd; padding: 8px;">낮음 (파일 직접 수신 안 함)</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><strong>속도</strong></td>
      <td style="border: 1px solid #ddd; padding: 8px;">네트워크 상황에 따라 지연 가능</td>
      <td style="border: 1px solid #ddd; padding: 8px;">S3에 바로 업로드로 빠름</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><strong>보안 설정</strong></td>
      <td style="border: 1px solid #ddd; padding: 8px;">서버가 제어</td>
      <td style="border: 1px solid #ddd; padding: 8px;">URL 유효 시간, MIME 타입 등으로 제어</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><strong>확장성</strong></td>
      <td style="border: 1px solid #ddd; padding: 8px;">트래픽 증가 시 병목 위험</td>
      <td style="border: 1px solid #ddd; padding: 8px;">유저 수 증가에도 안정적</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 8px;"><strong>적합한 상황</strong></td>
      <td style="border: 1px solid #ddd; padding: 8px;">소규모 파일 처리, 간단한 백오피스 등</td>
      <td style="border: 1px solid #ddd; padding: 8px;">대용량/다수 유저 업로드, 외부 공개 업로드 등</td>
    </tr>
  </tbody>
</table>

<p>한 줄 요약하면</p>
<blockquote>
<p>✅ <code>multipart/form-data</code>: 서버가 파일을 직접 받고 저장<br />✅ <code>presigned URL</code>: 서버는 URL만 발급, 클라이언트가 직접 S3에 업로드</p>
</blockquote>
<hr />
<h3 id="🔁-presigned-url의-get-vs-put-차이">🔁 Presigned URL의 GET vs PUT 차이</h3>
<p>Solvenyang에서는 단순히 이미지를 업로드하는 것뿐만 아니라, <strong>투표 페이지에서 이미지를 사용자에게 보여주기 위해 Presigned URL을 통한 다운로드(GET 요청)도 함께 사용</strong>했습니다.</p>
<p>Presigned URL은 업로드와 다운로드 모두에 사용되지만, 실제 내부 요청 방식은 다릅니다.</p>
<ul>
<li><p><strong>이미지 업로드(PUT 요청)</strong>: 클라이언트가 서버에서 발급받은 Presigned URL로 <code>PUT</code> 요청을 보내 직접 파일을 S3에 업로드합니다.</p>
</li>
<li><p><strong>이미지 다운로드(GET 요청)</strong>: 클라이언트는 Presigned URL로 <code>GET</code> 요청을 보내 S3에서 파일을 가져옵니다.</p>
</li>
</ul>
<blockquote>
<p>이처럼 클라이언트는 각각의 목적에 따라 다른 HTTP 메서드를 사용하는 Presigned URL을 사용하며,
서버는 <code>PutObjectPresignRequest</code> 또는 <code>GetObjectPresignRequest</code>를 이용해 각각 생성해야 합니다.</p>
</blockquote>
<hr />
<h3 id="⚙️-sdk-v2-presigned-url-발급-예시">⚙️ SDK v2 Presigned URL 발급 예시</h3>
<h4 id="🟩-업로드용-put">🟩 업로드용 (PUT)</h4>
<p>업로드 URL을 생성할 때는 <code>PUT</code> 요청을 위한 Presigned URL을 사용합니다.</p>
<pre><code class="language-java">public String generatePresignedUrlForUpload(String originalFilename, String contentType) {
    String fileKey = &quot;contest/&quot; + UUID.randomUUID() + &quot;-&quot; + originalFilename;

    PutObjectRequest putObjectRequest = PutObjectRequest.builder()
            .bucket(bucket)
            .key(fileKey)
            .contentType(contentType)
            .build();

    PutObjectPresignRequest presignRequest = PutObjectPresignRequest.builder()
            .signatureDuration(Duration.ofMinutes(10))
            .putObjectRequest(putObjectRequest)
            .build();

    return s3Presigner.presignPutObject(presignRequest).url().toString();
}</code></pre>
<h4 id="🟦-다운로드용-get">🟦 다운로드용 (GET)</h4>
<p>투표 페이지에서 이미지를 보여줄 때는 <code>GET</code> 요청을 위한 Presigned URL을 사용합니다.</p>
<pre><code class="language-java">public String generatePresignedUrlForDownload(String fileKey) {
    GetObjectRequest getObjectRequest = GetObjectRequest.builder()
            .bucket(bucket)
            .key(fileKey)
            .build();

    GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
            .signatureDuration(Duration.ofMinutes(10))
            .getObjectRequest(getObjectRequest)
            .build();

    return s3Presigner.presignGetObject(presignRequest).url().toString();
}</code></pre>
<p>Presigner 객체는 스프링 빈으로 주입받고, 서버 종료 시 <code>@PreDestroy</code>로 닫아주는 패턴도 함께 적용했습니다.
검색해보니 SDK v2는 구조적으로 <strong>Presigner 객체가 명확히 분리</strong>되어 있어
기능 확장이나 테스트에도 유리한 구조를 제공한다고 합니다.</p>
<hr />
<h3 id="🛠️-presigner란-왜-닫아줘야-할까">🛠️ Presigner란? 왜 닫아줘야 할까?</h3>
<p><code>S3Presigner</code>는 AWS SDK v2에서 <strong>Presigned URL을 생성하는 역할을 담당하는 객체</strong>입니다. 내부적으로 HTTP 클라이언트, 인증 정보, 서명 정보 등을 포함하고 있으며, 네트워크 리소스를 관리합니다.</p>
<p>서버가 종료될 때 JVM이 대부분의 리소스를 정리하긴 하지만, <strong>예측 가능한 자원 관리를 위해 명시적으로 닫아주는 것이 안전하고 권장되는 방법</strong>입니다.</p>
<p>특히 Spring 환경에서는 <code>@PreDestroy</code> 어노테이션을 통해 이 과정을 자동화할 수 있습니다.</p>
<h4 id="☑️-안전하게-닫는-코드-예시">☑️ 안전하게 닫는 코드 예시</h4>
<pre><code class="language-java">@Configuration
public class S3Config {
    private S3Presigner s3Presigner;

    @Bean
    public S3Presigner s3Presigner() {
        AwsBasicCredentials awsCredentials = AwsBasicCredentials.create(accessKey, secretKey);
        s3Presigner = S3Presigner.builder()
                .region(Region.of(region))
                .credentialsProvider(StaticCredentialsProvider.create(awsCredentials))
                .build();
        return s3Presigner;
    }

    @PreDestroy
    public void closePresigner() {
        if (s3Presigner != null) {
            s3Presigner.close();
        }
    }
}</code></pre>
<blockquote>
<p>서버가 종료될 때 <code>@PreDestroy</code>를 이용해 <code>S3Presigner</code>를 안전하게 닫아 리소스를 정리합니다.</p>
</blockquote>
<hr />
<h3 id="🔄-전체-업로드-및-저장-흐름">🔄 전체 업로드 및 저장 흐름</h3>
<ol>
<li>클라이언트가 <code>filename</code>, <code>contentType</code>을 포함해 presigned URL 요청</li>
<li>서버가 업로드할 S3 Key를 생성하고 Presigned URL 발급</li>
<li>클라이언트가 해당 URL로 S3에 직접 업로드</li>
<li>업로드 성공 후, 클라이언트가 <code>storedFilename</code>을 서버에 전송</li>
<li>서버는 해당 key를 DB에 저장해 이미지 메타데이터 관리</li>
<li>추후 이미지 조회 시 해당 key를 기반으로 presigned URL 발급하여 접근 가능</li>
</ol>
<blockquote>
<p>서버는 실제 파일을 저장하지 않지만, <strong>파일의 경로(key)는 반드시 DB에 저장</strong>해야 관리가 가능합니다.</p>
</blockquote>
<hr />
<h3 id="🧳-과거-경험-여행지-쇼츠-프로젝트에선-sdk-v1-사용">🧳 과거 경험: 여행지 쇼츠 프로젝트에선 SDK v1 사용</h3>
<p>예전 &quot;사용자가 여행지에서 쇼츠 영상을 올리는 프로젝트&quot;를 진행할 때는 S3 Presigned URL 생성을 위해 <strong>AWS SDK v1</strong>을 사용했습니다.</p>
<p>하지만 SDK v1은 <a href="https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/welcome.html">공식적으로 deprecated 예정</a>이고, API도 복잡하며 유지보수 부담도 있어 이번에는 <strong>AWS SDK v2 기반으로 Presigned URL</strong>을 구현했습니다.</p>
<hr />
<h4 id="🗳️-추가-하루에-한-번만-투표하기---redis-사용">🗳️ 추가: 하루에 한 번만 투표하기 - Redis 사용</h4>
<p>공모전 투표는 유저당 하루 한 번만 가능하도록 Redis를 이용해 제어했습니다.</p>
<pre><code class="language-java">String key = &quot;vote:&quot; + memberId + &quot;:&quot; + LocalDate.now();
Boolean isFirstVote = redisTemplate.opsForValue()
    .setIfAbsent(key, &quot;voted&quot;, Duration.ofDays(1).toSeconds(), TimeUnit.SECONDS);

if (Boolean.FALSE.equals(isFirstVote)) {
    throw new HttpClientErrorException(HttpStatus.BAD_REQUEST, &quot;오늘은 더 이상 투표할 수 없습니다.&quot;);
}</code></pre>
<blockquote>
<p><code>setIfAbsent + TTL</code> 조합으로 간단하고 효율적인 투표 제한 구현</p>
</blockquote>
<hr />
<h3 id="✅-왜-지금-presigned-url-구조를-택했나">✅ 왜 지금 Presigned URL 구조를 택했나?</h3>
<ul>
<li>서버가 직접 파일 받으면 → 확장성 떨어지고, 보안/트래픽 리스크</li>
<li>클라이언트가 직접 S3에 업로드하면 → 서버 부하 ↓, 속도 ↑</li>
<li>비록 지금은 가볍지만, 미래를 고려한 구조적 설계</li>
</ul>
<blockquote>
<p>한두 장의 이미지라면 지금은 문제 없지만,
사용자가 많아지고, 영상 등 다른 미디어로 확장될 가능성을 생각하면
지금 구조를 다져두는 게 훨씬 현명하다고 판단했습니다.</p>
</blockquote>
<hr />
<h3 id="🤔-회고">🤔 회고</h3>
<p>Presigned URL은 단순한 성능 개선 기술이 아니라, <strong>서버-클라이언트 역할 분리, 보안성, 확장성</strong>을 고려한 설계였습니다.
당장은 캐릭터 이미지가 서버에 큰 부담을 주지 않더라도, 유저 수 증가나 기능 확장을 고려할 때 <strong>미리 안정적인 구조를 준비하고자 하는 판단</strong>이었습니다.
또한 AWS SDK v2를 처음 적용하면서, <strong>의존성 선택이 미래 유지보수에 얼마나 큰 영향을 미치는지도 실감</strong>했습니다.</p>
<blockquote>
<p>이 경험은 단순히 기능 구현을 넘어서, <strong>더 나은 아키텍처를 설계하는 개발자로서의 사고방식을 다지게 해준 계기</strong>였습니다.</p>
</blockquote>