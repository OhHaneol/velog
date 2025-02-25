<h2 id="📍-문제-상황">📍 문제 상황</h2>
<ul>
<li><p>도서 정보를 print 하는 메서드를 다음과 같이 같은 이름, 다른 파라미터로 오버로딩 하였다.</p>
<pre><code class="language-java">  private static void printBookInfo(Book&lt;String&gt; book) {}
  private static void printBookInfo(Book&lt;Integer&gt; book) {}</code></pre>
</li>
<li><p>다음과 같이 오류가 발생했다.</p>
<blockquote>
<p>both methods have same <strong>erasure</strong></p>
</blockquote>
</li>
</ul>
<h2 id="📍-분석">📍 분석</h2>
<ul>
<li><p>자바의 제너릭은 컴파일과 런타임 시 다른 코드로 읽히도록 JVM이 관리한다.</p>
</li>
<li><p>컴파일을 할 때 제너릭은 타입의 제약 조건을 사용하도록 정의하고, 런타임에 들어가면 소거를 하게 된다. 즉, <strong>컴파일을 하는 시점</strong>에서 제너릭에 명시한 Object의 <strong>타입이 유효</strong>하지만, <strong>런타임 시</strong>에는 제너릭에 명시한 Object의 <strong>타입이 소거</strong>되는 것을 말한다.</p>
</li>
<li><p>즉, Book 이라는 Object 타입은 명시되지만, Book 의 제너릭 타입인 String 과 Integer 는 소거된다. 따라서 위에서 문제가 된 두 메서드는 다음과 같이, 사실 완전히 동일한 메서드로 인식되어 오버로딩 되지 않는 것이다.</p>
<pre><code class="language-java">  private static void printBookInfo(Book book) {}
  private static void printBookInfo(Book book) {}</code></pre>
</li>
</ul>
<h2 id="📍-해결">📍 해결</h2>
<ul>
<li>따라서 이러한 타입 소거에 따른 오버로딩 문제를, 메서드의 (기능과)이름을 달리 정의하여 해결하였다.</li>
</ul>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://velog.io/@power0080/%EC%9E%90%EB%B0%94-Overloading-a-method-both-methods-have-same-erasure-%EC%A0%95%EB%A6%AC">자바 Overloading a method: both methods have same erasure 정리</a></li>
</ul>