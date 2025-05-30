<h2 id="개요">개요</h2>
<p>&nbsp;&nbsp; 이전 글에서는 CI/CD의 개념과 GitHub Actions의 특징에 대해 알아보았다. 이번 글에서는 실제로 GitHub Actions를 사용하여 SpringBoot 프로젝트를 AWS EC2에 자동으로 배포하는 구체적인 방법과 코드에 대해 알아보도록 하겠다.</p>
<h3 id="실제-워크플로우-파일-구성">실제 워크플로우 파일 구성</h3>
<p>&nbsp;&nbsp;이론적인 부분은 이전 글에서 정리했으니, 이제 실제 프로젝트에 적용한 워크플로우 파일을 살펴보자. 아래는 이번 프로젝트에 사용한 Github Actions 워크플로우 파일 구조다.</p>
<pre><code class="language-yaml">name: {PROJECT_NAME}-deploy.yml
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Create database-stage.properties
        run: |
          echo '${{ secrets.DATABASE_PROPERTIES }}' &gt; ./src/main/resources/database.properties

      # ... 기타 설정 파일 생성 코드 ...

      - name: Execute deploy script
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          script: |
            export SPRING_ENV=prod
            cd {PROJECT_NAME}/
            ./deploy.sh</code></pre>
<h3 id="워크플로우-구조-분석">워크플로우 구조 분석</h3>
<h4 id="1-워크플로우-트리거-설정">&nbsp;&nbsp;1. 워크플로우 트리거 설정</h4>
<pre><code class="language-yaml">name: {PROJECT_NAME}-deploy.yml
on:
  push:
    branches:
      - main</code></pre>
<ul>
<li><p><code>name</code>
워크플로우의 이름을 정의한다.</p>
</li>
<li><p><code>on</code>
워크플로우가 실행될 조건을 지정한다.</p>
</li>
<li><p><code>push</code>
main 브랜치에 푸시 이벤트가 발생할 때마다 워크플로우가 실행된다.
(main 브랜치가 아니라 다른 브랜치에 적용하고 싶다면 여기를 변경해주면 된다.)</p>
</li>
</ul>
<h4 id="2-보안-설정-파일-생성">&nbsp;&nbsp;2. 보안 설정 파일 생성</h4>
<pre><code class="language-yaml">steps:
  - uses: actions/checkout@v3

  - name: Create database-stage.properties
    run: |
      echo '${{ secrets.DATABASE_PROPERTIES }}' &gt; ./src/main/resources/database.properties

  # 기타 설정 파일 생성...</code></pre>
<p>&nbsp;&nbsp;이 부분에서는 이전 포스팅에서 언급했던 EC2에서 직접 설정 파일을 만들어주던 문제를 해결하고 있다.</p>
<p>&nbsp;&nbsp;GitHub의 Secrets 기능을 활용해 민감한 설정 정보를 저장한다.
&nbsp;&nbsp; 워크플로우 실행 시 이러한 시크릿 값들을 파일로 생성한다.
&nbsp;&nbsp; 데이터베이스, JWT, 외부 API, S3, Redis 연결 정보 등 모든 설정 파일을 자동으로 생성한다.</p>
<h4 id="3-빌드-환경-설정-및-캐싱">&nbsp;&nbsp;3. 빌드 환경 설정 및 캐싱</h4>
<pre><code class="language-yaml">- name: Set up JDK 17
  uses: actions/setup-java@v3
  with:
    java-version: '17'
    distribution: 'liberica'

- name: Gradle Caching
  uses: actions/cache@v3
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
    restore-keys: |
      ${{ runner.os }}-gradle-</code></pre>
<p>&nbsp;&nbsp;JDK 17을 설치하여 빌드 환경을 구성한다.
&nbsp;&nbsp; Gradle 의존성을 캐싱해 빌드 속도를 개선한다.</p>
<h4 id="4-애플리케이션-빌드">&nbsp;&nbsp;4. 애플리케이션 빌드</h4>
<pre><code class="language-yaml">- name: Grant execute permission for gradlew
  run: chmod +x gradlew

- name: Build with Gradle
  run: ./gradlew clean build -x test</code></pre>
<p>&nbsp;&nbsp;gradlew 파일에 실행 권한을 부여한다.
&nbsp;&nbsp; Gradle을 사용해 애플리케이션을 빌드한다. 테스트는 제외(<code>-x test</code>)했다.</p>
<h4 id="5-ec2-서버에-배포">&nbsp;&nbsp;5. EC2 서버에 배포</h4>
<pre><code class="language-yaml">- name: Deploy to EC2
  uses: appleboy/scp-action@master
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ${{ secrets.EC2_USERNAME }}
    key: ${{ secrets.EC2_PRIVATE_KEY }}
    source: &quot;build/libs/*.jar&quot;
    target: &quot;/home/ubuntu/{PROJECT_NAME}&quot;
    strip_components: 2</code></pre>
<p><code>appleboy/scp-action</code>을 사용해 빌드된 JAR 파일을 EC2 서버로 전송한다.
EC2 접속 정보는 GitHub Secrets에서 가져온다.
<code>strip_components: 2</code>는 전송 시 디렉토리 구조에서 상위 2개 경로를 제거한다.</p>
<ul>
<li>여기서 원하는 경로에 확실히 해당 파일이 생기는 지 폴더 구조를 정확히 파악해야 오류가 생기지 않는다.</li>
</ul>
<h4 id="6-배포-스크립트-실행">&nbsp;&nbsp;6. 배포 스크립트 실행</h4>
<pre><code class="language-yaml">- name: Execute deploy script
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ${{ secrets.EC2_USERNAME }}
    key: ${{ secrets.EC2_PRIVATE_KEY }}
    script: |
      export SPRING_ENV=prod
      cd {PROJECT_NAME}/
      ./deploy.sh</code></pre>
<p>&nbsp;&nbsp;<code>appleboy/ssh-action</code>을 사용해 EC2 서버에 SSH 접속한다.
&nbsp;&nbsp; 환경 변수를 설정하고 이전에 작성한 <code>deploy.sh</code> 스크립트를 실행한다.</p>
<h3 id="배포-스크립트">배포 스크립트</h3>
<p>&nbsp;&nbsp;GitHub Actions 워크플로우의 마지막 단계에서는 EC2 서버에 있는 deploy.sh 스크립트를 실행한다. 이 스크립트는 이전 애플리케이션을 중지하고 새 버전을 시작하는 역할을 한다. 아래는 이번 프로젝트에서 실제 사용한 배포 스크립트 구조이다.</p>
<pre><code class="language-bash">#!/bin/bash
# 환경 변수 설정
PROJECT_DIR=&quot;/home/ubuntu/{PROJECT_NAME}&quot;
APP_NAME=&quot;{APP_NAME}&quot;
JAR_FILE=&quot;{PROJECT_NAME}-{VERSION}.jar&quot;

# 로그 설정
LOG_TIME=$(date &quot;+%Y%m%d-%H%M%S&quot;)
LOG_FILE=&quot;$PROJECT_DIR/logs/spring-$LOG_TIME.log&quot;
mkdir -p $PROJECT_DIR/logs
echo &quot;[$LOG_TIME] Starting deploy script...&quot; &gt;&gt; $LOG_FILE
echo &quot;Stopping existing process...&quot; &gt;&gt; $LOG_FILE

# 기존 프로세스 종료
ps -ef | grep $APP_NAME | grep -v grep | awk '{print $2}' | xargs -r kill -9
sleep 3

# 새 애플리케이션 시작
echo &quot;Starting new application...&quot; &gt;&gt; $LOG_FILE
nohup java -jar -Dspring.profiles.active=prod $PROJECT_DIR/$JAR_FILE &gt; $LOG_FILE 2&gt;&amp;1 &amp;
sleep 10

# 실행 확인
PID=$(pgrep -f $JAR_FILE)
if [ -n &quot;$PID&quot; ]; then
    echo &quot;Application started successfully with PID: $PID&quot; &gt;&gt; $LOG_FILE
    echo &quot;Logs available at: $LOG_FILE&quot;
else
    echo &quot;Failed to start application. Check logs at: $LOG_FILE&quot; &gt;&gt; $LOG_FILE
    exit 1
fi</code></pre>
<h4 id="1-환경-변수-설정">&nbsp;&nbsp;1. 환경 변수 설정</h4>
<p>&nbsp;&nbsp; 프로젝트 디렉토리, 애플리케이션 이름, JAR 파일명을 변수로 정의하여 관리한다.
&nbsp;&nbsp; 이렇게 하면 다른 프로젝트에 스크립트를 재사용할 때 상단의 변수만 변경하면 된다.</p>
<h4 id="2-로그-관리">&nbsp;&nbsp;2. 로그 관리</h4>
<p>&nbsp;&nbsp; 타임스탬프를 이용해 각 배포마다 고유한 로그 파일을 생성한다.
&nbsp;&nbsp; 로그 디렉토리가 없으면 자동으로 생성한다.</p>
<h4 id="3-이전-프로세스-종료">&nbsp;&nbsp;3. 이전 프로세스 종료</h4>
<p>&nbsp;&nbsp; 현재 실행 중인 애플리케이션을 찾아서 종료한다.
&nbsp;&nbsp; 설정된 애플리케이션 이름으로 프로세스를 검색하고 해당 PID를 종료한다.</p>
<h4 id="4-새-애플리케이션-시작">&nbsp;&nbsp;4. 새 애플리케이션 시작</h4>
<p>&nbsp;&nbsp; nohup을 사용해 SSH 세션이 종료되어도 애플리케이션이 계속 실행되도록 한다.
&nbsp;&nbsp; spring.profiles.active=prod로 프로덕션 환경 설정을 적용한다.</p>
<h4 id="5-실행-확인">&nbsp;&nbsp;5. 실행 확인</h4>
<p>&nbsp;&nbsp; 10초 대기 후 프로세스가 정상적으로 시작되었는지 확인한다.
&nbsp;&nbsp; 시작 실패 시 오류 메시지를 로그에 기록하고 스크립트를 종료한다.</p>
<p>&nbsp;&nbsp; 이 스크립트는 이전 포스팅에서 소개한 기본 배포 스크립트에 로깅 기능과 오류 처리 기능을 추가하여 더욱 안정적인 배포가 가능하도록 개선한 버전이다.</p>
<h2 id="후기">후기</h2>
<p>&nbsp;&nbsp; GitHub Actions를 통한 CI/CD 파이프라인 구축 후 다음과 같은 이점을 경험할 수 있었다</p>
<h4 id="1-개발-생산성-향상">&nbsp;&nbsp;1. 개발 생산성 향상</h4>
<p>&nbsp;&nbsp; 코드 푸시 한 번으로 전체 배포 과정이 자동화되어 배포에 드는 시간과 노력이 크게 줄었다.</p>
<h4 id="2-설정-파일-관리-간소화">&nbsp;&nbsp;2. 설정 파일 관리 간소화</h4>
<p>&nbsp;&nbsp; 이전에는 EC2에 직접 설정 파일을 만들어야 했지만, 이제는 GitHub Secrets를 통해 안전하게 관리된다.</p>
<h4 id="3-배포-안정성-향상">&nbsp;&nbsp;3. 배포 안정성 향상</h4>
<p>&nbsp;&nbsp; 항상 동일한 과정으로 빌드 및 배포가 이루어져 인적 오류가 감소했다.</p>
<h4 id="4-투명한-배포-이력">&nbsp;&nbsp;4. 투명한 배포 이력</h4>
<p>&nbsp;&nbsp; GitHub에서 모든 배포 과정과 결과를 확인할 수 있어 문제 발생 시 빠른 대응이 가능해졌다.</p>
<p>&nbsp;&nbsp; 비록 CI(테스트 자동화)는 아직 적용하지 않았지만, CD(배포 자동화)만으로도 개발 workflow에 큰 개선이 이루어졌다. 앞으로 테스트 자동화도 점진적으로 도입하여 완전한 CI/CD 파이프라인을 구축할 계획이다.
&nbsp;&nbsp; 처음 배포 자동화를 적용할 때는 설정이 다소 복잡하게 느껴질 수 있지만, 한 번 구축해놓으면 그 효율성은 분명히 체감할 수 있다. 이 글을 보는 분들의 프로젝트에도 이러한 자동화 파이프라인을 적용해보시길 추천드린다.</p>
<h2 id="참고문헌">참고문헌</h2>
<p><a href="https://velog.io/@bagt/Github-Actions%EB%A5%BC-%ED%86%B5%ED%95%9C-%EB%B0%B0%ED%8F%AC">Github-Actions를 통한 배포</a></p>