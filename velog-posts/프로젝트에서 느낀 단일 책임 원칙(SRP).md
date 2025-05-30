<h2 id="srp단일-책임-원칙-왜-신경-써야-할까">SRP(단일 책임 원칙), 왜 신경 써야 할까?</h2>
<p>단일 책임 원칙(Single Responsibility Principle)은<br />SOLID 원칙 중 첫 번째 원칙으로, 아래와 같이 정의됩니다.</p>
<blockquote>
<p>&quot;A class should have only one reason to change.&quot;<br />클래스는 오직 하나의 변경 이유만 가져야 한다.</p>
</blockquote>
<p>조금 더 현실적으로 말하면,<br /><strong>&quot;하나의 클래스는 하나의 역할만 맡아야 유지보수가 쉽다&quot;</strong>는 뜻입니다.</p>
<hr />
<h3 id="🤔-왜-srp가-중요할까">🤔 왜 SRP가 중요할까?</h3>
<p>개발을 하다 보면,<br />“하나의 흐름인데 어차피 이 클래스에서 이 작업도 같이 하면 편하잖아?” 
라는 생각으로 <strong>조금씩 다른 역할들을 한 클래스에 넣게 되는 경우가 흔하게 보이곤 합니다.</strong></p>
<p>처음엔 괜찮아 보여도, 점점 클래스가 비대해지고 책임이 섞이게 되며 문제가 드러납니다.</p>
<ul>
<li>클래스 하나를 수정하려 했는데, 다른 기능까지 영향을 받는 구조가 되어버리고</li>
<li>단위 테스트가 어려워지고</li>
<li>기능 확장이 어렵고</li>
<li>다른 팀원이 합류 했을 때 코드의 의도를 파악하기 어려워집니다</li>
</ul>
<p>이런 상황을 방지하기 위해 <strong>단일 책임 원칙은 구조의 건강함을 지키는 최소한의 기준</strong>이 되어줍니다</p>
<hr />
<h2 id="💡-실전에서-겪은-srp-위반-사례">💡 실전에서 겪은 SRP 위반 사례</h2>
<p>제가 참여했던 프로젝트 <strong><a href="https://www.solve-nyang.com/">솔브냥</a></strong>에서는<br />사용자가 문제를 풀면, 난이도에 따라 점수를 부여하는 기능이 있었습니다.</p>
<p>이 로직은 <code>Problem</code> 클래스에 다음과 같이 작성되어 있었어요.</p>
<pre><code class="language-java">public void updateProblemCountAndGivePoint(int level, 
                                    int newSolvedCount) {
    updateProblemCount(level, newSolvedCount);
    member.addPoint(calculatePoint(level)); // ❌ SRP 위반
}</code></pre>
<h2 id="🤯-이-코드의-문제점은">🤯 이 코드의 문제점은?</h2>
<p>Problem 클래스는 본래 문제 풀이 기록만 관리하는 책임을 가져야 합니다.</p>
<p>그런데 여기에 포인트 보상 로직까지 함께 다루게 되면서, 두 가지 책임이 뒤섞였습니다.</p>
<p>포인트 정책이 변경될 경우, Problem 클래스까지 수정해야 하고
→ 그로 인해 클래스가 여러 이유로 변경될 수 있는 구조가 되어버렸습니다.</p>
<hr />
<h2 id="🔨-srp를-지킨-리팩토링">🔨 SRP를 지킨 리팩토링</h2>
<p>이 문제를 해결하기 위해, 책임을 다음과 같이 분리했습니다.</p>
<p>문제 풀이 수는 Problem 클래스에서만 관리</p>
<p>포인트 보상은 서비스 계층이나 Member 쪽에서만 관리</p>
<pre><code class="language-java">@Transactional
public void handleProblemSolved(Member member, 
                            int level, int newSolvedCount) {
    Problem problem = problemRepository.findByMember(member);
    // 문제 기록은 Problem의 책임
    problem.updateProblemCount(level, newSolvedCount); 
    // 포인트는 Member의 책임
    member.addPoint(calculatePoint(level)); 
}</code></pre>
<p>이렇게 분리한 뒤에는:</p>
<p>각 클래스가 자기 책임만 가지게 되었고 로직을 파악하기 쉬워졌으며</p>
<p>테스트와 유지보수도 깔끔하게 분리되었습니다</p>
<hr />
<h2 id="👥-협업-중-깨달은-srp의-진짜-의미">👥 협업 중 깨달은 SRP의 진짜 의미</h2>
<p>이 코드를 처음 작성한 팀원은 자바 개발이 처음이었고,
문제 풀이와 포인트 보상이 자연스럽게 연결되다 보니 &quot;같이 처리하는 게 맞지 않을까?&quot;라는 생각으로 작성했습니다.</p>
<p>하지만 이 경험을 통해 저는 SRP가 단순한 이론이 아니라,
협업 속에서 코드의 방향성을 지켜주는 중요한 설계 기준이라는 걸 느낄 수 있었습니다.</p>
<hr />
<h3 id="📌-srp를-지켜야-하는-이유">📌 SRP를 지켜야 하는 이유</h3>
<table>
<thead>
<tr>
<th>이유</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>✅ 유지보수성</td>
<td>기능 변경 시, 어디만 고치면 되는지 명확함</td>
</tr>
<tr>
<td>✅ 테스트 용이성</td>
<td>클래스가 자기 책임만 갖고 있어 테스트 격리 쉬움</td>
</tr>
<tr>
<td>✅ 협업 안정성</td>
<td>여러 개발자가 동시에 작업해도 충돌 가능성 낮음</td>
</tr>
<tr>
<td>✅ 구조 가독성</td>
<td>한눈에 클래스의 역할이 명확해짐</td>
</tr>
</tbody></table>
<hr />
<h2 id="✉️-마무리하며">✉️ 마무리하며</h2>
<p>SRP는 &quot;굳이 이렇게까지 분리해야 하나?&quot; 싶을 수도 있지만,
한 번 책임이 섞이기 시작하면 결국 구조는 얽히고 꼬이게 됩니다.</p>
<p>이번 경험을 통해, 저는 SRP를 <strong>&quot;코드를 장기적으로 건강하게 유지하기 위한 기본&quot;</strong> 이라고 생각하게 됐습니다.</p>