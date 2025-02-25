<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/92b11f87-a6ce-48e7-b089-b71f8ae16cfb/image.png" /></p>
<h2 id="ip-주소를-캐싱-및-스케줄링">IP 주소를 캐싱 및 스케줄링</h2>
<p>게시글 조회수를 정확하게 집계하기 위한 다음 두 가지 핵심 컴포넌트를 구성하였다.</p>
<ul>
<li>방문자 정보를 캐싱하는 인터셉터 (QuestionVisitInterceptor)</li>
<li>캐싱된 정보를 DB에 저장하는 스케줄러 (QuestionVisitScheduler)</li>
</ul>
<p>해당 작업으로 얻을 수 있는 기대효과는 다음과 같다.</p>
<ul>
<li>날짜별 방문자 구분과 IP 기반 중복 방문 필터링, User-Agent 정보를 통한 접근 패턴 분석을 통해 정확한 조회수 집계가 가능하다.</li>
<li>Redis를 통한 빠른 중복 체크와 배치 처리를 통한 DB 부하 감소, 비동기 처리로 사용자 응답 시간을 최소화하여 시스템 성능의 최적화가 가능하다.</li>
<li>기간별 조회수 통계와 방문자 패턴 분석, 접근 디바이스 정보 수집을 통해 추후 데이터 분석 지원을 기대할 수도 있다.</li>
</ul>
<h3 id="questionvisitinterceptor">QuestionVisitInterceptor</h3>
<p>동일 사용자의 중복 조회를 방지하고 성능을 최적화하는 컴포넌트로, 동작 순서는 다음과 같다.</p>
<ol>
<li>/api/questions/{id} 경로로 요청이 들어오면 인터셉터가 작동</li>
<li>방문자의 IP 주소, 게시글 ID, 날짜 정보를 조합하여 Redis 키 생성</li>
<li>Redis에 해당 키가 없는 경우에만 방문 정보 저장</li>
</ol>
<p>키의 구조는 아래의 형식을 따른다.</p>
<p>question:{questionId}:visit:{visitorIp}:{date}</p>
<p>IP 주소의 경우 <code>IPv4</code>는 그대로 사용하고(예: 127.0.0.1), <code>IPv6</code>의 경우 Redis의 구분자와 달리하기 위해 ':'를 '_'로 변환하여 저장한다.(예: 0:0:0:0:0:0:0:1 → 0_0_0_0_0_0_0_1)</p>
<pre><code class="language-java">@Component
@RequiredArgsConstructor
public class QuestionVisitInterceptor implements HandlerInterceptor {

    private final RedisTemplate&lt;String, String&gt; redisTemplate;
    private final QuestionRepository questionRepository;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws Exception {

        String questionId = extractQuestionId(request);
        if (questionId == null) {
            return true;
        } else if (!questionRepository.existsById(Long.parseLong(questionId))) {
            throw new QuestionInvalidException(ErrorType.QUESTION_NOT_FOUND_ERROR);
        }

        String visitorIdentifier = request.getRemoteAddr();

        // IPv6 주소 처리
        if (visitorIdentifier.contains(":")) {
            visitorIdentifier = visitorIdentifier.replace(":", "_");
        }

        String today = LocalDate.now().toString();

        String key = "question:" + questionId + ":visit:" + visitorIdentifier + ":" + today;

        String userAgent = request.getHeader("User-Agent");

        ValueOperations&lt;String, String&gt; valueOperations = redisTemplate.opsForValue();

        // 새로운 접근자일 경우 Redis 에 방문 정보 저장
        if (!valueOperations.getOperations().hasKey(key)) {
            valueOperations.set(key, userAgent);
        }

        return true;
    }

    private String extractQuestionId(HttpServletRequest request) {
        String path = request.getRequestURI();  // HTTP 요청의 URI 경로를 가져온다.
        Pattern pattern = Pattern.compile("/api/questions/(\\d+)"); // (\\d+): 하나 이상의 숫자를 캡처 그룹으로 지정한다.
        Matcher matcher = pattern.matcher(path);    // 주어진 path에 대해 패턴 매칭을 수행할 Matcher 객체를 생성한다.
        return matcher.find() ? matcher.group(1) : null;    // find() 는 패턴과 일치하는 부분을 찾고, 일치하는 부분이 있으면 group(1)로 첫 번째 캡처 그룹(숫자 부분)을 반환한다.
    }
}</code></pre>
<h3 id="questionvisitscheduler">QuestionVisitScheduler</h3>
<p>캐싱된 방문 정보를 영구 저장소에 기록하기 위한 컴포넌트로, 1초에 한 번씩 다음과 같은 처리를 진행한다.</p>
<ol>
<li><p>Redis에서 모든 방문 기록 키를 조회한다.</p>
</li>
<li><p>각 키에 대해 다음과 같은 처리를 한다.</p>
<ul>
<li>방문자 정보 파싱</li>
<li>중복 방문 체크</li>
<li>QuestionView 테이블에 방문 기록 저장</li>
<li>게시글의 전체 조회수 증가</li>
<li>처리 완료된 Redis 키 삭제</li>
</ul>
</li>
</ol>
<p><code>QuestionView</code> 테이블에 저장된 정보는 특정 기간의 조회수 산출에 사용하고, <code>Question</code> 테이블에 updateViewCnt()를 통해 증가시킨 게시글 조회수는 게시글 별 전체 조회수 산출에 사용</p>
<pre><code class="language-java">@Slf4j
@Component
@RequiredArgsConstructor
public class QuestionVisitScheduler extends AbstractValidateService {

    private final RedisTemplate&lt;String, String&gt; redisTemplate;
    private final QuestionViewRepository questionViewRepository;

    @Transactional
    @Scheduled(initialDelay = 1000, fixedDelay = 1000)  // 1초 지연, 1초 주기
    public void updateQuestionVisitorData() {
        Set&lt;String&gt; keys = redisTemplate.keys("question:*:visit:*:*");

        for (String key : keys) {
            try {
                String[] parts = key.split(":");
                if (parts.length != 5) {
                    log.warn("게시글 조회 Key 형식이 잘못되었습니다: {}", key);
                    continue;
                }

                long questionId = Long.parseLong(parts[1]);
                String visitorIdentifier = parts[3];
                String dateStr = parts[4];

                LocalDate date;
                try {
                    date = LocalDate.parse(dateStr);
                } catch (DateTimeParseException e) {
                    log.error("Failed to parse date '{}' from key: {}", dateStr, key, e);
                    continue;
                }

                ValueOperations&lt;String, String&gt; valueOperations = redisTemplate.opsForValue();
                String userAgent = valueOperations.get(key);

                Question findQuestion = getQuestionOrThrowIfNotExist(questionId);
                Community findCommunity = getCommunityOrThrowIfNotExist(findQuestion.getCommId());

                if (!questionViewRepository.existsByVisitorIdentifierAndViewedAtAndQuestionId(
                        visitorIdentifier, date, questionId)
                ) {
                    questionViewRepository.save(
                            QuestionView.builder()
                                    .userAgent(userAgent)
                                    .viewedAt(LocalDate.now())
                                    .questionId(questionId)
                                    .visitorIdentifier(visitorIdentifier)
                                    .commId(findCommunity.getId())
                                    .build());
                    findQuestion.updateViewCnt();

                    log.info("게시글 조회: questionId={}, visitorIdentifier={}, date={}", questionId,
                            visitorIdentifier, date);
                }

                redisTemplate.delete(key);
                log.debug("Processed and deleted key: {}", key);

            } catch (Exception e) {
                log.error("Error processing key: {}", key, e);
            }
        }

    }
}
</code></pre>