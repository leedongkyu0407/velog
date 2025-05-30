<h1 id="presigned-url을-위한-완벽한-s3-버킷-설정-가이드">Presigned URL을 위한 완벽한 S3 버킷 설정 가이드</h1>
<p>웹 애플리케이션에서 파일 업로드/다운로드를 구현할 때, S3 Presigned URL은 매우 효율적인 솔루션입니다. 서버의 부하를 줄이면서도 안전하게 파일을 처리할 수 있기 때문입니다. 하지만 올바른 S3 버킷 설정이 없다면 제대로 동작하지 않을 수 있습니다.</p>
<p>이 글에서는 Presigned URL을 위한 최적의 S3 버킷 설정 방법을 단계별로 알아보겠습니다.</p>
<h2 id="1-presigned-url이란">1. Presigned URL이란?</h2>
<p>Presigned URL은 AWS가 미리 서명한 임시 URL로, 특정 시간 동안만 유효한 S3 접근 권한을 제공합니다.</p>
<p><strong>동작 방식:</strong></p>
<ol>
<li><strong>백엔드 서버</strong>가 AWS SDK로 presigned URL 생성 (서명)</li>
<li><strong>프론트엔드</strong>가 그 URL을 받아서 브라우저에서 직접 S3와 통신</li>
<li>서버를 거치지 않고 클라이언트 ↔ S3 직접 통신</li>
</ol>
<p>이 방식의 장점은 서버 리소스 절약, 빠른 전송 속도, 그리고 임시 권한을 통한 보안성입니다.
Presigned URL에 대한 상세 설명은 <a href="https://velog.io/@donggyu47/Presigned-URL%EC%9D%B4-%EB%AD%90%EA%B8%B8%EB%9E%98-%EC%BA%90%EB%A6%AD%ED%84%B0-%EA%B3%B5%EB%AA%A8%EC%A0%84-%EA%B8%B0%EB%8A%A5%EC%97%90-%EC%A0%81%EC%9A%A9%ED%95%9C-%EC%8B%A4%EC%A0%84-%EC%82%AC%EB%A1%80">다른 글</a>에서 설명했기에 이 글의 주제로 바로 넘어가겠습니다.</p>
<h2 id="2-s3-버킷-생성하기">2. S3 버킷 생성하기</h2>
<h3 id="2-1-버킷-생성-및-기본-설정">2-1. 버킷 생성 및 기본 설정</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/b2a8d3c5-6021-4fc0-89ba-23c54183f656/image.png" /></p>
<p>AWS 콘솔에서 S3 서비스로 이동하여 새 버킷을 생성합니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/f414a41e-38e3-4a0d-beef-24ceaa9a27ce/image.png" /></p>
<p><strong>기본 설정:</strong></p>
<ul>
<li><strong>버킷 이름</strong>: 전역적으로 고유한 이름 (예: <code>my-presigned-bucket</code>)</li>
<li><strong>AWS 리전</strong>: 애플리케이션과 가까운 리전 선택</li>
</ul>
<h3 id="2-2-객체-소유권-설정">2-2. 객체 소유권 설정</h3>
<p><strong>ACL 비활성화됨 (권장)</strong> 을 선택합니다.</p>
<p>이 설정의 장점:</p>
<ul>
<li>버킷 소유자가 모든 업로드된 객체를 소유</li>
<li>ACL 대신 IAM 정책과 버킷 정책으로만 권한 관리</li>
<li>더 단순하고 안전한 권한 관리</li>
</ul>
<h3 id="2-3-퍼블릭-액세스-차단-설정-⭐-중요">2-3. 퍼블릭 액세스 차단 설정 ⭐ 중요</h3>
<p><strong>모든 퍼블릭 액세스 차단</strong> 옵션을 체크합니다:
개발 과정에서는 편의성을 위해 모든 퍼블릭 액세스 차단을 해제하고 생성한 뒤 추후에 서비스 배포시 변경하는 방법도 추천드립니다. (나중에 변경 가능하기에 편하신 대로 선택하셔도 됩니다.)</p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/41f81248-d12a-4d7d-8e6d-8f5eba93283a/image.png" /></p>
<ul>
<li>✅ 새 ACL을 통해 부여된 버킷 및 객체에 대한 퍼블릭 액세스 차단</li>
<li>✅ 임의의 ACL을 통해 부여된 버킷 및 객체에 대한 퍼블릭 액세스 차단</li>
<li>✅ 새 퍼블릭 버킷 또는 액세스 지점 정책을 통해 부여된 버킷 및 객체에 대한 퍼블릭 액세스 차단</li>
<li>✅ 임의의 퍼블릭 버킷 또는 액세스 지점 정책을 통해 부여된 버킷 및 객체에 대한 퍼블릭 및 교차 계정 액세스 차단</li>
</ul>
<p><strong>중요한 점:</strong> Presigned URL은 서명된 요청이므로 퍼블릭 액세스 차단과 관계없이 정상 동작합니다.</p>
<h3 id="2-4-버킷-버전-관리-설정">2-4. 버킷 버전 관리 설정</h3>
<p><strong>비활성화 권장</strong> (presigned URL 용도에서는 불필요하며 비용만 증가)</p>
<p><strong>버전 관리 활성화 시:</strong></p>
<ul>
<li>같은 파일명으로 업로드할 때 기존 파일을 덮어쓰지 않고 새 버전으로 저장</li>
<li>각 버전마다 고유한 Version ID 생성</li>
<li>실수로 삭제/수정해도 이전 버전 복구 가능</li>
</ul>
<p><strong>Presigned URL 용도에서 비활성화 권장 이유:</strong></p>
<ul>
<li>사용자가 파일 업로드할 때마다 새 버전이 쌓임</li>
<li>스토리지 비용 증가 (모든 버전이 과금됨)</li>
<li>일반적으로 최신 파일만 필요한 경우가 많음</li>
<li>관리 복잡성 증가</li>
</ul>
<p><strong>활성화가 유용한 경우:</strong></p>
<ul>
<li>문서 버전 관리가 중요한 시스템</li>
<li>백업/복구가 중요한 데이터</li>
<li>실수 방지가 중요한 환경</li>
</ul>
<h3 id="2-5-기본-암호화-설정">2-5. 기본 암호화 설정</h3>
<p><strong>Amazon S3 관리형 키(SSE-S3) 권장</strong> (추가 비용 없이 자동 암호화)</p>
<p><strong>암호화 옵션 비교:</strong></p>
<table>
<thead>
<tr>
<th>암호화 방식</th>
<th>특징</th>
<th>비용</th>
<th>권장도</th>
</tr>
</thead>
<tbody><tr>
<td>SSE-S3</td>
<td>AWS가 키 완전 관리</td>
<td>무료</td>
<td>⭐⭐⭐</td>
</tr>
<tr>
<td>SSE-KMS</td>
<td>세밀한 키 관리 가능</td>
<td>추가 비용</td>
<td>⭐⭐</td>
</tr>
<tr>
<td>SSE-C</td>
<td>직접 키 제공</td>
<td>무료</td>
<td>⭐</td>
</tr>
</tbody></table>
<p><strong>Presigned URL 용도에서는 SSE-S3가 최적:</strong></p>
<ul>
<li>추가 설정/비용 없음</li>
<li>presigned URL이 자동으로 암호화 처리</li>
<li>투명하게 동작 (클라이언트는 암호화 인식 불필요)</li>
<li>충분한 보안성 제공</li>
</ul>
<h2 id="3-iam-사용자-생성-및-권한-설정">3. IAM 사용자 생성 및 권한 설정</h2>
<p>S3 Presigned URL을 생성하려면 백엔드 애플리케이션이 S3에 접근할 수 있는 권한이 필요합니다.</p>
<h3 id="3-1-iam-사용자-생성">3-1. IAM 사용자 생성</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/b0c0fbf1-888c-45dd-8679-cc9ebe67d698/image.png" /></p>
<p><strong>IAM 콘솔 → 사용자 → 사용자 생성</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/c0099429-3b32-47e5-9c47-d12697c5277f/image.png" /></p>
<ul>
<li><strong>사용자 이름</strong>: <code>s3-presigned-user</code> (또는 원하는 이름)</li>
<li><strong>AWS 액세스 유형</strong>: 프로그래밍 방식 액세스 선택</li>
</ul>
<h3 id="3-2-권한-설정-방식-선택하기">3-2. 권한 설정 방식 선택하기</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/f88b56ee-06f6-4014-a152-ead7dc39128b/image.png" /></p>
<p>AWS IAM에서 사용자 권한을 설정하는 방법은 크게 3가지입니다:</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>적용 상황</th>
<th>장점</th>
<th>단점</th>
</tr>
</thead>
<tbody><tr>
<td><strong>그룹에 사용자 추가</strong> ⭐</td>
<td>여러 사용자에게 동일 권한</td>
<td>관리 효율성, 일관성</td>
<td>초기 그룹 설정 필요</td>
</tr>
<tr>
<td><strong>권한 복사</strong></td>
<td>기존 사용자와 동일 권한</td>
<td>빠른 설정</td>
<td>불필요한 권한 복사 위험</td>
</tr>
<tr>
<td><strong>직접 정책 연결</strong></td>
<td>단일 사용자, 테스트 용도</td>
<td>세밀한 제어</td>
<td>사용자 증가 시 관리 복잡</td>
</tr>
</tbody></table>
<blockquote>
<p><strong>권장</strong>: 실무에서는 <strong>그룹에 사용자 추가</strong> 방식이 가장 보편적이고 효율적입니다.</p>
</blockquote>
<h3 id="3-3-s3-정책-선택하기">3-3. S3 정책 선택하기</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/67cf3880-a326-4429-8739-555d4027a410/image.png" /></p>
<p><strong>직접 정책 연결</strong>을 선택했다면 <code>amazons3</code>를 검색하여 적절한 정책을 선택합니다.</p>
<p><strong>Presigned URL 목적별 필요 권한:</strong></p>
<table>
<thead>
<tr>
<th>사용 목적</th>
<th>필요한 S3 권한</th>
<th>추천 정책</th>
</tr>
</thead>
<tbody><tr>
<td>파일 업로드 (PUT)</td>
<td><code>s3:PutObject</code></td>
<td><code>AmazonS3FullAccess</code></td>
</tr>
<tr>
<td>파일 다운로드 (GET)</td>
<td><code>s3:GetObject</code></td>
<td><code>AmazonS3ReadOnlyAccess</code></td>
</tr>
<tr>
<td>파일 삭제 (DELETE)</td>
<td><code>s3:DeleteObject</code></td>
<td><code>AmazonS3FullAccess</code></td>
</tr>
<tr>
<td>종합적 사용</td>
<td>위 모든 권한</td>
<td><code>AmazonS3FullAccess</code> ⭐</td>
</tr>
</tbody></table>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/44ccf024-37d3-4a88-81ea-121baf6fcb9c/image.png" /></p>
<p>업로드, 다운로드, 삭제를 모두 사용할 예정이라면 <code>AmazonS3FullAccess</code> 권한을 선택합니다.</p>
<h3 id="3-4-커스텀-정책-생성-선택사항">3-4. 커스텀 정책 생성 (선택사항)</h3>
<p>더 세밀한 권한 제어가 필요하다면 커스텀 정책을 생성할 수 있습니다:</p>
<p><strong>IAM 콘솔 → 정책 → 정책 생성 → JSON 탭</strong></p>
<pre><code class="language-json">{
    &quot;Version&quot;: &quot;2012-10-17&quot;,
    &quot;Statement&quot;: [
        {
            &quot;Effect&quot;: &quot;Allow&quot;,
            &quot;Action&quot;: [
                &quot;s3:GetObject&quot;,
                &quot;s3:PutObject&quot;,
                &quot;s3:DeleteObject&quot;,
                &quot;s3:GetBucketLocation&quot;
            ],
            &quot;Resource&quot;: [
                &quot;arn:aws:s3:::your-bucket-name&quot;,
                &quot;arn:aws:s3:::your-bucket-name/*&quot;
            ]
        }
    ]
}</code></pre>
<p><strong>정책 이름</strong>: <code>S3-PresignedURL-Policy</code></p>
<h3 id="3-5-액세스-키-생성">3-5. 액세스 키 생성</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/4a362720-3cf3-4164-b7bf-63224dc3cf79/image.png" /></p>
<p>Presigned URL을 사용하는 백엔드 서버(Spring Boot 등)가 <strong>Amazon EC2</strong>, <strong>AWS Lambda</strong>, <strong>ECS</strong> 등 <strong>AWS 컴퓨팅 서비스</strong>에서 실행될 경우, 가장 적절한 선택지는 다음과 같습니다:</p>
<ul>
<li>✅ <strong>AWS 컴퓨팅 서비스에서 실행되는 애플리케이션</strong><ul>
<li>Amazon EC2, Amazon ECS, AWS Lambda와 같은 AWS 내 컴퓨팅 서비스에서 실행되는 애플리케이션이 AWS 리소스에 액세스할 수 있도록 키를 사용하는 방식입니다.</li>
<li>Presigned URL을 생성하는 서버가 AWS 내부에서 운영된다면 이 항목이 가장 안전하며 AWS에서도 권장합니다.</li>
</ul>
</li>
</ul>
<h4 id="액세스-키-생성-화면-예시">액세스 키 생성 화면 예시</h4>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/70026c54-2634-462e-95f0-3dd0be777562/image.png" /></p>
<hr />
<h4 id="기타-사용-사례-설명-및-권장-여부">기타 사용 사례 설명 및 권장 여부</h4>
<table>
<thead>
<tr>
<th>사용 사례 항목</th>
<th>설명</th>
<th>용도 예시</th>
<th>권장 여부</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Command Line Interface(CLI)</strong></td>
<td>AWS CLI를 통해 명령어 기반으로 리소스를 조작</td>
<td>로컬 터미널에서 수동 작업</td>
<td>❌ 운영 환경에서는 지양</td>
</tr>
<tr>
<td><strong>로컬 코드</strong></td>
<td>로컬 개발환경에서 AWS SDK 사용</td>
<td>개인 개발용 테스트</td>
<td>⚠️ 테스트/학습용으로는 가능, 운영에는 부적합</td>
</tr>
<tr>
<td><strong>AWS 컴퓨팅 서비스에서 실행되는 애플리케이션</strong></td>
<td>EC2, Lambda, ECS 등에서 동작하는 서버 애플리케이션</td>
<td>✅ Presigned URL 생성 서버</td>
<td>✅ <strong>가장 권장</strong></td>
</tr>
<tr>
<td><strong>서드 파티 서비스</strong></td>
<td>외부 SaaS 도구가 AWS 리소스를 모니터링 또는 제어</td>
<td>Datadog, New Relic 등</td>
<td>❌ 외부 연동 시 별도 인증 권장</td>
</tr>
<tr>
<td><strong>AWS 외부에서 실행되는 애플리케이션</strong></td>
<td>온프레미스, 외부 데이터센터, 타 클라우드에서 실행</td>
<td>자체 서버에서 AWS 리소스 접근</td>
<td>⚠️ 가능하지만 키 유출 위험 고려 필요</td>
</tr>
<tr>
<td><strong>기타</strong></td>
<td>위에 해당하지 않는 모든 상황</td>
<td>예외적 사용 사례</td>
<td>❌ 구체적 목적이 없으면 선택 지양</td>
</tr>
</tbody></table>
<hr />
<h4 id="액세스-키-발급-절차-요약">액세스 키 발급 절차 요약</h4>
<ul>
<li>IAM 콘솔 → 사용자 선택 → <strong>보안 자격 증명</strong> 탭 → <strong>액세스 키 만들기</strong></li>
<li>생성된 액세스 키 ID와 비밀 액세스 키는 <strong>절대 외부에 노출되지 않도록 안전하게 저장</strong></li>
<li>Spring Boot와 같은 백엔드 서버에서는 <strong>환경 변수 또는 AWS SDK 설정 파일</strong>을 통해 안전하게 키를 사용</li>
</ul>
<blockquote>
<p>⚠️ 운영 환경에서는 <code>로컬 코드</code>나 <code>CLI</code> 항목을 선택하지 말고, 실제 실행 위치에 기반하여 <strong>정확한 항목을 선택</strong>해야 합니다. 다만, 이 글을 보시는 대부분은 EC2 등 AWS 컴퓨팅 서비스에서 실행되는 애플리케이션을 통해 S3를 접근할 것이라고 생각하기에 세 번 째를 가장 추천합니다.</p>
</blockquote>
<p>이 때 생성된 액세스 키의 경우 csv파일로 저장하여 꼭 기억하셔야 합니다. 또한, 외부 노출될 경우 S3 버킷에 접근할 수 있는 권한이 노출되는 것이기 때문에 프로젝트 진행 과정에서도 properties나 env 파일로 관리하여 유출되지 않도록 관리하는데 주의를 기울여 주셔야 합니다.</p>
<h2 id="4-버킷-정책-설정하기">4. 버킷 정책 설정하기</h2>
<p>퍼블릭 액세스 차단을 설정했지만, presigned URL이 동작하려면 적절한 버킷 정책이 필요합니다.</p>
<h3 id="4-1-aws-policy-generator-사용법">4-1. AWS Policy Generator 사용법</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/1acadf48-5137-4ce8-846c-cb540ce0ef95/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/b8964d35-90c6-4dd3-aa11-5954d918b510/image.png" /></p>
<p><strong>AWS Policy Generator 접속 후 설정:</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/f52fcb4e-4e0f-415e-a411-3716234254fb/image.png" /></p>
<ul>
<li><strong>Policy Type</strong>: <code>S3 Bucket Policy</code> 선택</li>
<li><strong>Effect</strong>: <code>Allow</code></li>
<li><strong>Principal</strong>: <code>*</code> (모든 사용자)</li>
<li><strong>Actions</strong>: <code>GetObject</code>, <code>PutObject</code>, <code>DeleteObject</code> 선택</li>
<li><strong>ARN</strong>: <code>arn:aws:s3:::your-bucket-name/*</code></li>
</ul>
<h3 id="4-2-생성된-정책-예시">4-2. 생성된 정책 예시</h3>
<pre><code class="language-json">{
  &quot;Version&quot;: &quot;2012-10-17&quot;,
  &quot;Id&quot;: &quot;Policy1738656245626&quot;,
  &quot;Statement&quot;: [
    {
      &quot;Sid&quot;: &quot;Stmt1738656243471&quot;,
      &quot;Effect&quot;: &quot;Allow&quot;,
      &quot;Principal&quot;: &quot;*&quot;,
      &quot;Action&quot;: [
        &quot;s3:GetObject&quot;,
        &quot;s3:PutObject&quot;,
        &quot;s3:DeleteObject&quot;
      ],
      &quot;Resource&quot;: [
        &quot;arn:aws:s3:::your-bucket-name/*&quot;
      ]
    }
  ]
}</code></pre>
<h3 id="4-3-정책-구성-요소-설명">4-3. 정책 구성 요소 설명</h3>
<ul>
<li><strong>Version</strong>: 정책 언어 버전 (항상 &quot;2012-10-17&quot;)</li>
<li><strong>Id</strong>: 정책의 고유 식별자 (선택사항)</li>
<li><strong>Sid</strong>: Statement ID, 각 문장을 구별하는 식별자</li>
<li><strong>Principal</strong>: &quot;*&quot;는 모든 사용자를 의미하지만, presigned URL의 경우 서명된 요청만 허용</li>
<li><strong>Resource</strong>: <code>/*</code>를 포함해야 객체 레벨 작업이 가능</li>
</ul>
<p><strong>중요한 점:</strong></p>
<ul>
<li><strong>버킷 정책</strong>: 외부에서 S3 버킷에 접근할 수 있는 권한을 정의</li>
<li><strong>IAM 정책</strong>: 백엔드 애플리케이션이 presigned URL을 생성할 수 있는 권한을 부여</li>
<li><strong>둘 다 설정해야 presigned URL이 정상 동작합니다!</strong></li>
</ul>
<h2 id="5-cors-설정하기-⭐-필수">5. CORS 설정하기 ⭐ 필수</h2>
<p>브라우저에서 presigned URL을 사용하려면 CORS(Cross-Origin Resource Sharing) 설정이 반드시 필요합니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/0dfe61e3-ad5c-420a-820f-652a6ba30b72/image.png" /></p>
<h3 id="5-1-cors-설정-예시">5-1. CORS 설정 예시</h3>
<pre><code class="language-json">[
  {
    &quot;AllowedHeaders&quot;: [&quot;*&quot;],
    &quot;AllowedMethods&quot;: [&quot;GET&quot;, &quot;HEAD&quot;, &quot;PUT&quot;, &quot;POST&quot;, &quot;DELETE&quot;],
    &quot;AllowedOrigins&quot;: [
      &quot;http://localhost:3000&quot;,
      &quot;http://localhost:5173&quot;,
      &quot;https://your-domain.com&quot;
    ],
    &quot;ExposeHeaders&quot;: [&quot;ETag&quot;]
  }
]</code></pre>
<h3 id="5-2-cors-설정-설명">5-2. CORS 설정 설명</h3>
<ul>
<li><strong>AllowedHeaders</strong>: 브라우저가 보낼 수 있는 헤더 (보통 &quot;*&quot; 사용)</li>
<li><strong>AllowedMethods</strong>: <ul>
<li><code>PUT/POST</code>: 파일 업로드용</li>
<li><code>GET</code>: 파일 다운로드용</li>
<li><code>DELETE</code>: 파일 삭제용</li>
</ul>
</li>
<li><strong>AllowedOrigins</strong>: 요청을 허용할 도메인 목록 (보안을 위해 구체적으로 명시)</li>
<li><strong>ExposeHeaders</strong>: 브라우저가 접근할 수 있는 응답 헤더</li>
</ul>
<h2 id="6-보안-고려사항">6. 보안 고려사항</h2>
<h3 id="6-1-최소-권한-원칙">6-1. 최소 권한 원칙</h3>
<p>버킷 정책에서 필요한 최소한의 권한만 부여합니다. 업로드만 필요하다면 <code>PutObject</code>만, 다운로드만 필요하다면 <code>GetObject</code>만 허용합니다.</p>
<h3 id="6-2-cors-도메인-제한">6-2. CORS 도메인 제한</h3>
<p><code>AllowedOrigins</code>에 &quot;*&quot;를 사용하지 말고, 실제 사용할 도메인만 명시합니다.</p>
<h3 id="6-3-presigned-url-유효시간-제한">6-3. Presigned URL 유효시간 제한</h3>
<p>백엔드 코드에서 presigned URL 생성 시 적절한 만료시간을 설정합니다:</p>
<pre><code class="language-java">@Service
public class S3Service {

    @Value(&quot;${aws.s3.bucket}&quot;)
    private String bucketName;

    private final AmazonS3 amazonS3;

    // 업로드용 Presigned URL 생성 (15분 유효)
    public String generatePresignedUploadUrl(String fileName) {
        Date expiration = new Date();
        long expTimeMillis = expiration.getTime();
        expTimeMillis += 1000 * 60 * 15; // 15분
        expiration.setTime(expTimeMillis);

        GeneratePresignedUrlRequest generatePresignedUrlRequest = 
            new GeneratePresignedUrlRequest(bucketName, fileName)
                .withMethod(HttpMethod.PUT)
                .withExpiration(expiration);

        return amazonS3.generatePresignedUrl(generatePresignedUrlRequest).toString();
    }

    // 다운로드용 Presigned URL 생성 (1시간 유효)
    public String generatePresignedDownloadUrl(String fileName) {
        Date expiration = new Date();
        long expTimeMillis = expiration.getTime();
        expTimeMillis += 1000 * 60 * 60; // 1시간
        expiration.setTime(expTimeMillis);

        GeneratePresignedUrlRequest generatePresignedUrlRequest = 
            new GeneratePresignedUrlRequest(bucketName, fileName)
                .withMethod(HttpMethod.GET)
                .withExpiration(expiration);

        return amazonS3.generatePresignedUrl(generatePresignedUrlRequest).toString();
    }
}</code></pre>
<h2 id="마무리">마무리</h2>
<p>Presigned URL을 위한 S3 설정은 보안과 기능성의 균형이 중요합니다. 퍼블릭 액세스는 완전히 차단하면서도, 서명된 요청을 통한 제한적 접근은 허용하는 것이 핵심입니다.</p>
<p>처음 S3 bucket을 생성하며 자료를 찾아볼 때, 글로만 설명되어있는 경우가 많아 초보자 입장에서 많은 고민이 있었습니다. 특히, 각 항목들에 대한 자세한 설명이 없어 '이대로 정말 따라해도 되는건가'에 대해 찾아보는데 많은 시간을 투자했기에 최대한 자세히 이미지를 통해 각 항목에 대해서도 설명하고자 노력했습니다. 
이 가이드를 따라 설정하면 안전하고 효율적인 파일 업로드/다운로드 시스템을 구축할 수 있습니다. 특히 CORS 설정을 놓치기 쉬우니 꼭 확인하시기 바랍니다!</p>
<h3 id="주요-포인트-요약">주요 포인트 요약</h3>
<ul>
<li>✅ <strong>S3 버킷</strong>: 퍼블릭 액세스 완전 차단</li>
<li>✅ <strong>버킷 정책</strong>: 서명된 요청에 대한 권한 부여  </li>
<li>✅ <strong>IAM 설정</strong>: 백엔드가 presigned URL 생성할 수 있는 권한</li>
<li>✅ <strong>CORS 설정</strong>: 브라우저 호환성 확보</li>
<li>✅ <strong>보안 원칙</strong>: 최소 권한 + 도메인별 접근 제한</li>
<li>✅ <strong>환경변수</strong>: 크리덴셜 보안 관리</li>
</ul>
<p>이 가이드를 따라 설정하면 안전하고 효율적인 파일 업로드/다운로드 시스템을 구축할 수 있습니다!</p>