<h2 id="해결">해결</h2>
<ul>
<li>FK 를 참조하는 쪽 뿐만 아니라, FK 를 갖고 있는 쪽(연관관계의 주인)까지 양방향으로 <code>cascate = CascadeType=All</code> 옵션을 설정해야 한다.</li>
</ul>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://bcp0109.tistory.com/344">JPA 관련 Hibernate 에러: object references an unsaved transient instance - save the transient instance before flushing</a></li>
</ul>