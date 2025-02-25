<h2 id="📍-코드">📍 코드</h2>
<ul>
<li><code>note</code>와 <code>hashtag</code>, <code>note_hashtag</code>를 모두 저장하는 로직을 구현 중에 오류가 발생했다.</li>
<li><code>note_hashtag</code> 가 저장되는 시점에, 미리 만들어진(build 된) <code>note</code>를 함께 저장해야 했다.</li>
<li>아래 코드와 같이 set 키워드로 <code>note</code>에 <code>note_hashtag</code>를 지정하는 방식을 택했다.</li>
</ul>
<h3 id="service">Service</h3>
<pre><code class="language-java">// 기타 request 요소들에 대해 build된 note

note.setNoteHashtags(savedNoteHashtags);</code></pre>
<h2 id="📍-error">📍 Error</h2>
<h3 id="메시지">메시지</h3>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/50c79e4d-6aa5-4aaa-9c31-bda1b3945d68/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/70e75aa1-a479-4dc2-9214-f6078e7e63eb/image.png" /></p>
<blockquote>
<ul>
<li>Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Handler dispatch failed: java.lang.<strong>StackOverflowError</strong>] with root cause</li>
</ul>
</blockquote>
<ul>
<li><strong>hashCode</strong></li>
</ul>
<h3 id="분석">분석</h3>
<ul>
<li>문제가 되는 클래스의 라인들은, 모두 노트와 노트_해시태그 각각의 <code>@Data</code> 어노테이션 중에서도 <code>@EqualsAndHashcode</code> 부분이었음. (앞에서 설정한 set 으로 인해?) 무한순환참조 오류가 생긴 것으로 추정된다.</li>
</ul>
<h2 id="📍-해결">📍 해결</h2>
<ul>
<li>일단 <code>note</code>는 그대로 두고, <code>note_hashtag</code> 엔터티의 <code>@Data</code> 대신 <code>@Getter</code> 로 바꿨다.</li>
<li>이후 정상적으로 작동하였다.</li>
<li>원인의 이유를 원론적으로 요약하자면, 엔터티의 경우 이미 <code>PK</code>를 이용해서 동일성을 검증하기 충분하기 때문에 <code>@EqualsAndHashCode</code>를 사용할 이유가 크게 없다. 괜히 사용했다 이렇게 오류에 빠지기 십상이다.</li>
<li>내 프로젝트 내에서 이 이유를 적용했을 때, <strong><em>왜 <code>note</code>는 <code>@Data</code>를 그대로 뒀는데도 정상 작동하는지 아직 이해하지 못한 것 같다. <del>문제가 되는 것 중 하나만 제거하면 되는 걸까..?</del></em></strong></li>
<li>다만 왜 하이버네이트를 이용할 때 롬복 애너테이션을 활용할 경우 <code>@EqualsAndHashCode</code>를 피하라고 하는지 알 것 같다.</li>
<li>아니 잘 모르겠다. 추가로 공부를 진행하자🥺</li>
</ul>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://murphymoon.tistory.com/entry/JPA-repository-%EC%82%AC%EC%9A%A9%EC%8B%9C-%EB%AC%B4%ED%95%9C-%EC%88%9C%ED%99%98%EC%97%90-%EB%B9%A0%EC%A7%80%EB%8A%94-%EB%AC%B8%EC%A0%9CEquals-and-HashCode-lombok">JPA repository 사용시 무한 순환에 빠지는 문제(Equals and HashCode) [lombok]</a></li>
</ul>