<h1 id="🚀-ec2-프리티어에서-docker로-spring-boot-배포하며-마주한-문제들">🚀 EC2 프리티어에서 Docker로 Spring Boot 배포하며 마주한 문제들</h1>
<p>이전 글에서는 GitHub Actions를 활용하여 <code>.jar</code> 파일을 EC2에 배포하고 실행하는 자동화 파이프라인을 구축하는 과정을 소개했습니다.</p>
<p>이후 운영의 유연성과 확장성을 높이기 위해 <strong>Docker 기반 배포 방식으로 구조를 개선</strong>하고자 했습니다.<br />하지만 Docker 기반 배포로 전환하는 과정은 순탄하지 않았습니다.</p>
<p>Dockerfile과 <code>.jar</code> 파일을 EC2로 전송하여 EC2 내부에서 이미지를 빌드하려 했을 때는 <strong>디스크 용량 부족</strong> 문제가 있었고,<br />컨테이너에서 Redis에 연결하려 하자 <strong>네트워크 설정 문제</strong>로 인해 접속이 실패하기도 했습니다.</p>
<p>이번 글에서는 Docker 기반 배포 환경으로 전환하는 과정에서 겪었던 <strong>실전 문제들</strong>과  
각각의 <strong>원인 분석 및 해결 방법</strong>을 공유하고자 합니다.</p>
<hr />
<h2 id="⚙️-초기-docker-배포-시도와-문제의-시작">⚙️ 초기 Docker 배포 시도와 문제의 시작</h2>
<p>Docker를 적용한 초기에는 다음과 같은 구조로 배포를 진행했습니다.</p>
<ul>
<li><strong>GitHub Actions</strong>에서 <code>.jar</code> 파일 빌드  </li>
<li>EC2 프리티어 서버에 <code>.jar</code>, <code>Dockerfile</code> 전송  </li>
<li>EC2 내부에서 Docker 이미지 빌드 및 실행  </li>
</ul>
<p>하지만 이 과정에서 디스크 용량 부족 문제가 발생해 다음과 같은 오류가 나타났습니다:</p>
<pre><code>failed to copy files: no space left on device</code></pre><p>프리티어 EC2의 8GB 디스크는 <code>openjdk</code> 베이스 이미지와 <code>.jar</code> 파일, Docker 레이어 등을 감당하기엔 턱없이 부족했습니다.</p>
<hr />
<h2 id="💥-문제-1-no-space-left-on-device-디스크-용량-부족">💥 문제 1: no space left on device (디스크 용량 부족)</h2>
<p>디스크 부족 문제는 다음과 같은 요소들이 복합적으로 작용한 결과였습니다:</p>
<h3 id="📌-근본-원인">📌 근본 원인</h3>
<ul>
<li><code>openjdk:17-jdk-slim</code> 베이스 이미지도 약 300MB 이상  </li>
<li><code>.jar</code> 파일을 Docker 이미지에 COPY하면서 <strong>중복 저장</strong>  </li>
<li>Docker 이미지 빌드시 <strong>중간 레이어와 캐시가 EC2에 그대로 남음</strong>  </li>
</ul>
<blockquote>
<p>결과적으로 EC2 디스크는 빠르게 가득 차고, Docker는 더 이상 레이어를 쌓을 수 없어 빌드가 실패했습니다.</p>
</blockquote>
<hr />
<h2 id="✅-해결-방법-1-이미지-빌드-위치-변경-github-actions-→-dockerhub-→-ec2-pull">✅ 해결 방법 1: 이미지 빌드 위치 변경 (GitHub Actions → DockerHub → EC2 pull)</h2>
<p>이 문제를 해결하기 위해 EC2에서 Docker 이미지를 직접 빌드하는 방식을 폐기하고,<br /><strong>GitHub Actions에서 Docker 이미지를 빌드하여 DockerHub에 push</strong>,  
<strong>EC2에서는 이미지를 pull만 해서 실행하는 구조</strong>로 전환했습니다.</p>
<h3 id="🔧-변경-사항">🔧 변경 사항</h3>
<ol>
<li><strong>GitHub Actions에서 <code>.jar</code> 빌드 + Docker 이미지 생성</strong>  </li>
<li><strong>DockerHub에 이미지 푸시</strong>  </li>
<li><strong>EC2에서는 <code>docker pull</code>로 이미지만 받아 실행</strong></li>
</ol>
<h3 id="🛠️-관련-github-actions-설정">🛠️ 관련 GitHub Actions 설정</h3>
<pre><code class="language-yaml">- name: Build and push Docker image
  run: |
    docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/solve-nyang:latest .
    docker push ${{ secrets.DOCKERHUB_USERNAME }}/solve-nyang:latest

- name: Deploy to EC2
  uses: appleboy/ssh-action@master
  with:
    script: |
      docker stop solve-nyang || true
      docker rm solve-nyang || true
      docker pull ${{ secrets.DOCKERHUB_USERNAME }}/solve-nyang:latest
      docker run -d \
        --name solve-nyang \
        -p 8080:8080 \
        --add-host host.docker.internal:host-gateway \
        -v /home/ubuntu/solve-nyang/logs:/logs \
        --restart unless-stopped \
        ${{ secrets.DOCKERHUB_USERNAME }}/solve-nyang:latest</code></pre>
<p>이 구조로 변경하고 나니 <strong>EC2 디스크 용량 부족 문제는 완전히 해결</strong>되었습니다.</p>
<hr />
<h2 id="🧱-문제-2-redis-연결-실패">🧱 문제 2: Redis 연결 실패</h2>
<p>Docker로 전환 후, Redis에 접근하는 API를 호출하자 다음과 같은 오류가 발생했습니다:</p>
<pre><code class="language-json">{
  &quot;error&quot;: &quot;Authentication Failed&quot;,
  &quot;message&quot;: &quot;Unable to connect to Redis&quot;,
  &quot;status&quot;: 401
}</code></pre>
<h3 id="🕵️-원인-분석">🕵️ 원인 분석</h3>
<p>Docker 컨테이너는 <strong>자체 네트워크 공간에서 실행</strong>되기 때문에,<br />기존의 <code>localhost:6379</code> 설정으로는 <strong>호스트 Redis에 접근할 수 없습니다.</strong></p>
<pre><code># 기존 구조 (직접 실행)
Spring Boot (호스트) → Redis (127.0.0.1:6379) ✅

# Docker 컨테이너 구조
Spring Boot (컨테이너) → Redis (127.0.0.1:6379) ❌</code></pre><hr />
<h2 id="✅-해결-방법-2-redis-접근-경로와-docker-네트워크-설정-수정">✅ 해결 방법 2: Redis 접근 경로와 Docker 네트워크 설정 수정</h2>
<h3 id="1-redisproperties-설정-수정">1. <code>redis.properties</code> 설정 수정</h3>
<pre><code class="language-properties">REDIS.HOST=host.docker.internal</code></pre>
<h3 id="2-docker-run-시---add-host-옵션-추가">2. <code>docker run</code> 시 <code>--add-host</code> 옵션 추가</h3>
<pre><code class="language-bash">--add-host host.docker.internal:host-gateway</code></pre>
<ul>
<li>이 옵션을 통해 <strong>컨테이너 내부에서 호스트를 <code>host.docker.internal</code>로 인식</strong>할 수 있습니다.  </li>
<li>Docker Desktop(macOS/Windows)은 기본 지원하지만, <strong>리눅스에서는 수동 설정 필수</strong>입니다.</li>
</ul>
<h3 id="3-redis-서버-설정-변경">3. Redis 서버 설정 변경</h3>
<pre><code class="language-conf"># /etc/redis/redis.conf

bind 127.0.0.1 172.17.0.1 -::1
protected-mode no</code></pre>
<ul>
<li><code>bind</code> 설정으로 Docker 브리지 네트워크 접근 허용  </li>
<li><code>protected-mode no</code> 설정으로 외부 접근 허용 (단, 방화벽 등 보안 조치 병행 필요)</li>
</ul>
<hr />
<h2 id="📦-최종-구조-요약">📦 최종 구조 요약</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>이전 방식</th>
<th>현재 방식</th>
</tr>
</thead>
<tbody><tr>
<td>Docker 이미지 빌드</td>
<td>EC2에서 직접 빌드</td>
<td>GitHub Actions에서 빌드 후 DockerHub 푸시</td>
</tr>
<tr>
<td>이미지 전송 방식</td>
<td><code>.jar</code> + Dockerfile 전송</td>
<td>Docker 이미지 pull (EC2에서 실행)</td>
</tr>
<tr>
<td>Redis 연결</td>
<td>localhost 접근 (실패)</td>
<td><code>host.docker.internal</code> 경유로 연결 성공</td>
</tr>
<tr>
<td>디스크 사용량</td>
<td>중복 저장으로 과도</td>
<td>최소화, 빌드/캐시 이전 처리로 최적화</td>
</tr>
</tbody></table>
<hr />
<h2 id="✍️-마무리하며">✍️ 마무리하며</h2>
<p>Docker는 굉장히 유연하고 강력한 도구이지만, <strong>&quot;감싸고 실행&quot;만으로는 충분하지 않습니다.</strong><br />네트워크, 스토리지, 환경변수, 시크릿, 레이어 구조, OS별 설정 등 고려해야 할 요소들이 많고,<br />특히 리소스가 제한된 <strong>프리티어 EC2 환경에서는 더더욱 철저한 최적화</strong>가 필요합니다.</p>
<p>이번 경험을 통해, 단순한 배포 자동화 이상의 <strong>인프라 전반에 대한 이해</strong>가 중요하다는 것을 다시금 느꼈습니다.<br />앞으로도 시행착오를 두려워하지 않고, 한 단계씩 성장해 나가겠습니다.</p>