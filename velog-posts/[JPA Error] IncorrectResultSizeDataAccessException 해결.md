<h2 id="📍-코드">📍 코드</h2>
<ul>
<li>해시태그 필드, 관련 비즈니스 로직 중 일부를 발췌했다.</li>
</ul>
<h3 id="hashtag">Hashtag</h3>
<pre><code class="language-java">private String name;</code></pre>
<h3 id="noteservice">NoteService</h3>
<pre><code class="language-java">hashtag = hashtagRepository.findByName(name);</code></pre>
<h2 id="📍-error">📍 Error</h2>
<h3 id="메시지">메시지</h3>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/b2a3a15e-359f-4fc1-b992-ea8f53f1bdce/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/d023877d-44bf-4b7d-80c7-551e733ee3eb/image.png" /></p>
<blockquote>
<ul>
<li>Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: org.springframework.dao.<strong>IncorrectResultSizeDataAccessException</strong>: Query did not return a unique result: 7 results were returned] with root cause</li>
</ul>
</blockquote>
<ul>
<li>org.hibernate.<strong>NonUniqueResultException</strong>: Query did not return a unique result: 7 results were returned</li>
</ul>
<h3 id="분석">분석</h3>
<ul>
<li>JPA 레포지토리에서 findBy~~ 메서드를 통해 단일 select를 하면서 발생한 예외이다.</li>
<li>원하는 조건의 데이터는 단일 데이터인데, db에 해당 조건의 데이터가 여러개가 존재하며 모든 데이터가 반환되어 예외가 발생했다.</li>
<li>초반에 해시태그를 생성하는 비즈니스 로직에서 존재 여부 검증이 누락되며 데이터가 쌓인 것 같다. MySQL 테이블을 조회해보니 실제로 중복 데이터가 쌓여 있는 것을 확인할 수 있었다.</li>
</ul>
<h2 id="📍-해결">📍 해결</h2>
<ul>
<li>단순히 아래 명령어로 <code>hashtag</code> 테이블 내의 레코드들을 삭제하는 방법을 생각해봤다.<pre><code class="language-sql">  truncate table hashtag;</code></pre>
</li>
<li>하지만 <code>note_hashtag</code> 테이블에 의해 외래 키로 제약이 걸려 있었고, 함부로 데이터를 삭제할 수 없다고 한다.</li>
<li>극단적으로 해당 DB 전체를 삭제하는 것이 그 다음으로 떠오른 방법이었다:D (이는 실제 사용하는 데이터가 없기 때문에 가능한 일이다.)</li>
<li>생각한 방법들은 총 다음과 같다.<ul>
<li>전체 DB 삭제</li>
<li>FK 를 연관된 데이터 sql 쿼리로 직접 삭제
 <a href="https://happylulurara.tistory.com/127">https://happylulurara.tistory.com/127</a>
<a href="https://blog.naver.com/simple7540/90096172063">https://blog.naver.com/simple7540/90096172063</a></li>
</ul>
</li>
<li>우선은 <strong>db 를 삭제</strong>하고 다음에 기타 방법을 알아보도록 하자.</li>
<li>삭제 후에는 같은 문제를 방지하기 위해 해당 컬럼에 unique 설정을 해주었다.</li>
</ul>
<h3 id="hashtag-1">Hashtag</h3>
<pre><code class="language-java">    @Column(name = "name", unique = true)
    private String name;</code></pre>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://ldh-6019.tistory.com/251">[JPA] UNIQUE 설정</a></li>
</ul>