<h1 id="개요">개요</h1>
<p>&nbsp;&nbsp;이번에 프로젝트를 진행하며 EC2 배포를 진행했다. 검색과 시행착오를 통해 배포를 완료했기에 추후 잊지 않기 위해 기록을 남긴다.</p>
<h2 id="ec2-생성하기">EC2 생성하기</h2>
<h4 id="1-리전-선택">&nbsp;&nbsp;1. 리전 선택</h4>
<p>&nbsp;&nbsp;&nbsp;&nbsp;먼저, EC2 인스턴스를 생성하기 전 원하는 지역을 먼저 선택한다. 
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/2912c279-53a8-46f1-80c8-468cd5c8f9bc/image.png" /></p>
<h4 id="2-ec2---인스턴스-시작">&nbsp;&nbsp;2. EC2 -&gt; 인스턴스 시작</h4>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/ae3e0ba2-4e8e-4417-ace2-dc761d7eac66/image.png" /></p>
<h4 id="3-서버-선택">&nbsp;&nbsp;3. 서버 선택</h4>
<p>&nbsp;&nbsp;&nbsp;&nbsp;본 프로젝트에서는 <strong>ubuntu 서버</strong>를 선택했다. 
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/e66fcf76-87c6-4032-9673-d8a10c31a5db/image.png" /></p>
<h4 id="4-키페어-생성">&nbsp;&nbsp;4. 키페어 생성</h4>
<p>&nbsp;&nbsp;&nbsp;&nbsp;ssh 접속을 위해 pem 키 생성을 해주었다. 
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/b4b3d01b-9df8-47ed-8a03-8601eecae2e6/image.png" />
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/468153e7-cedb-4df2-9ae4-e669f76e6588/image.png" /></p>
<h4 id="5-네트워크-설정">&nbsp;&nbsp;5. 네트워크 설정</h4>
<p>&nbsp;&nbsp;&nbsp;&nbsp;기존 보안그룹이 없다면 보안그룹을 생성하고 프로젝트 배포를 위한 것이기 때문에 원격 서버 접속을 위해 SSH 트래픽 허용, HTTPS, HTTP 허용을 체크해준다.
SSH 트래픽 허용의 경우 pem키가 없으면 원격 접속이 어차피 불가하기도 하고 집과 독서실, 카페 등등 이곳 저곳에서 작업을 하기 때문에 모든 IP를 열어놓았지만, 해커들의 접근이 걱정된다면 집, 혹은 작업 환경의 IP만 열어놓으면 된다. 
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/80c7a141-bd74-4544-ba3a-e67c9c094d93/image.png" /></p>
<h4 id="6-인스턴스-실행-확인">&nbsp;&nbsp;6. 인스턴스 실행 확인</h4>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/39531ad1-dce0-466b-8dbc-20df2550fdb9/image.png" /></p>
<h4 id="7-pem키로-원격-서버-접속하기">&nbsp;&nbsp;7. pem키로 원격 서버 접속하기</h4>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/a58eb076-d64f-49a9-bab6-3ca4db093e2c/image.png" />
&nbsp;&nbsp;&nbsp;&nbsp;pem키 파일이 위치한 곳에서 _<strong>ssh -i {pem키 이름}.pem {서버}@{EC2 퍼블릭 IPv4 주소}</strong>_를 입력하여 원격 서버에 접속 가능하다.
해당 로컬에서 처음 접속할 경우 이 곳에서 원격 접속하겠냐고 묻는데 yes를 입력하면 된다.
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/3de89646-d67e-4756-9f8b-570317e3d411/image.png" /></p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;아래와 같이 화면이 뜬다면 제대로 서버에 접속한 것이다.
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/0bf09dba-26d0-4daf-a786-ccaf484a7b03/image.png" /></p>
<h3 id="참고">참고</h3>
<p>&nbsp;&nbsp;배포 방식으로 내가 알고 있는 건 아래와 같이 세 가지가 있는데
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 직접 jar 파일을 EC2에 배포
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. Docker 컨테이너 사용
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. CI/CD 파이프라인 구축 (예: GitHub Actions, Jenkins)</p>
<p>&nbsp;&nbsp;처음에는 직접 jar 파일을 EC2에 배포했었고 현재는 Github Actions를 통해 CI/CD 파이프라인을 구축하여 놓았다. 다만, 이는 다른 글을 통해 따로 다루는 편이 낫다고 생각하여 여기서는 따로 설명하지 않았다.</p>
<h2 id="참고문헌">참고문헌</h2>
<p><a href="https://velog.io/@chwogus/AWS-SpringBoot-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-EC2-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-alj46g2b">SpringBoot 프로젝트 EC2 배포하기</a>
<a href="https://cobi-98.tistory.com/73">SpringBoot 프로젝트 EC2 배포하기</a></p>