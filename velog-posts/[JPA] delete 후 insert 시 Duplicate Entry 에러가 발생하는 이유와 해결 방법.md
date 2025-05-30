<blockquote>
<p>@OneToMany, UniqueConstraint, deleteAll → insert 로직에서 발생하는 SQLIntegrityConstraintViolationException 해결기</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/donggyu47/post/6e251683-ecc1-4f87-a206-c4b060080ffe/image.png" /></p>
<h2 id="문제-상황">문제 상황</h2>
<p>Spring Boot + JPA 기반 프로젝트에서 Room 엔티티가 관리하는 RoomYear 리스트를 갱신할 때, 다음과 같은 에러가 발생했다.</p>
<pre><code class="language-nginx">Duplicate entry '01JQKMZ5J5RNK746CMAF1QDRXM-2024' for key 'room_year.uk_room_year'</code></pre>
<ul>
<li>이 에러는 room_id와 year를 복합 유니크 키로 설정한 상황에서, 같은 transaction 내에서 deleteAll() 후 insert를 연이어 수행했을 때 발생했다.</li>
</ul>
<hr />
<h2 id="초기-구조">초기 구조</h2>
<p>초기에는 RoomYear → Room 단방향 연관관계만 존재했다.</p>
<pre><code class="language-java">@Entity
public class RoomYear {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = &quot;room_id&quot;, nullable = false)
    private Room room;

    @Column(name = &quot;year&quot;, columnDefinition = &quot;YEAR&quot;, nullable = false)
    private Integer year;
}</code></pre>
<p>Room 엔티티는 RoomYear에 대한 연관관계를 갖고 있지 않았다.</p>
<hr />
<h2 id="연도-업데이트-로직">연도 업데이트 로직</h2>
<pre><code class="language-java">
roomYearRepository.deleteAllByRoom(room); // 기존 연도 삭제
List&lt;RoomYear&gt; newYears = selectedYears.stream()
    .map(year -&gt; new RoomYear(room, year))
    .toList();
roomYearRepository.saveAll(newYears); // 새로운 연도 삽입</code></pre>
<p><strong>여기서 문제가 발생했다.</strong></p>
<p>JPA는 트랜잭션 안에서 delete 쿼리를 바로 DB에 반영하지 않고,
<strong>플러시 전까지는 영속성 컨텍스트 안에만 기록해 둔다.</strong></p>
<p>따라서, deleteAll() 직후 insert가 발생할 경우,
<strong>기존 연도 데이터가 아직 DB에 존재한 상태에서 insert가 시도되어</strong>
유니크 키 제약 조건 위반이 발생하게 된다.</p>
<hr />
<h2 id="해결-방법">해결 방법</h2>
<p>✅ flush() 호출</p>
<pre><code class="language-java">roomYearRepository.deleteAllByRoom(room);
roomYearRepository.flush(); // 삭제 즉시 DB 반영

List&lt;RoomYear&gt; newYears = selectedYears.stream()
    .distinct()
    .map(year -&gt; new RoomYear(room, year))
    .toList();

roomYearRepository.saveAll(newYears);</code></pre>
<ul>
<li><code>flush()</code>는 영속성 컨텍스트에 쌓인 변경사항을 DB에 즉시 반영한다.</li>
<li>이를 통해 <code>insert</code>가 실행되기 전에 <code>delete</code>가 DB에서 먼저 처리되도록 보장할 수 있다.</li>
</ul>
<hr />
<h2 id="다른-시도들-실패-경험">다른 시도들 (실패 경험)</h2>
<h3 id="❌-양방향-연관관계--orphanremoval">❌ 양방향 연관관계 + orphanRemoval</h3>
<pre><code class="language-java">@OneToMany(mappedBy = &quot;room&quot;, cascade = CascadeType.ALL, orphanRemoval = true)
private List&lt;RoomYear&gt; roomYears = new ArrayList&lt;&gt;();</code></pre>
<ul>
<li><code>roomYears.clear()</code> 이후 <code>addAll()</code> 방식도 시도했지만, 내부적으로는 역시 delete + insert 구조라 <strong>flush가 없기에 동일한 문제가 발생</strong>하였다.
또한, 도메인 객체 내부에서 연도 생성/제거를 다루는 것은 <strong>SRP 관점에서도 분리하는 것이 좋다고 생각한다.</strong></li>
</ul>
<hr />
<h2 id="결론">결론</h2>
<ul>
<li><code>JPA</code>에서 <code>delete</code> 후 <code>insert</code>를 연이어 수행할 경우,
<code>flush()</code> 없이 진행하면 <strong>기존 데이터가 삭제되기 전에 insert가 발생할 수 있다.</strong></li>
<li>유니크 키가 걸린 컬럼이라면 <strong>Duplicate Entry 에러가 발생</strong>한다.</li>
<li>해결책은 간단하다:
👉<code>delete 후 flush()</code> → <code>insert</code> <strong>순서</strong>를 명확히 보장하자!</li>
</ul>
<hr />
<h2 id="참고자료">참고자료</h2>
<ul>
<li><a href="https://blog.naver.com/rorean/221536561667">JPA / remove 후 바로 insert 하고 싶은데 duplicate entry violation 에러가 날 때</a></li>
<li><a href="https://keylog.tistory.com/entry/JPA-deleteAll-%EC%88%98%ED%96%89-%ED%9B%84-%EB%B0%94%EB%A1%9C-insert-%ED%96%88%EC%9D%84-%EB%95%8C-duplicate-entry-%EC%97%90%EB%9F%AC%EA%B0%80-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EB%AC%B8%EC%A0%9C">JPA / deleteAll 수행 후 바로 Insert 했을 때 duplicate entry 문제가 발생하는 문제</a></li>
</ul>