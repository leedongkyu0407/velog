<h2 id="개요">개요</h2>
<p>&emsp;프로젝트를 진행하며 백엔드 서버 배포를 AWS EC2 프리티어를 통해 진행했다. MySQL DB는 별도의 RDS 서버에 구축했으며, 보안을 위해 EC2에서 들어오는 요청만 받을 수 있도록 보안그룹의 인바운드 규칙을 제한해 놓았다.</p>
<p>개발 중에 매번 EC2에 SSH 접속 후 RDS에 접속하는 것이 번거로워 로컬에서 직접 접속할 수 있는 방법을 찾았고, SSH 터널링 방식을 사용하게 되어 이를 기록하고자 한다.</p>
<h2 id="터널링">터널링</h2>
<h3 id="정의">정의</h3>
<p>&emsp;<a href="http://www.ktword.co.kr/test/view/view.php?m_temp1=1708">정보통신기술용어해설</a>에 따르면, 터널링은 두 노드 또는 두 네트워크 간에 가상의 링크(VPN 등)를 형성하는 기법이다.</p>
<h3 id="특징">특징</h3>
<ul>
<li>하나의 프로토콜이 다른 프로토콜을 감싸는 캡슐화 기능을 통해 운반<ul>
<li>대부분, 보안 채널의 역할을 하므로, 암호화 기법 적용이 일반적임</li>
</ul>
</li>
</ul>
<p>&emsp;이러한 터널링을 통해 로컬 환경에서도 마치 EC2 서버에서 직접 접속하는 것처럼 RDS에 안전하게 접근할 수 있다.</p>
<h2 id="터널링-방법">터널링 방법</h2>
<h3 id="1-터미널에서-ssh-터널링-설정">1. 터미널에서 SSH 터널링 설정</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/b6813aa9-85ec-4305-87b5-4b403c0e3a82/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/7c3bf1ac-7ef0-473a-b14a-c6c840e19cf7/image.png" />
&emsp;위와 같이 입력할 경우, 터미널 창이 열려 있는 동안, 포트 포워딩이 유지되고 13306 포트로 RDS에 접속이 가능하다.</p>
<ul>
<li><code>-L</code> : Local 포트포워딩</li>
<li><code>13306</code> : local, 즉 ssh-client가 로컬에서 사용할 포트 번호</li>
<li><code>RDS주소:3306</code> : 접근할 RDS 서버 주소와 사용 포트</li>
<li><code>EC2 주소</code> : SSH 서버</li>
</ul>
<p>&emsp;만약 터미널 창이 열려 있는 동안만 터널링이 유지되는 것이 불편하다면 nohup으로 서버를 백그라운드로 실행시키듯이 옵션들을 포함시켜 백그라운드로 실행 가능하다.</p>
<ul>
<li><code>-f</code> : 백그라운드로 실행</li>
<li><code>-N</code> : -f 옵션만 지정할 경우 'Cannot fork into background without a command to execute.' 에러가 나기 때문에 -N 옵션을 함께 지정해 주어야 한다. </li>
<li><code>-v</code> : 디버깅용 로그를 표시</li>
</ul>
<h3 id="2-intellij-이용">2. IntelliJ 이용</h3>
<p>&emsp;이 방법으로 매번 CMD 창으로 터널링을 진행하기에는 불편한 점이 많다. 인텔리제이에서 데이터베이스에 등록하는 방식으로 바로 편하게 사용이 가능하다. </p>
<h4 id="1-intellij-우측-상단의-데이터베이스-클릭">&nbsp;&nbsp; 1) IntelliJ 우측 상단의 데이터베이스 클릭</h4>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/e3457bc8-f9b1-49e7-acb9-accafc9a7e71/image.png" /></p>
<h4 id="2-sshssl-탭에서-use-ssh-tunnel-사용">&nbsp;&nbsp; 2) SSH/SSL 탭에서 Use SSH Tunnel 사용</h4>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/cd53788f-7029-468a-b5f1-2b1eb435055b/image.png" /></p>
<p>&emsp;SSH 설정은 각자 환경에 맞게 설정하면 된다. 해당 프로젝트에서는 pem 키를 사용했기 때문에 pem 키 경로를 지정해 접근하도록 했다.  </p>
<h4 id="3-general-탭에서-아래-항목들을-세팅">&nbsp;&nbsp; 3) general 탭에서 아래 항목들을 세팅</h4>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/f890ebfa-2213-4b53-8e66-854179bb0ce6/image.png" /></p>
<p>&emsp;일반적으로 로컬 MySQL 에서 3306 포트를 사용중일 것이기 때문에 터널링용 포트는 13306 등 다른 것으로 해주어야 한다.</p>
<h2 id="결론">결론</h2>
<p>&emsp;이번 기회를 통해 private subnet에 위치한 RDS에 로컬 환경에서 SSH 터널링으로 접속하는 방법을 알아보았다. 일반적으로 프로젝트에서 RDS를 private 서브넷에 생성하고 EC2와의 인바운드 규칙만 허용하는데, SSH 터널링을 활용하면 EC2 서버에 직접 접속하지 않고도 로컬 환경에서 편리하게 개발을 진행할 수 있다.</p>
<h3 id="참고문헌">참고문헌</h3>
<p><a href="https://neverfadeaway.tistory.com/42">ssh 포트포워딩(터널링) 뚫기 (feat.로컬에서 AWS VPC 내 RDS 접속하기</a></p>