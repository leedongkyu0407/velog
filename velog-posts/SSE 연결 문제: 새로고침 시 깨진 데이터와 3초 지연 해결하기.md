<p>최근 프로젝트에서 방 목록을 실시간으로 업데이트하기 위해 SSE(Server-Sent Events)를 사용했습니다. 저희 서비스에서는 메인 화면에서 사용자가 채널을 선택하면 채널로 입장해서 게임방을 보고 해당 방에 입장이 가능했습니다. 이 부분에서 게임 방의 정보들을 실시간으로 보여주기 위해 SSE를 사용했는데 처음 채널에 입장할 때는 정상적으로 동작했으나, 사용자가 페이지를 새로고침할 때마다 이상한 문제가 발생했습니다. 바로 <strong>깨진 데이터가 나타나고 약 3초 정도 지나서야 정상적인 데이터가 내려오는 것</strong>이었습니다.
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/0476aeb1-f020-4f8a-b287-7b123201c831/image.png" /></p>
<ul>
<li>3초 뒤에 정상적으로 연결
<img alt="" src="https://velog.velcdn.com/images/donggyu47/post/63a610e2-c5c1-4b2d-a117-02bb93792759/image.png" /></li>
</ul>
<p>더욱 혼란스러웠던 것은 이 문제가 프론트엔드와 백엔드 <strong>로컬 개발 환경에서는 전혀 나타나지 않았다는 점</strong>입니다. 로컬에서는 정상적으로 작동하고 있었기 때문에 더더욱 원인을 찾기 어려웠습니다.</p>
<hr />
<h3 id="😵-처음-겪은-문제">😵 처음 겪은 문제</h3>
<p>브라우저에서 확인한 응답은 이상한 깨진 문자였습니다. 그래서 가장 먼저 떠올린 건 인코딩 문제였습니다. 서버 응답의 인코딩을 명시적으로 지정해보았습니다.</p>
<pre><code class="language-java">response.setCharacterEncoding(&quot;UTF-8&quot;);
response.setContentType(&quot;text/event-stream;charset=UTF-8&quot;);</code></pre>
<p>하지만 이 방법은 아무 효과가 없었습니다.</p>
<hr />
<h3 id="🤔-nginx-프록시-버퍼링">🤔 Nginx 프록시 버퍼링?</h3>
<p>다음으로 의심한 것은 Nginx 프록시 버퍼링 문제였습니다. 검색을 통해 SSE는 스트리밍 방식이라 Nginx에서 버퍼링을 꺼야 한다고 되어 있는 글을 보고 답을 찾았다고 생각했었습니다. 하지만, 설정을 점검해 보니 이미 <code>proxy_buffering off</code> 설정이 잘 되어 있었습니다.</p>
<pre><code class="language-nginx">location /sse {
    proxy_buffering off;
}</code></pre>
<p>결국, 이 또한 원인이 아니었습니다.</p>
<hr />
<h3 id="❤️-heartbeat-주기-줄이기-시도">❤️ Heartbeat 주기 줄이기 시도</h3>
<p>다음으로 SSE 연결의 heartbeat 주기를 줄이는 방법을 고려했습니다. 기본적으로 서버에서 10초마다 클라이언트에게 heartbeat 메시지를 보내고 있었는데, 이 주기를 1초로 줄여서 서버가 더 빨리 연결 종료를 감지할 수 있게 하는 방법을 시도해보려 했습니다.</p>
<pre><code class="language-java">@Scheduled(fixedRate = 10000) // 10초 간격
public void sendHeartbeat() {
    emitter.send(SseEmitter.event().name(&quot;heartbeat&quot;).data(&quot;&quot;));
}</code></pre>
<p>하지만 금방 의문이 들었습니다. &quot;10초마다 heartbeat라고 해도 1초에 heartbeat 받고 새로고침하면 똑같이 로딩 걸리는 거 아닌가?&quot; 만약 heartbeat를 받은 직후 새로고침을 한다면, 다음 heartbeat가 올 때까지 서버는 여전히 연결 종료를 알 수 없을 것입니다. 따라서 이 방법도 근본적인 해결책이 될 수 없다고 판단했습니다.</p>
<hr />
<h3 id="⏱️-tomcat-timeout-줄이기">⏱️ Tomcat timeout 줄이기</h3>
<p>그 후, 답을 알 수 없어 ChatGPT에게 물어보니 서버가 클라이언트의 연결 종료를 빨리 알아채도록 Tomcat의 Connection Timeout 값을 줄이는 방법을 제안했습니다.</p>
<pre><code class="language-properties">server.tomcat.connection-timeout=2000</code></pre>
<p>하지만 이렇게 설정하면 서비스 전체에 영향을 줄 수 있고 일반적인 HTTP 요청에서도, 클라이언트 요청 처리 과정에서 2초 내에 요청이 없으면 연결을 닫아버려서 오히려 HTTP Keep-Alive의 이점을 잃어 성능이 저하될 수 있기 때문에 근본적인 해결책이라고 보기는 어려웠습니다.</p>
<hr />
<h3 id="💡-문제의-원인을-깨닫게-된-과정">💡 문제의 원인을 깨닫게 된 과정</h3>
<p>서버 로그를 계속 관찰하며 클라이언트가 새로고침으로 연결을 종료했음에도 불구하고 서버는 이를 즉시 감지하지 못하고 계속 데이터를 보내려고 시도하고 있음을 발견했습니다. 약 3초가 지나서야 서버가 연결 종료를 인지하고 있었습니다.</p>
<p>이 순간, 문제는 서버의 설정이나 인코딩이 아니라, 클라이언트가 서버에게 연결 종료를 명확히 알리지 않았기 때문이라고 추측했습니다. 서버는 클라이언트의 상태를 즉시 파악하지 못하기 때문에, 클라이언트가 직접 종료를 알려주는 방식이 필요했습니다.</p>
<hr />
<h3 id="🎯-최종-해결법-클라이언트에서-명시적-disconnect-보내기">🎯 최종 해결법: 클라이언트에서 명시적 disconnect 보내기</h3>
<p>결국 클라이언트가 페이지를 닫거나 새로고침할 때 서버에게 직접 연결 종료를 알려주는 방식으로 문제를 깔끔하게 해결했습니다.
문제의 원인을 파악한 후, 프론트엔드 팀원에게 양해를 구하고 제가 직접 프론트 코드를 수정했습니다. 백엔드 개발자로서 프론트엔드 코드를 수정하는 것이 일반적인 상황은 아니었지만, 문제의 원인을 이해하고 빠르게 해결하기 위한 결정이었습니다.</p>
<p><strong>클라이언트 코드</strong></p>
<pre><code class="language-javascript">useEffect(() =&gt; {
  const handleUnload = () =&gt; {
    navigator.sendBeacon(`${API_URL}/sse/disconnect`);
  };

  window.addEventListener('beforeunload', handleUnload);

  return () =&gt; {
    window.removeEventListener('beforeunload', handleUnload);
  };
}, []);</code></pre>
<p><strong>서버 코드</strong></p>
<pre><code class="language-java">@PostMapping(&quot;/sse/disconnect&quot;)
public ResponseEntity&lt;Void&gt; disconnect(HttpServletRequest request) {
    String sessionId = request.getSession().getId();
    emitterManager.cleanupChannelSession(sessionId);
    return ResponseEntity.ok().build();
}</code></pre>
<p>이제는 페이지를 새로고침해도 깨진 데이터나 지연 없이 즉시 정상적으로 데이터를 받을 수 있게 되었습니다.</p>
<p>이 코드가 하는 일은 브라우저가 페이지를 떠날 때(새로고침이나 페이지 닫기) HTTP 요청을 서버로 보내는 것입니다. 여기서 핵심은 <code>navigator.sendBeacon</code> API입니다.
일반적인 HTTP 요청(axios, fetch)과 <code>navigator.sendBeacon</code>의 차이를 이해해봅시다:</p>
<p><strong>일반 HTTP 요청의 한계</strong></p>
<ul>
<li>브라우저가 페이지를 떠날 때 진행 중인 axios나 fetch 요청들은 대부분 취소됩니다.</li>
<li>따라서 서버 측에서 클라이언트의 연결이 갑자기 끊어진 것처럼 보이는데, 서버는 이 상황을 즉시 감지하지 못합니다.</li>
<li>결과적으로 서버는 이미 존재하지 않는 클라이언트에게 계속해서 데이터를 보내려고 시도합니다.</li>
</ul>
<p><strong>sendBeacon의 장점</strong></p>
<ul>
<li>이 API는 페이지 종료 과정에서도 요청이 백그라운드에서 확실히 전송되도록 합니다.</li>
<li>비동기적으로 작동하며 페이지 전환을 지연시키지 않습니다.</li>
<li>HTTP POST 메서드로 요청을 보내고, 서버 응답을 기다리지 않습니다.</li>
<li>마치 서버에 <strong>&quot;나 이제 연결 끊을게, 내 리소스 정리해도 돼&quot;</strong>라고 명시적으로 알려주는 역할을 합니다.</li>
</ul>
<p>서버 입장에서 보면, 클라이언트가 페이지를 새로고침할 때 기존 SSE 연결은 TCP 레벨에서 즉시 끊어지지만, 애플리케이션 레벨에서 서버는 이 사실을 바로 알 수 없습니다. <code>navigator.sendBeacon</code>을 통해 보내는 명시적인 신호가 있어야 서버가 빠르게 리소스를 정리하고 새로운 연결을 위한 준비를 할 수 있습니다.
이는 마치 데이터베이스 트랜잭션에서 명시적으로 commit이나 rollback을 호출하는 것과 비슷한 개념입니다. 클라이언트가 <strong>&quot;연결 종료&quot;를 서버에 알려줌</strong>으로써 애플리케이션 레벨의 리소스를 즉시 정리할 수 있게 되는 것입니다.</p>
<hr />
<h3 id="📌-결론--배운-점">📌 결론 &amp; 배운 점</h3>
<ul>
<li>SSE는 클라이언트의 연결 종료를 명시적으로 처리하는 게 가장 확실합니다.</li>
<li>로컬 환경에서는 문제없던 코드가 실제 배포 환경에서만 문제가 발생하는 경우가 있으므로, 환경 차이를 항상 염두에 두고 테스트하는 것이 중요합니다.</li>
<li>heartbeat 주기를 줄이는 것은 일시적인 해결책일 수 있지만, 모든 상황에서 효과적이지 않을 수 있습니다.</li>
<li>문제를 해결할 때는 증상을 관찰하고 정확한 원인을 분석하는 과정이 중요하다는 것을 다시 한번 깨달았습니다.</li>
</ul>
<p>이 글이 같은 문제로 고민하는 분들에게 도움이 되었으면 좋겠습니다! 😊</p>