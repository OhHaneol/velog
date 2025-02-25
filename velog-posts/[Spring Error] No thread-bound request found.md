<h2 id="📍-문제-상황">📍 문제 상황</h2>
<ul>
<li>AI 답변을 비동기 처리하면서, 관련 메서드에 어노테이션 기반 처리율 제한을 걸었다.</li>
<li>이 때 다음과 같은 오류가 발생했다.<blockquote>
<p>No thread-bound request found: Are you referring to request attributes outside of an actual web request, or processing a request outside of the originally receiving thread? If you are actually operating within a web request and still receive this message, your code is probably running outside of DispatcherServlet: In this case, use RequestContextListener or RequestContextFilter to expose the current request.</p>
</blockquote>
</li>
</ul>
<h2 id="📍-분석">📍 분석</h2>
<p>Spring MVC 에서는 웹 요청이 들어오면 각 요청마다 고유한 "요청 컨텍스트"를 생성한다.</p>
<blockquote>
<p><strong>요청 컨텍스트(Context)란?</strong>
하나의 HTTP 요청과 관련된 정보(Request, Response 객체 등)를 담고 있는 객체이다.</p>
</blockquote>
<blockquote>
<p><strong>요청 컨텍스트와 스레드의 관계</strong></p>
</blockquote>
<ul>
<li>요청 컨텍스트를 스레드에 바인딩해서 사용한다.</li>
<li>하나의 스레드가 여러 요청을 순차적으로 처리할 수 있다.</li>
<li>하나의 요청이 여러 스레드에 걸쳐 처리될 수도 있다.(이 경우 컨텍스트 전파가 필요하다.)</li>
</ul>
<p>다음 <code>rateLimit</code> 메서드에서 <code>HttpServletResponse</code>를 얻기 위해 <code>RequestContextHolder</code>를 사용하고 있는데, <strong>RequestContextHolder는 이 요청 컨텍스트를 ThreadLocal 에 저장해서 같은 스레드 내라면 어디서든 현재 요청 정보를 가져올 수 있게 해준다.</strong></p>
<p>그러나 비동기 처리 시(CompletableFuture.runAsync()를 사용)에는 새로운 스레드에서 작업이 실행되고, 이 새로운 스레드는 원래 웹 요청을 처리하던 스레드와 다르다.</p>
<p>따라서 <code>response.setHeader()</code> 를 호출하려고 할 때 <strong>원래 요청의 컨텍스트 정보를 가지고 있지 않아 response 객체를 가져올 수 없어 발생하는 문제</strong>였다.</p>
<pre><code class="language-java">    @Around("@annotation(rateLimit)")
    public Object rateLimit(ProceedingJoinPoint joinPoint, RateLimit rateLimit) throws Throwable {
        // APIRateLimiter 객체를 사용하여 API 키에 해당하는 버킷에서 토큰을 소비하려고 시도한다.
        // 토큰이 충분하면 요청이 성공적으로 처리되고, 그렇지 않으면 예외가 발생한다.
        String key = evaluateKey(joinPoint, rateLimit.key());
        long remainingTokens = apiRateLimiter.tryConsume(key, rateLimit.limit(),
                rateLimit.period());

        if (remainingTokens &gt;= 0) {
            Object result = joinPoint.proceed();
            log.debug("Method execution result: {}", result);

            // 현재 실행 컨텍스트에서 HttpServletResponse 얻기
            HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getResponse();
            if (response != null) {
                response.setHeader("X-RateLimit-Remaining", String.valueOf(remainingTokens));
                log.debug("Added X-RateLimit-Remaining header: {}", remainingTokens);
            } else {
                log.warn("Unable to set X-RateLimit-Remaining header: HttpServletResponse is null");
            }

            return result;
        } else {
            throw rateLimit.exceptionClass().getDeclaredConstructor(String.class)
                    .newInstance("Rate limit exceeded for key: " + key);
        }
    }</code></pre>
<h2 id="📍-해결">📍 해결</h2>
<p>기존에 헤더로 남은 횟수를 돌려주었는데, 현재 해당 기능 전용 API 를 만든 상태이기 때문에 코드 삭제를 통해 해결하였다.
만약 기능 유지를 원한다면 컨텍스트 전파를 이용할 수 있을 것 같다.</p>