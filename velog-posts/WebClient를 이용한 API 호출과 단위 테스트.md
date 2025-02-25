<h2 id="📍-webclient를-이용한-외부-api-호출에-대한-테스트">📍 WebClient를 이용한 외부 API 호출에 대한 테스트</h2>
<p><code>WebClient</code>를 사용한 외부 API 호출에 대한 테스트는 일반적으로 세 가지 주요 접근 방법을 사용한다.</p>
<ol>
<li><strong>실제 외부 API 호출 (통합 테스트)</strong><ul>
<li>실제 외부 API를 호출하고 응답을 검증하는 방식으로, 실 환경에서의 동작을 검증할 수 있다.</li>
<li>하지만 외부 API 호출은 네트워크 상태에 따라 테스트가 불안정할 수 있고, 비용이 발생할 수 있어 단위 테스트에서는 일반적으로 피한다.</li>
</ul>
</li>
<li><strong>Mocking WebClient (단위 테스트)</strong><ul>
<li>외부 API 호출을 <code>WebClient</code>를 통해 모킹(mocking)하여 실제 API 호출을 하지 않고 응답을 시뮬레이션한다.</li>
<li>이 방법을 사용하면 테스트가 빠르고, 외부 API 상태에 의존하지 않으며, 실패가 외부 API와 무관하게 발생하지 않는다.</li>
</ul>
</li>
<li><strong>MockWebServer를 이용한 테스트 (Mock 서버 테스트)</strong><ul>
<li><code>MockWebServer</code>를 사용하여 실제 API 서버처럼 동작하는 가짜 서버를 구성하고, <code>WebClient</code>가 호출할 URL을 해당 서버로 설정하여 외부 API 호출을 모킹한다.</li>
<li>이 방법은 외부 API 서버의 응답을 시뮬레이션하고, <code>WebClient</code>의 동작을 보다 세밀하게 테스트할 수 있다.</li>
<li>여기서는 해당 방법을 이용한 예시를 설명한다.</li>
</ul>
</li>
</ol>
<hr />
<h2 id="📍-mockwebserver를-이용한-테스트-작성">📍 MockWebServer를 이용한 테스트 작성</h2>
<p><code>MockWebServer</code>를 사용하여 외부 API의 응답을 모킹하고 <code>WebClient</code>를 테스트하는 과정은 다음과 같다.</p>
<h3 id="1-buildgradle-의존성-추가">1. build.gradle 의존성 추가</h3>
<pre><code>    testImplementation 'com.squareup.okhttp3:mockwebserver:4.9.3'</code></pre><h3 id="2-mockwebserver-설정">2. MockWebServer 설정</h3>
<p>그 다음 <code>MockWebServer</code>를 설정하여 가짜 API 서버를 시작한다. 서버의 URL을 <code>WebClient</code>의 base URL로 설정하여, <code>WebClient</code>가 실제로 해당 서버에 요청을 보내도록 한다.</p>
<pre><code class="language-java">@BeforeAll
static void setUpServer() throws IOException {
    mockWebServer = new MockWebServer();
    mockWebServer.start();
    objectMapper = new ObjectMapper();
}

@AfterAll
static void tearDownServer() throws IOException {
    mockWebServer.shutdown();
}</code></pre>
<h3 id="3-webclient-설정">3. WebClient 설정</h3>
<p>테스트 메서드에서 <code>WebClient</code>를 설정하고, <code>MockWebServer</code>의 URL을 base URL로 설정하여 외부 API 호출을 시뮬레이션한다.</p>
<pre><code class="language-java">@BeforeEach
void setUp() {
    WebClient webClient = WebClient.builder().baseUrl(mockWebServer.url("/").toString()).build();
    initService = new InitService(userRepository, roomRepository, webClient);
}</code></pre>
<h3 id="4-mock-응답-설정">4. Mock 응답 설정</h3>
<p>각 테스트에서 <code>MockWebServer</code>에 특정 응답을 설정하고, <code>WebClient</code>가 해당 응답을 받도록 한다. 응답은 JSON 형식으로 설정하여, 실제 외부 API 응답을 흉내 낸다.</p>
<pre><code class="language-java">@Test
void Valid_api_response_is_returned() throws Exception {
    // Given: Mock 응답 생성
    FakerApiResponse mockResponse = new FakerApiResponse();
    mockResponse.setData(List.of(new FakerUser(1, "John Doe", "john@example.com")));

    mockWebServer.enqueue(new MockResponse()
            .setBody(objectMapper.writeValueAsString(mockResponse))
            .addHeader("Content-Type", "application/json"));

    InitRequest request = new InitRequest(123, 1);

    // When
    FakerApiResponse response = initService.getFakerApiResponse(request);

    // Then
    assertNotNull(response);
    assertEquals(1, response.getData().size());
    assertEquals("John Doe", response.getData().get(0).getUsername());
}</code></pre>
<h3 id="5-테스트-실행">5. 테스트 실행</h3>
<p>테스트 실행 시, <code>WebClient</code>는 <code>MockWebServer</code>에서 제공하는 응답을 받게 되며, 이를 기반으로 테스트를 진행한다. 외부 API가 아닌 가짜 서버를 사용하므로, 실제 네트워크 상황이나 외부 API 상태에 영향을 받지 않는다.</p>
<hr />
<h2 id="📍-결론">📍 결론</h2>
<p>이 방식은 외부 API와의 연동을 실제로 테스트할 필요 없이, API 호출의 결과와 동작을 안정적으로 테스트할 수 있는 강력한 방법이다. <code>MockWebServer</code>를 사용하면 외부 의존성을 완전히 차단하고, 특정 조건에 맞는 API 응답을 쉽게 테스트할 수 있다.</p>