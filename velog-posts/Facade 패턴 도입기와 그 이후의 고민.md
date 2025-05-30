<h2 id="서비스-구조-발전기-facade-패턴-도입기와-그-이후의-고민">서비스 구조 발전기: Facade 패턴 도입기와 그 이후의 고민</h2>
<p>프로젝트가 커지면 커질수록, 서비스 구조는 단순하게 유지하기 어려워집니다.<br />처음에는 Controller → Service → Repository 구조만 잘 지켜도 충분했는데,<br />어느새 <strong>서비스 간 순환 참조</strong>, <strong>비대해지는 책임</strong>, <strong>복잡해지는 흐름</strong>까지 고민해야 했습니다.</p>
<p>이 글은 그런 현실적인 문제를 겪고, <strong>Facade 패턴</strong>을 도입하게 된 경험과,<br />그 이후 <strong>이벤트 기반 처리에 대한 고민</strong>까지 담은 이야기입니다.</p>
<hr />
<h2 id="🧱-1-service에서-다른-service를-직접-호출">🧱 1. Service에서 다른 Service를 직접 호출</h2>
<p>처음엔 비즈니스 흐름을 자연스럽게 만들기 위해,
Service 간 직접 호출로 필요한 로직을 구성했습니다.</p>
<pre><code class="language-java">// Ex. MemberService
public void register(MemberRequest request) {
    Member member = memberRepository.save(request.toEntity());
    problemService.init(member); // 다른 Service 호출
    memberDisplayService.init(member);
    apiService.sync(member);
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/40a4e4e3-8f15-4863-b5c0-00e9e4a6e610/image.png" /></p>
<p>이 방식은 기능을 한 곳에서 조합하기 쉬웠지만,
서비스 간 의존 관계가 얽히기 시작하면서
<strong>&quot;이러다 순환 참조 생기겠는데?&quot;</strong>라는 불안감이 생겼습니다.</p>
<hr />
<h2 id="🧪-2-순환-참조-가능성을-회피하기-위해-controller에서-service-여러-개-호출">🧪 2. 순환 참조 가능성을 회피하기 위해 Controller에서 Service 여러 개 호출</h2>
<p>그래서 Service 간 호출을 끊고, Controller에서 직접 여러 Service를 호출하도록 구조를 바꿨습니다.</p>
<pre><code class="language-java">@PostMapping(&quot;/register&quot;)
public void register(@RequestBody MemberRequest request) {
    Member member = memberService.save(request);
    problemService.init(member);
    memberDisplayService.init(member);
    apiService.sync(member);
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/63e72f74-bfe0-4d5f-9590-312695c1060b/image.png" />
순환 참조 문제는 피했지만,
컨트롤러 단이 점점 비즈니스 로직으로 더러워지기 시작했습니다.</p>
<hr />
<h2 id="🧯-3-service에서-repository를-직접-호출">🧯 3. Service에서 Repository를 직접 호출</h2>
<p>&quot;컨트롤러가 너무 많은 책임을 지는 것 같다&quot;고 느껴서,
이번에는 다시 Service에서 처리하되 <strong>다른 Service를 호출하지 않고 Repository를 직접 사용하는 방식</strong>으로 바꿨습니다.</p>
<pre><code class="language-java">// Ex. UserService
public void register(Member member) {
    memberRepository.save(member);
    problemRepository.save(...); // 다른 도메인 직접 접근
    memberDisplayRepository.save(...);
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/0fce8095-8e83-4d59-b8df-32f808df4841/image.png" /></p>
<p>하지만 이 방식도 <strong>SRP(단일 책임 원칙)</strong>에 어긋나는 느낌이 들었고,
<strong>기존 Service에 이미 구현된 기능을 중복 구현</strong>해야 하는 상황이 계속 생겼습니다.</p>
<hr />
<h2 id="🛠-4-facade-패턴-도입">🛠 4. Facade 패턴 도입</h2>
<p>이런 흐름 끝에 도입한 구조가 <strong>Facade 패턴</strong>이었습니다.
비즈니스 로직은 각 도메인 Service에 맡기고,
<strong>전체 흐름은 하나의 Facade에서 조율</strong>하도록 만들었습니다.</p>
<h3 id="📦-facade-패턴이란">📦 Facade 패턴이란?</h3>
<blockquote>
<p><strong>복잡한 내부 시스템을 감추고,
외부에는 단순한 인터페이스만 제공하는 디자인 패턴</strong></p>
</blockquote>
<p>현실 세계의 예를 들어,
만약 커피를 내리기 위해 물 끓이기, 원두 갈기, 압력 조절, 추출 등 여러 단계를 거쳐야 한다면
이걸 하나하나 직접 하는 대신 <strong>‘커피 머신’이라는 하나의 인터페이스를 사용하는 것</strong>이 Facade 패턴입니다.</p>
<p>소프트웨어에서의 Facade도 유사합니다.
여러 서비스의 로직이 복잡하게 얽혀 있을 때, 이를 <strong>하나의 조율자(Facade)</strong>가 감싸고
외부에서는 <strong>&quot;initializeUser()&quot; 한 줄로 처리</strong>하는 겁니다.</p>
<h3 id="💡-언제-facade를-쓰면-좋을까">💡 언제 Facade를 쓰면 좋을까?</h3>
<ul>
<li><p>여러 서비스가 협력해야 하나, 외부에서는 단순하게 호출하고 싶을 때</p>
</li>
<li><p>컨트롤러에서 여러 서비스 호출이 반복될 때</p>
</li>
<li><p>서비스 간 순환 참조가 발생하거나 그럴 위험이 있을 때</p>
</li>
<li><p>도메인 로직은 잘 분리되어 있는데, 흐름을 하나로 묶고 싶을 때</p>
</li>
</ul>
<p>즉, <strong>복잡한 비즈니스 로직의 흐름을 한 곳에서 조율하고 싶을 때</strong> Facade는 훌륭한 해결책이 됩니다.</p>
<hr />
<h2 id="🔁-5-프로젝트-적용-예시">🔁 5. 프로젝트 적용 예시</h2>
<p>저는 Facade를 이런 식으로 활용했습니다</p>
<p><strong>Before</strong></p>
<blockquote>
<p>Controller
   ↓
MemberService
   ├── ProblemService
   └── MemberDisplayService</p>
</blockquote>
<p><strong>After</strong></p>
<blockquote>
<p>Controller
   ↓
UserFacade
   ├── MemberService
   ├── ProblemService
   └── MemberDisplayService</p>
</blockquote>
<p>그리고 실제로 UserFacade에는 다음과 같은 흐름이 들어있습니다.</p>
<pre><code class="language-java">public void initializeNewUserInfo(Member member) {
    problemService.save(...);
    memberDisplayService.save(...);
    syncUserInfo(member); // 외부 API와의 동기화
}
</code></pre>
<p>이를 통해 컨트롤러에서는 다음과 같이 한 줄로 처리가 가능해졌습니다.</p>
<pre><code class="language-java">@PostMapping(&quot;/register&quot;)
public void register(@RequestBody MemberRequest request) {
    Member member = memberService.save(request);
    userFacade.initializeNewUserInfo(member);
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/a25ad43a-cc70-4847-975c-7e4f47854cc8/image.png" /></p>
<hr />
<h2 id="🤯-6-facade의-한계-facade의-비대화">🤯 6. Facade의 한계: Facade의 비대화</h2>
<p>Facade 도입 후 순환 참조는 해결됐고, 로직 흐름도 깔끔해졌습니다.
하지만 기능이 늘어나면서 하나의 Facade 클래스가 점점 커지기 시작했습니다.</p>
<blockquote>
<p>“이전에는 Service가 비대했는데…
이제는 Facade가 비대해진 건 아닐까?”</p>
</blockquote>
<p>비즈니스 흐름이 많아질수록 Facade는 자연스럽게 거대한 조율자가 되어버렸습니다.
이건 &quot;책임의 이전&quot;이지, 완전한 해결은 아니라고 생각했습니다.</p>
<h3 id="🧭-이벤트-기반-처리-고려">🧭 '이벤트 기반 처리' 고려</h3>
<p>이런 이유로 지금은 Facade 이후의 다음 구조로 이벤트 기반 설계를 고민하고 있습니다.</p>
<ul>
<li><p>유저 가입 → NewUserRegisteredEvent 발행</p>
</li>
<li><p>각 도메인의 Listener가 자신이 필요한 작업만 수행</p>
</li>
</ul>
<p>이렇게 되면:</p>
<p>Facade의 비대화를 방지하고 도메인 간 결합도를 낮출 수 있으며
나아가 확장성과 병렬성도 확보할 수 있을 것이라 생각하고 있습니다.
아마 조만간 다른 글을 통해 이벤트 기반 처리와 실제 적용 사례를 설명할 수 있을 것 같습니다.</p>
<hr />
<h2 id="🧠-마무리하며-구조는-계속해서-다듬어져야-한다">🧠 마무리하며: 구조는 계속해서 다듬어져야 한다</h2>
<p>이번 경험을 통해 깨달은 건 하나입니다.</p>
<blockquote>
<p>“좋은 구조란, 한 번에 완성되는 것이 아니라 지속적인 고민 속에서 다듬어지는 것이다.”</p>
</blockquote>
<p>순환 참조를 막기 위해 Facade를 도입하고,
Facade가 비대해지는 걸 막기 위해 이벤트 구조를 고민하고...
이 모든 흐름이 결국 더 나은 구조를 위한 여정이라는 걸 느끼고 있습니다.
그때그때 가장 적절한 방법을 선택하고, 필요할 때 다시 개선해가는 유연함이 중요한 것 같습니다.</p>