<h2 id="📍-코드">📍 코드</h2>
<ul>
<li>특정 키워드를 포함한 게시글을 검색하는 api 를 구현 중에 문제가 생겼다.</li>
</ul>
<h3 id="repository">Repository</h3>
<pre><code class="language-java">@Repository
@Transactional(readOnly = true)
public interface NoteRepository extends JpaRepository&lt;Note, Long&gt; {

    List&lt;Note&gt; findByUser_IdAndTitleContaining(Long userId, String keyword);
    List&lt;Note&gt; findByUser_IdAndContentsContaining(Long userId, String keyword);

}</code></pre>
<h3 id=""></h3>
<h2 id="📍-error">📍 Error</h2>
<h3 id="메시지">메시지</h3>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/ab061bc3-10b4-4b32-84d5-59fb11a08d7d/image.png" /></p>
<blockquote>
<p>Error creating bean with name 'searchService' defined in file [...]: Unsatisfied dependency expressed through constructor parameter 0: Error creating bean with name 'noteRepository' defined in NoteRepository defined in @EnableJpaRepositories declared on JpaRepositoriesRegistrar.EnableJpaRepositoriesConfiguration: <strong>Operand of 'member of' operator must be a plural path</strong></p>
</blockquote>
<h3 id="분석">분석</h3>
<ul>
<li>plural 은 '복수의' 라는 뜻이다.</li>
<li>jpa 메서드 조건으로 넣은 contents 가 <code>note</code> 엔티티에서 문자열 List로 선언되어 있는데, 그게 문제였던 것이다.</li>
</ul>
<h2 id="📍-해결">📍 해결</h2>
<ul>
<li>컬렉션 객체를 엔티티에 필드로 선언할 때는, <code>@ElementCollection</code> 을 필드에 붙여서 jpa 에 알려야 한다.</li>
</ul>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://ttl-blog.tistory.com/121">[JPA] 필드와 컬럼 매핑 - @ElementCollection (값 타입 컬렉션 매핑), @CollectionTable
출처: https://ttl-blog.tistory.com/121 [Shin._.Mallang:티스토리]</a></li>
</ul>