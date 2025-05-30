<h2 id="개요">개요</h2>
<p>&nbsp;&nbsp; 이전 포스팅에서는 EC2에 직접 접속하여 SpringBoot 프로젝트를 배포하는 방법에 대해 알아보았다. 하지만 이 방식은 수동 작업이 많아 반복적이고 번거로운 과정이 많아 Gihub Actions를 사용하여 CI/CD 파이프라인을 구축하여 배포 자동화를 적용하였다. 이번 글에서는 더 효율적인 배포 방식인 GitHub Actions를 사용한 CI/CD 파이프라인에 대한 개념과 장점에 대해 알아보려 한다.</p>
<h3 id="cicd란">CI/CD란?</h3>
<p>&nbsp;&nbsp; CI/CD는 현대 소프트웨어 개발 프로세스에 있어 핵심 요소로, 애플리케이션 개발 단계를 자동화하여 더 짧은 주기로 고객에게 제공할 수 있도록 하는 방법론이다. 자세히 살펴보기에는 CI/CD만 따로 분리해서 하나의 글로 써야 할 정도로 내용이 많지만 현재 포스팅의 주제는 CI/CD가 아니므로 간단히 이런 것이 있다 정도로 넘어가려고 한다. </p>
<h4 id="1-cicontinuous-integration-지속적-통합">&nbsp;&nbsp;1. CI(Continuous Integration, 지속적 통합)</h4>
<p>&nbsp;&nbsp;CI는 개발자들이 코드 변경사항을 중앙에 자주 병합하는 개발 방식이다. 일반적으로 다음과 같은 과정을 자동화한다. </p>
<ul>
<li>코드 변경 사항을 정기적으로 빌드</li>
<li>자동화된 테스트 실행</li>
<li>코드 품질 검사</li>
</ul>
<p>&nbsp;&nbsp;CI의 주요 목표는 버그를 신속하게 발견하고, 소프트웨어 품질을 개선하며, 새로운 업데이트의 검증 및 출시 시간을 단축하는 것이다.</p>
<h4 id="2-cdcontinuous-deliverydeployment-지속적-배포">&nbsp;&nbsp;2. CD(Continuous Delivery/Deployment, 지속적 배포)</h4>
<p>&nbsp;&nbsp; CD는 일반적으로 두 가지 의미로 사용된다. </p>
<ul>
<li>Continuous Delivery(지속적 전달)<ul>
<li>모든 코드 변경 사항이 테스트 환경이나 프로덕션 환경에 자동으로 배포될 준비가 되게 하는 것으로 최종 프로덕션 배포는 수동 승인을 통해 이뤄진다.</li>
</ul>
</li>
<li>Continuous Deployment(지속적 배포)<ul>
<li>코드 변경사항이 파이프라인의 모든 단계를 자동으로 통과한 후 사용자에게 자동으로 배포되는 것을 의미한다. 수동 개입 없이 프로덕션까지 자동화된다.</li>
</ul>
</li>
</ul>
<h3 id="cicd의-장점">CI/CD의 장점</h3>
<p>&nbsp;&nbsp; 그렇다면 왜 굳이 CI/CD를 적용하여 배포를 진행할까?</p>
<ul>
<li>빠른 피드백<ul>
<li>문제를 조기에 발견 가능</li>
</ul>
</li>
<li>위험 감소<ul>
<li>작은 변경사항을 자주 배포하므로 각 배포의 위험이 감소</li>
</ul>
</li>
<li>생산성 향상<ul>
<li>수동 프로세스 대신 자동화를 통해 생산성 향상에 기여</li>
</ul>
</li>
<li>신뢰성 있는 배포<ul>
<li>일관된 프로세스로 인적 오류 감소 가능</li>
</ul>
</li>
<li>빠른 출시<ul>
<li>시장에 더 빠르게 제품 출시 가능</li>
</ul>
</li>
</ul>
<h2 id="github-actions란">GitHub Actions란?</h2>
<p>&nbsp;&nbsp;GitHub Actions는 Github에서 제공하는 워크플로우 자동화 도구로, 소프트웨어 개발 워크플로우의 다양한 작업을 자동화할 수 있다. 코드 저장소에서 직접 CI/CD 파이프라인을 구축하고 실행할 수 있어 별도의 외부 도구나 서비스 없이도(ex. Jenkins) 개발, 테스트, 배포 과정을 자동화 할 수 있다.</p>
<h3 id="github-acitons-특징">Github Acitons 특징</h3>
<h4 id="1-github-통합">&nbsp;&nbsp;1. GitHub 통합</h4>
<p>&nbsp;&nbsp; GitHub 저장소와 완벽하게 통합되어 있어 코드 변경, 이슈 생성, 풀 리퀘스트 등 다양한 GitHub 이벤트에 반응할 수 있다.</p>
<h4 id="2-워크플로우-기반">&nbsp;&nbsp;2. 워크플로우 기반</h4>
<p>&nbsp;&nbsp; YAML 파일로 정의된 워크플로우를 통해 자동화 과정을 구성한다. 이 파일은 .github/workflows/ 디렉토리에 저장된다.</p>
<h4 id="3-다양한-실행-환경">&nbsp;&nbsp;3. 다양한 실행 환경</h4>
<p>&nbsp;&nbsp; Linux, Windows, macOS 등 다양한 환경에서 워크플로우를 실행할 수 있다.</p>
<h4 id="4-재사용-가능한-액션">&nbsp;&nbsp;4. 재사용 가능한 액션</h4>
<p>&nbsp;&nbsp; 커뮤니티에서 제공하는 수많은 사전 구성된 액션을 활용하거나, 직접 커스텀 액션을 만들어 사용할 수 있다.</p>
<h4 id="5-매트릭스-빌드">&nbsp;&nbsp;5. 매트릭스 빌드</h4>
<p>&nbsp;&nbsp; 여러 버전의 런타임이나 OS에서 동시에 테스트를 실행할 수 있다.</p>
<h4 id="6-시크릿-관리">&nbsp;&nbsp;6. 시크릿 관리</h4>
<p>&nbsp;&nbsp; 민감한 정보를 GitHub Secrets에 안전하게 저장하고 워크플로우 내에서 사용할 수 있다.</p>
<h3 id="기존-ec2-직접-배포-방식과의-차이점">기존 EC2 직접 배포 방식과의 차이점</h3>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/f8a1ecd6-ee07-4fe6-95e0-72fc921828f0/image.png" /></p>
<h2 id="마무리">마무리</h2>
<p>&nbsp;&nbsp; 이번 글에서는 CI/CD의 개념과 GitHub Actions의 특징에 대해 알아보았다. 다음 글에서는 실제로 GitHub Actions를 사용하여 SpringBoot 프로젝트를 AWS EC2에 자동으로 배포하는 구체적인 방법과 코드에 대해 알아보도록 하겠다.</p>
<h2 id="참고문헌">참고문헌</h2>
<p><a href="https://artist-developer.tistory.com/24">CI/CD란 무엇인가?</a></p>