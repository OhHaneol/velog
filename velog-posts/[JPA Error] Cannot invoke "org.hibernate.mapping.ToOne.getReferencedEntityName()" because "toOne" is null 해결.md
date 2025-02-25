<h2 id="📍-코드">📍 코드</h2>
<ul>
<li>글을 올릴 <code>Note</code>, 해시태그를 저장하는 <code>Hashtag</code>, 그리고 중간 테이블인 <code>NoteHashtag</code> 가 있다.</li>
</ul>
<h3 id="note">Note</h3>
<pre><code class="language-java">    @OneToMany(mappedBy = "note")
    private Set&lt;Hashtag&gt; hashtags = new HashSet&lt;&gt;();</code></pre>
<h3 id="hashtag">Hashtag</h3>
<pre><code class="language-java">    @OneToMany(mappedBy = "hashtags")
    private Set&lt;Note&gt; notes = new HashSet&lt;&gt;();</code></pre>
<h3 id="notehashtag">NoteHashtag</h3>
<pre><code class="language-java">    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "note_id")
    private Note note;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "hashtag_id")
    private Hashtag hashtag;</code></pre>
<h2 id="📍-error">📍 Error</h2>
<h3 id="메시지">메시지</h3>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/3f6a39fc-5741-451f-a0ce-c9f8d58aa7ff/image.png" /></p>
<blockquote>
<p>org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]: Cannot invoke "org.hibernate.mapping.ToOne.getReferencedEntityName()" <strong>because "toOne" is null</strong></p>
</blockquote>
<h3 id="분석">분석</h3>
<ul>
<li>ToOne 에 해당하는 Note 가 null 이 될 수 있다는 말인 것 같다.</li>
<li><code>NoteHashtag</code> 는 <code>Note</code> 에 종속적인 개념이기때문에 해당 필드로 찾아가서 <code>cascade = CascadeType.ALL</code> 옵션을 걸어주면 된다는 결론을 내리고 Note 와 Hashtag 각각에 찾아가보니 FK 참조(?)까지 잘못 되어 있었다!</li>
<li>N:M 관계인 테이블 Note 와 Hashtag 를 이어줄 중간 테이블 NoteHashtag 를 만들어놓고, FK 는 똑같이 Note 와 Hashtag 서로를 참조하고 있었다.</li>
</ul>
<h2 id="📍-해결">📍 해결</h2>
<ul>
<li><code>cascade = CascadeType.ALL</code> 옵션을 걸어주었다.</li>
<li>NoteHashtag 에 있는 FK 를 똑바로 참조하도록 바꾸고, <code>mappedBy</code> 의 값 또한 해당 필드의 이름으로 바꿔주었다.</li>
<li>정상 동작!</li>
</ul>
<h3 id="note-1">Note</h3>
<pre><code class="language-java">    @OneToMany(mappedBy = "note", cascade = CascadeType.ALL)
    private Set&lt;NoteHashtag&gt; noteHashtags = new HashSet&lt;&gt;();</code></pre>
<h3 id="hashtag-1">Hashtag</h3>
<pre><code class="language-java">    @OneToMany(mappedBy = "hashtag", cascade = CascadeType.ALL)
    private Set&lt;NoteHashtag&gt; noteHashtags = new HashSet&lt;&gt;();</code></pre>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://tecoble.techcourse.co.kr/post/2023-08-14-JPA-Cascade/">JPA Cascade는 무엇이고 언제 사용해야 할까?</a></li>
</ul>