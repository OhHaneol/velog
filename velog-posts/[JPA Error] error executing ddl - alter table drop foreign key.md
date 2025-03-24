<h2 id="📍문제-상황">📍문제 상황</h2>
<blockquote>
<p>error executing ddl alter table drop foreign key</p>
</blockquote>
<p>스프링 어플리케이션 실행 시 위와 같은 오류 메시지가 나왔다.
당시 <code>application.yml</code> 파일은 다음과 같다.</p>
<pre><code class="language-yml">spring:
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate.format_sql: true
    defer-datasource-initialization: true</code></pre>
<p><code>ddl-auto</code>의 옵션이 <strong>create-drop</strong>이었다.</p>
<h2 id="📍분석">📍분석</h2>
<p><code>spring.jpa.hibernate.ddl-auto</code>옵션의 경우 JAVA의 Entity설정을 참고하여 Spring Application실행시점에 Hibernate에서 자동으로 DDL을 생성해 필요한 DATABASE의 Table설정들을 자동으로 수행해준다. 이 때 <strong>create-drop</strong>의 경우 기존 테이블을 삭제 후 다시 생성하되, 종료시점에 테이블을 DROP한다.</p>
<p>해당 오류는 실행 시 모든 table을 drop시켜 alter할 table을 찾지못해 나타나는 에러였다. </p>
<h2 id="📍해결">📍해결</h2>
<p>create-drop 옵션은 애플리케이션 시작 시 스키마를 새로 만들고, 종료할 때 모두 삭제한다. 이 경우 개발 중 테스트 데이터나 수동으로 입력한 데이터가 매번 사라져 디버깅이나 반복 테스트 시 불편함이 있다.
반면 <strong>update</strong> 옵션은 기존 테이블을 유지하면서 변경된 엔티티에 맞게 스키마를 수정하기 때문에 데이터가 삭제되지 않아서, <strong>개발 초기 단계에서는 보통 데이터의 지속성과 스키마 변경의 편리성을 고려하여 create 또는 update를 권장한다.</strong></p>