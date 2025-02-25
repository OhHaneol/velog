<h2 id="📍-코드">📍 코드</h2>
<pre><code class="language-java">    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "hashtag_id")
    private Set&lt;Hashtag&gt; hashtags = new HashSet&lt;&gt;();</code></pre>
<h2 id="📍-error">📍 Error</h2>
<h3 id="메시지">메시지</h3>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/b84f357e-986e-4d63-a9c9-d9e68d7c2faf/image.png" /></p>
<blockquote>
<p>org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]: Association 'ohahsis.dailydirecter.note.domain.NoteHashtag.hashtags' <strong>targets an unknown entity named 'java.util.Set'</strong></p>
</blockquote>
<h3 id="분석">분석</h3>
<ul>
<li>조인하는 컬럼의 이름은 <code>hashtag_id</code>인데, 이에 해당하면서 <code>java.util.Set</code>에도 해당하는 엔티티의 이름을 찾을 수 없다고 한다.</li>
</ul>
<h2 id="📍-해결">📍 해결</h2>
<ul>
<li>데이터 타입이 Set&lt;Hashtag&gt; 가 아니라 매핑한 테이블인 Hashtag 여야 한다.</li>
<li>이에 맞춰 비즈니스 로직 등을 수정했다.</li>
</ul>
<pre><code class="language-java">    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "hashtag_id")
    private Hashtag hashtag;</code></pre>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://hulrud.tistory.com/m/24">[SpringBoot] AnnotationException: targets an unknown entity named : 조인 맵핑 시 데이터 타입이 다를 경우 발생하는 에러</a></li>
</ul>