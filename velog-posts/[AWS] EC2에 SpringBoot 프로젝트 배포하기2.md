<h2 id="개요">개요</h2>
<p>&nbsp;&nbsp;프로젝트를 진행하며 현재는 Github Actions를 이용하여 CI/CD pipeline을 구축해놓았으나 초기에는 직접 EC2 서버 내부에서 git repository에서 clone을 받는 방법으로 AWS EC2에 배포를 진행했었다. 처음 서버에 배포할 때 꽤나 헤맸었기에 한 번 정리해 놓고자 한다.
&nbsp;&nbsp; 기본적으로, SpringBoot 프로젝트와 AWS EC2 인스턴스 세팅은 다 되어있다고 가정하고 설명을 이어나가도록 하겠다.</p>
<h2 id="배포-과정">배포 과정</h2>
<h3 id="1-ec2-서버-접속">1. EC2 서버 접속</h3>
<p>&nbsp;&nbsp;먼저 EC2 서버에 접속한다. 앞서 다른 글에서 설명한 대로 pem키를 이용해서 ssh 접속으로 서버 접속을 실시하였다. </p>
<h3 id="2-jdk-설치">2. JDK 설치</h3>
<p>&nbsp;&nbsp;아마 추후 따로 설명할 글이 있을 지 모르겠지만 이번에 기본 프로젝트 세팅을 JAVA 21에 맞춰서 진행했었다. 하지만, ubuntu 서버 내부에서 jdk 21을 사용한 관련 자료들이 적은 점, Java 21이 Java 17에 비해 우리 서비스에서 특장점을 가지지 않는 점을 이유로 17로 변경해서 진행하였다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/a494be94-ce64-4260-a9c1-5571fc3d3953/image.png" /></p>
<h3 id="3-프로젝트-디렉토리-생성">3. 프로젝트 디렉토리 생성</h3>
<p>&nbsp;&nbsp; 프로젝트 이름으로 디렉토리를 생성한다.(다른 이름으로 해도 무관)
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/282796cc-7814-4af7-a8fc-93afa5888b52/image.png" /></p>
<h3 id="4-배포-파일-준비-방법">4. 배포 파일 준비 방법</h3>
<h4 id="4-1-로컬에서-jar파일을-전송할-경우">&nbsp;&nbsp; 4-1. 로컬에서 jar파일을 전송할 경우</h4>
<p>&nbsp;&nbsp; 프로젝트 루트 디렉토리에서 build를 통해 jar파일을 만들고 scp 명령어를 통해 서버에 해당 파일을 전송하는 방식이다. 
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/921ea4e7-e019-4e4f-9b9d-8ec7ab93b2e6/image.png" /></p>
<pre><code class="language-ubuntu">scp -i {pem키 이름}.pem build/libs/[프로젝트이름].jar {ubuntu}@{IpV4 퍼블릭 주소}: ~/{원하는 프로젝트 디렉토리 위치}</code></pre>
<h4 id="4-2-원격-ec2에서-바로-git-clone을-통해-빌드할-경우">&nbsp;&nbsp; 4-2. 원격 ec2에서 바로 git clone을 통해 빌드할 경우</h4>
<p>&nbsp;&nbsp; 이 방법을 사용한 이유는, 로컬에서 전송하는 과정이 필요 없고, 기능 추가 등과 같은 코드 변경시에 어차피 git에 push를 하는데 build를 따로 하는 작업이 불필요하다고 느껴졌기 때문이다. 더 체계적인 배포가 가능하기도 하고 가장 큰 이유는 아무래도 '불편해서'였다.
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/37f2b812-3618-44d4-a96a-81d91402d4d6/image.png" /></p>
<p>&nbsp;&nbsp; 해당 위치는 /home/ubuntu/{디렉토리}가 된다. ~없이 그냥 생성할 경우 루트 디렉토리 아래에 생성되는데 이는 관리나 보안 측면에서 좋지 않기에 해당 경로를 특정하여 생성해주었다.
&nbsp;&nbsp; 이렇게 하면 테스트 실패와 관련해서 에러가 발생했는데(ConfigDataResourceNotFoundException) 주로, 배포 시 특히 RDS 연동 등의 문제로 인해 테스트는 제외하고 빌드하는 <code>./gradlew build -x test</code>를 많이 사용한다고 하여 해당 옵션으로 빌드를 진행했다. </p>
<h4 id="4-3-gitignore된-설정파일-생성">&nbsp;&nbsp; 4-3. gitignore된 설정파일 생성</h4>
<p>&nbsp;&nbsp; git clone으로 가져온 프로젝트에는 보안상의 이유로 .gitignore에 등록된 중요 설정 파일들이 포함되어 있지 않습니다. 따라서 이러한 파일들은 서버에서 직접 생성해주어야 합니다.
ex. <code>vim src/main/resources/application-prod.yml</code> 등등</p>
<h4 id="4-4-빌드된-jar-파일-확인">&nbsp;&nbsp; 4-4. 빌드된 jar 파일 확인</h4>
<p>&nbsp;&nbsp; <code>ls build/libs</code> 명령어를 통해 빌드된 jar 파일을 확인 할 수 있다.</p>
<h4 id="5-배포-스크립트-생성">&nbsp;&nbsp; 5. 배포 스크립트 생성</h4>
<p>&nbsp;&nbsp; 배포를 자동화하기 위한 스크립트를 생성한다.
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/ebfe84a6-a21c-4a03-80f9-aec7ffe0907a/image.png" /></p>
<p>&nbsp;&nbsp; nano도 사용 가능하지만 vim이 더 익숙해서 vim으로 진행하였다. 여기서 i키를 누르면 입력보드로 전화되고 아래와 같이 내용을 입력해준다.
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/8ddcfcaf-ca43-480c-a5cd-bb9439b2ef02/image.png" /></p>
<p>&nbsp;&nbsp; 각 내용의 세부 설명은 다음과 같다.
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/1d4b9d62-3bb2-4740-815c-4513a979c86d/image.png" />
&nbsp;&nbsp; 참고로, nohup 명령어가 빠지면 해당 콘솔창을 열어놓는 동안에만 애플리케이션이 실행되는 것으로 필요에 따라 선택하면 된다.
&nbsp;&nbsp; 중간의 profiles.active=prod의 경우 application-prod.yml(프로덕션 환경에 맞는 설정)으로 적용되도록 하였다.
&nbsp;&nbsp; 또한, nohup.out 파일에는 로그가 저장된다.</p>
<h2 id="배포-스크립트-실행-및-확인">배포 스크립트 실행 및 확인</h2>
<p>&nbsp;&nbsp; 1. 스크립트 파일에 실행 권한을 부여하고 실행
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/2e051aaa-11fc-45fa-9dde-af0b62b28de6/image.png" />
&nbsp;&nbsp; 2. 애플리케이션 실행 확인
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/4757020a-8303-4a79-97dd-26dc84940995/image.png" />
&nbsp;&nbsp; 3. 로그 확인
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/aec39bfd-55fc-4dde-bf73-7511de6be663/image.png" /></p>
<h3 id="참고">참고</h3>
<p>&nbsp;&nbsp; 이처럼 EC2에 직접 배포하는 방법은 어떻게 본다면 간단하지만, 매번 서버에 접속하여 작업해야 하는 번거로움이 있다. 또한, 보안 혹은 관리를 위해 gitignore로 설정 파일들을 등록해놓았는데 갱신 혹은 추가될 때마다 서버에 접속해서 이를 추가해주어야 한다는 것도 실수의 여지가 있고 다소 번거로웠다. 따라서, 이후 Github Actions를 통한 CI/CD 파이프라인 구축으로 배포 과정을 자동화하여 개발 생산성을 크게 향상시켰다.</p>
<h2 id="참고문헌">참고문헌</h2>
<p><a href="https://velog.io/@wndudrla1011/chapter-8">EC2 서버에 프로젝트 배포하기</a>
<a href="https://ironmask43.tistory.com/24">AWS EC2 서버에 프로젝트를 배포해보자</a></p>