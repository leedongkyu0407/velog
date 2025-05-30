<h2 id="개요">개요</h2>
<p>&emsp;springboot를 이용하여 프로젝트를 진행하며 객체 생성 시 @Builder 어노테이션을 통해 builder 패턴을 사용해 왔으나 어느 순간부터 사용 이유나 다른 객체 생성 방식들과의 장단점을 고려하지 않고 무분별하게 습관적으로 사용한다고 느꼈고 이번 기회에 장단점을 정리하고자 한다.</p>
<h2 id="객체-생성-방법">객체 생성 방법</h2>
<p>일반적으로 자바에서 객체 생성 방법은 세 가지가 있다.</p>
<h3 id="1-점층적-생성자-방식">1. 점층적 생성자 방식</h3>
<h3 id="2-자바-빈즈-방식">2. 자바 빈즈 방식</h3>
<h3 id="3-빌더-패턴">3. 빌더 패턴</h3>
<p>&emsp;먼저, <strong>점층적 생성자 방식</strong>은 필수 매개변수와 함께 선택 매개변수를 0개, 1개, 2개… 받는 형태로 메서드 오버로딩 방식을 통해 추가적으로 작성해 놓는 방식이다. </p>
<p>&emsp;이러한 방식은 구현이 쉽다는 장점이 있지만, 매개 변수가 많아질수록 생성자 메서드의 수가 기하급수적으로 늘어나고 객체 생성 시에 몇 번째 인자가 어떤 필드에 해당하는지 헷갈릴 여지가 많다.</p>
<p>&emsp;아래 예시의 경우에도 매개 변수 1개일 경우를 username 필드에 대해서만 고려했지만 모든 필드마다 각각 개수를 고려한다고 하면 총 필요한 생성자 메서드의 수는 더욱 늘어나게 된다.</p>
<pre><code class="language-java">public class Member {

    private String username;
    private String password;
    private Long point;

    // 매개변수 0개
    Member(){}

    // 매개변수 1개
    Member(String username){
        this.username = username;
    }

    // 매개변수 2개
    Member(String username, String password){
        this.username = username;
        this.password = password;
    }

    // 매개변수 전체
    Member(String username, String password, Long point){
        this.username = username;
        this.password = password;
        this.point = point;
    }
}
</code></pre>
<p>&emsp;두 번째로, <strong>자바 빈즈 방식</strong>은 이를 보완하기 위해 setter 메서드를 활용하는 객체 생성 방식이다. 매개 변수가 없는 객체로 객체를 생성 후 각 필드에 해당하는 setter 메서드들을 활용하여 객체의 초기값을 설정해주는 방식이다. </p>
<p>&emsp;이를 통해 유연한 객체 생성이 가능해지고 직관적으로 필드에 맞는 매개 변수를 전달할 수 있다는 장점이 있지만 객체 생성 시점과 초기화 시점이 달라지기 때문에 <strong>일관성</strong>과 <strong>불변성</strong> 문제를 가져 객체의 무결성을 보장하기 어렵게 된다. 뿐만 아니라 객체 생성과 초기화 시점이 다르기에 멀티 쓰레드 환경에서 동기화 문제가 발생할 여지가 있다.</p>
<pre><code class="language-java">package com.ssafy.solvedpick.member.domain;

public class Member {

    private String username;
    private String password;
    private Long point;

    Member(){}

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setPoint(Long point) {
        this.point = point;
    }
}</code></pre>
<pre><code class="language-java">
Member member = new Member();
member.setUsername(&quot;username&quot;);
member.setPassword(&quot;1234&quot;);
member.setPoint(1000L);
</code></pre>
<h4 id="일관성-문제">일관성 문제</h4>
<p>&emsp;일관성이란 객체가 항상 유효하고 예상 가능한 상태를 유지해야 한다는 원칙을 말한다. 자바 빈즈 방식의 경우 객체 생성 시점과 초기화 시점이 다르기에 객체를 생성하는 개발자가 실수로 필수 매개변수를 설정하는 setter 메서드를 호출하지 않을 경우 해당 객체는 유효하지 않은, 즉 일관성이 무너진 상태가 된다. 만약 해당 인스턴스를 다른 곳에서 사용한다면 런타임 에러가 발생할 수 있다. </p>
<h4 id="불변성-문제">불변성 문제</h4>
<p>&emsp;불변성이란 객체가 최초 생성된 이후 값이 변하지 않는다는 것을 의미한다. 자바 빈즈는 빈 객체를 생성하고 setter로 객체를 초기화하는데 문제는 setter를 다른 개발자가 다른 곳에서 다른 시점에도 사용 가능하다는 점이다. 이는 시스템에 있어 치명적 오류를 불러올 수 있으며 자바 빈즈는 생성 방식의 구조적 문제로 인해 불변성을 보장할 수 없다.</p>
<p>&emsp;위에 기술한 일관성 문제의 경우 생성자(constructor)와 결합하여 어느 정도 해결 가능하지만 불변성 문제는 자바 빈즈 방식 자체의 구조적 문제이기 때문에 해결이 불가능하다. 따라서, 자바 빈즈 방식은 지양하는 것이 맞다.</p>
<p>&emsp;마지막으로 살펴볼 <strong>빌더 패턴</strong>은 위의 문제들을 해결하기 위해 별도의 builder 클래스를 만들어 메서드 체이닝을 통해 값을 입력 받고 build() 메서드를 이용해 최종적으로 하나의 인스턴스를 만들어 리턴하는 객체 생성 패턴이다. 나는 기존에 빌더 패턴을 이용하기 위해 Lombok에서 어노테이션을 통해 지원하는 @Builder 어노테이션을 사용했으나 이번 기회에 정확히 builder 클래스를 직접 생성해 빌더 패턴을 적용해 보았다.</p>
<pre><code class="language-java">public class Member {

    private String username;
    private String password;
    private Long point;

    public Member(String username, String password, Long point) {
        this.username = username;
        this.password = password;
        this.point = point;
    }
}</code></pre>
<pre><code class="language-java">public class MemberBuilder {

    private String username;
    private String password;
    private Long point;

    public MemberBuilder username(String username) {
        this.username = username;
        return this;
    }

    public MemberBuilder password(String password) {
        this.password = password;
        return this;
    }

    public MemberBuilder point(Long point) {
        this.point = point;
        return this;
    }

    public Member build() {
        return new Member(username, password, point);
    }
}
</code></pre>
<pre><code class="language-java">@Service
public class MemberService {

    public Member create(String username, String password, Long ponit) {
        return new MemberBuilder()
                .username(username)
                .password(password)
                .point(ponit)
                .build();
    }
}</code></pre>
<h3 id="빌더-패턴의-장점">빌더 패턴의 장점</h3>
<h4 id="1-가독성이-좋다">1. 가독성이 좋다</h4>
<p>&emsp;생성자 방식으로 객체를 생성하는 경우, 매개 변수가 많아질수록 가독성이 급격하게 떨어진다. </p>
<p>&emsp;반면, 빌더 패턴을 적용하면 직관적으로 어떤 필드에 어떤 매개변수가 매칭 되는지 한 눈에 파악할 수 있게 된다. 특히, 동일 타입의 매개 변수를 연속하여 설정해야 할 경우에 발생할 수 있는 오류와 같은 실수를 방지할 수 있다.</p>
<h4 id="2-객체의-불변성-보장">2. 객체의 불변성 보장</h4>
<p>&emsp;Setter를 사용하지 않기 때문에 객체의 불변성이 보장된다. 불변 객체는 객체 생성 이후 내부의 상태가 변하지 않는 객체로 불변 객체는 오로지 읽기(get)  메소드만을 제공하며 쓰기(set) 메서드는 제공하지 않는다. 대표적으로 자바에서 final 키워드를 붙인 변수가 바로 불변이다.</p>
<p>&emsp;불변 객체를 이용해 개발해야 하는 이유는 다음과 같다.</p>
<ul>
<li>불변 객체는 Thread-Safe 하기 때문에 동기화를 고려하지 않아도 된다.</li>
<li>다른 사람이 개발한 함수를 이용할 때 안정성을 보장할 수 있어 협업과 유지보수에 유용하다.</li>
</ul>
<p>&emsp;예를 들어, 클래스를 만드는 개발자와 이를 이용하여 서비스 로직을 만드는 개발자가 다르다고 할 때, 객체를 불변으로 만들지 않는다면 초기 설계에 따라 객체의 상태가 유지된다는 보장이 없다.</p>
<p>&emsp;따라서, 클래스들은 가변적이어야 하는 타당한 이유가 없다면 반드시 불변으로 만들어야 한다. 불변으로 만드는 것이 불가능 하다면 가능한 변경 가능성을 최소화해야 한다.</p>
<p>&emsp;즉, 빌더 패턴은 생성자 없이 어느 객체에 대해 변경 가능성의 최소화를 추구하여 불변성을 갖게 해주는 것이다.</p>
<h4 id="3-디폴트-매개변수를-간접적으로-지원">3. 디폴트 매개변수를 간접적으로 지원</h4>
<p>&emsp;디폴트 매개변수란 인자 값을 생략해도 설정해 놓은 기본값이 할당 되는 것을 말한다. 파이썬에서는 메서드에 대해 기본적으로 디폴트 매개 변수를 지원하지만 자바의 경우는 해당 기능을 지원하지 않는다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/2f5b4707-6cc4-4f23-aadb-2720a96eeda6/image.png" /></p>
<p>이를 빌더 패턴을 이용하면 디폴트 매개변수가 설정된 필드를 설정하는 메서드를 호출하지 않는 방식으로 별도의 생성자 오버로딩 없이 구현 가능하다</p>
<pre><code class="language-java">package com.ssafy.solvedpick.member.domain;

public class MembersBuilder {

    private String username;
    private String password=&quot;1234&quot;;
    private Long point;

    public MembersBuilder username(String username) {
        this.username = username;
        return this;
    }

    public MembersBuilder password(String password) {
        this.password = password;
        return this;
    }

    public MembersBuilder point(Long point) {
        this.point = point;
        return this;
    }

    public Members build() {
        return new Members(username, password, point);
    }
}
</code></pre>
<pre><code class="language-java">@Service
public class MembersService {

    public Members create(String username, Long point) {
        return new MembersBuilder()
                .username(username)
                .point(point)
                .build();
    }
}</code></pre>
<h3 id="빌더-패턴-단점">빌더 패턴 단점</h3>
<h4 id="1-코드-복잡성-증가">1. 코드 복잡성 증가</h4>
<p>&emsp;클래스에 빌더 패턴을 적용하려면 추가적으로 위와 같이 빌더 클래스를 만들어야 해서 관리해야 할 클래스가 늘어나고 구조가 복잡해질 수 있다. 다만, 일반적으로 lombok을 통해 @Builder 어노테이션을 사용하기 때문에 크게 문제가 될 여지는 크지 않다.</p>
<h4 id="2-성능-저하">2. 성능 저하</h4>
<p>&emsp;생성자 방식과 달리 매번 빌더를 거쳐 인스턴스화 하기 때문에 생성자 방식 보다 성능이 떨어질 수 있다. 일반적으로 해당 리소스 자체가 크지 않지만 만약 성능이 극도로 중요한 임베디드와 같은 환경이라면 문제가 발생할 여지가 있다. (일반적인 웹개발에서는 크게 문제가 되지 않는다고 생각한다.)</p>
<h3 id="builder-어노테이션-주의점">@Builder 어노테이션 주의점</h3>
<p>&emsp;@Builder 어노테이션을 클래스에 적용하면 Lombok이 내부적으로 모든 필드를 포함하는 private 생성자를 자동으로 생성하는데 이는 @AllArgsConstructor(access = AccessLevel.PRIVATE)와 동일한 효과를 가진다.</p>
<h2 id="결론">결론</h2>
<p>&emsp;그렇다면 과연 빌더 패턴을 언제 사용해야 할까?(@Builder를 어떤 클래스에 붙이는 게 좋은가)</p>
<p>&emsp;이론적으로는 파라미터의 개수가 많지 않다면 빌더 패턴을 사용하는 것은 오히려 복잡성을 증대시키는 문제가 있다. 하지만, 개발을 진행하다 보면 많은 경우에 있어 파라미터의 개수가 늘어나는 일이 많고 특히, lombok을 사용한다면 추가 클래스 선언 없이 사용 가능하기 때문에 적절히 빌더 패턴을 사용한다면 단점에 비해 장점이 많다고 생각된다.</p>