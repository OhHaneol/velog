<h2 id="📍-단위-테스트unit-test와-통합-테스트integration-test-비교">📍 단위 테스트(Unit Test)와 통합 테스트(Integration Test) 비교</h2>
<table>
<thead>
<tr>
<th><strong>항목</strong></th>
<th><strong>단위 테스트 (Unit Test)</strong></th>
<th><strong>통합 테스트 (Integration Test)</strong></th>
</tr>
</thead>
<tbody><tr>
<td><strong>테스트 대상</strong></td>
<td>개별 클래스나 메서드</td>
<td>여러 구성 요소(컨트롤러, 서비스, DB 등)의 상호작용</td>
</tr>
<tr>
<td><strong>목적</strong></td>
<td>외부 시스템과 독립적으로 로직의 정확성 검증</td>
<td>시스템 구성 요소들이 실제 환경에서 정상적으로 동작하는지 검증</td>
</tr>
<tr>
<td><strong>주요 특성</strong></td>
<td>- 외부 의존성을 mocking하여 독립적인 테스트 진행- 빠른 실행</td>
<td>- 여러 컴포넌트와 실제 환경을 염두에 둔 테스트 진행- 실행 시간이 길어짐</td>
</tr>
<tr>
<td><strong>외부 의존성</strong></td>
<td>mocking을 사용하여 외부 의존성(예: DB, 서비스)을 시뮬레이션</td>
<td>실제 데이터베이스, 외부 시스템과의 연결을 사용하여 테스트</td>
</tr>
<tr>
<td><strong>실행 환경</strong></td>
<td>별도의 환경 설정 없이 테스트할 수 있음</td>
<td>실제 환경 설정을 로드하고 동작하는지 확인</td>
</tr>
<tr>
<td><strong>테스트 시간</strong></td>
<td>빠름</td>
<td>상대적으로 길어짐</td>
</tr>
</tbody></table>
<hr />
<h2 id="📍-주요-테스트-어노테이션">📍 주요 테스트 어노테이션</h2>
<h3 id="1-webmvctest">1. @WebMvcTest</h3>
<ul>
<li><p><strong>목적</strong>: 컨트롤러 관련 테스트</p>
</li>
<li><p><strong>설명</strong>: <code>@WebMvcTest</code>는 컨트롤러의 테스트에 사용되며, Spring MVC 관련 설정만 로드하고, 서비스나 레포지토리와 같은 다른 빈들은 로드하지 않는다. 따라서 주로 컨트롤러와 관련된 기능만 테스트할 때 사용한다.</p>
</li>
<li><p><strong>사용 예시</strong>:</p>
<pre><code class="language-java">  java
  복사편집
  @WebMvcTest(HealthController.class)
  public class HealthControllerTest {
      // 테스트 코드
  }
</code></pre>
</li>
<li><p><strong>특징</strong>:</p>
<ul>
<li>Spring MVC 관련 컴포넌트만 로드</li>
<li><code>@MockBean</code>을 사용하여 서비스나 레포지토리 의존성 주입 가능</li>
<li>컨트롤러의 요청과 응답을 확인하는 데 유용</li>
<li>테스트 환경이 비교적 가볍고 빠르다.</li>
</ul>
</li>
</ul>
<h3 id="2-springboottest">2. @SpringBootTest</h3>
<ul>
<li><p><strong>목적</strong>: 통합 테스트</p>
</li>
<li><p><strong>설명</strong>: <code>@SpringBootTest</code>는 애플리케이션 전체를 로드하고, 실제 애플리케이션 컨텍스트를 사용하여 시스템이 정상적으로 동작하는지 테스트한다. DB나 외부 API와 통합된 테스트를 진행할 때 주로 사용된다.</p>
</li>
<li><p><strong>사용 예시</strong>:</p>
<pre><code class="language-java">  java
  복사편집
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  public class IntegrationTest {
      // 테스트 코드
  }
</code></pre>
</li>
<li><p><strong>특징</strong>:</p>
<ul>
<li>전체 애플리케이션을 로드하고 실행</li>
<li>실제 데이터베이스나 서버를 사용하여 시스템 통합 테스트</li>
<li>테스트 환경이 무겁고 시간이 오래 걸린다.</li>
</ul>
</li>
</ul>
<h3 id="3-mockbean">3. @MockBean</h3>
<ul>
<li><p><strong>목적</strong>: 외부 의존성을 mocking 하여 서비스나 레포지토리와 같은 외부 의존성의 동작을 시뮬레이션</p>
</li>
<li><p><strong>설명</strong>: <code>@MockBean</code>은 <code>@WebMvcTest</code>나 <code>@SpringBootTest</code>와 함께 사용하여 외부 의존성을 mocking하는 데 사용된다. 이를 통해 실제 구현체 대신 mock 객체를 사용하여 테스트를 진행할 수 있다.</p>
</li>
<li><p><strong>사용 예시</strong>:</p>
<pre><code class="language-java">  java
  복사편집
  @MockBean
  private HealthService healthService;
</code></pre>
</li>
<li><p><strong>특징</strong>:</p>
<ul>
<li>실제 구현체를 로드하지 않고 mock 객체를 주입</li>
</ul>
</li>
</ul>
<h3 id="4-extendwithmockitoextensionclass">4. @ExtendWith(MockitoExtension.class)</h3>
<ul>
<li><p><strong>목적</strong>: Mockito 기능을 사용한 테스트</p>
</li>
<li><p><strong>설명</strong>: <code>@ExtendWith(MockitoExtension.class)</code>는 Mockito를 사용하여 mock 객체를 생성하고, <code>@Mock</code>, <code>@InjectMocks</code>와 같은 Mockito 어노테이션을 사용할 수 있도록 설정하는 어노테이션이다.</p>
</li>
<li><p><strong>사용 예시</strong>:</p>
<pre><code class="language-java">  java
  복사편집
  @ExtendWith(MockitoExtension.class)
  public class MockitoTest {
      @Mock
      private HealthService healthService;

      @InjectMocks
      private HealthController healthController;

      // 테스트 코드
  }
</code></pre>
</li>
<li><p><strong>특징</strong>:</p>
<ul>
<li>Mockito를 사용한 mocking 및 의존성 주입</li>
<li><code>@Mock</code>, <code>@InjectMocks</code>와 함께 사용하여 외부 의존성의 동작을 시뮬레이션</li>
</ul>
</li>
</ul>
<h3 id="5-datajpatest">5. @DataJpaTest</h3>
<ul>
<li><p><strong>목적</strong>: 데이터베이스 관련 테스트</p>
</li>
<li><p><strong>설명</strong>: <code>@DataJpaTest</code>는 JPA 관련 컴포넌트만 로드하여 데이터베이스와 관련된 레포지토리 테스트를 진행하는 데 사용된다. 서비스나 컨트롤러는 로드되지 않는다.</p>
</li>
<li><p><strong>사용 예시</strong>:</p>
<pre><code class="language-java">  java
  복사편집
  @DataJpaTest
  public class UserRepositoryTest {
      // 테스트 코드
  }
</code></pre>
</li>
<li><p><strong>특징</strong>:</p>
<ul>
<li>JPA 관련 빈만 로드되고 DB와 관련된 테스트에 유용</li>
<li>DB와 상호작용하는 코드의 테스트가 필요할 때 사용</li>
</ul>
</li>
</ul>
<hr />
<h2 id="📍-테스트-방법-정리">📍 테스트 방법 정리</h2>
<table>
<thead>
<tr>
<th><strong>테스트 종류</strong></th>
<th><strong>설명</strong></th>
<th><strong>주요 어노테이션</strong></th>
<th><strong>특징</strong></th>
</tr>
</thead>
<tbody><tr>
<td><strong>통합 테스트</strong></td>
<td>- 여러 컴포넌트 간의 상호작용을 실제 환경처럼 테스트- 실제 DB나 외부 시스템과의 연결을 사용</td>
<td><code>@SpringBootTest</code>, <code>@DataJpaTest</code></td>
<td>- 실행 시간이 길어짐- 실제 환경에서 테스트</td>
</tr>
<tr>
<td><strong>단위 테스트</strong></td>
<td>- 개별 메서드나 클래스의 로직을 테스트 - 외부 의존성 mocking을 사용하여 독립적으로 진행</td>
<td><code>@WebMvcTest</code>, <code>@ExtendWith(MockitoExtension.class)</code></td>
<td>- 빠른 실행- DB나 외부 시스템과의 연결 없이 테스트</td>
</tr>
<tr>
<td><strong>컨트롤러 테스트</strong></td>
<td>- 컨트롤러가 올바르게 동작하는지 테스트</td>
<td><code>@WebMvcTest</code></td>
<td>- HTTP 요청과 응답 검증- 서비스나 레포지토리는 mocking</td>
</tr>
<tr>
<td><strong>비즈니스 로직 테스트</strong></td>
<td>- 서비스나 비즈니스 로직이 의도대로 동작하는지 테스트</td>
<td><code>@ExtendWith(MockitoExtension.class)</code></td>
<td>- 외부 의존성 mocking- 비즈니스 로직 검증</td>
</tr>
<tr>
<td><strong>JPA 관련 테스트</strong></td>
<td>- 데이터베이스와 관련된 레포지토리 테스트</td>
<td><code>@DataJpaTest</code></td>
<td>- JPA 관련 빈만 로드- DB와의 상호작용 테스트</td>
</tr>
</tbody></table>
<hr />
<h2 id="📍-결론">📍 결론</h2>
<ul>
<li><strong>단위 테스트</strong>는 독립적으로 로직을 검증하는 데 유용하며, 주로 <code>@WebMvcTest</code>나 <code>@ExtendWith(MockitoExtension.class)</code>를 사용하여 외부 의존성 없이 비즈니스 로직을 테스트한다.</li>
<li><strong>통합 테스트</strong>는 여러 구성 요소 간의 상호작용을 실제 환경처럼 테스트하며, <code>@SpringBootTest</code>나 <code>@DataJpaTest</code>를 사용하여 실제 DB나 외부 시스템과의 통합을 검증한다.</li>
<li>컨트롤러, 서비스, DB 등 각 계층별로 적절한 어노테이션을 사용하여 테스트를 진행한다.</li>
</ul>