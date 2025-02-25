<h1 id="📍-개발-순서">📍 개발 순서</h1>
<ul>
<li>글쓰기 model과 dto 작성</li>
<li>글쓰기 Repository 구현</li>
<li>글쓰기 Service 구현</li>
<li>글쓰기 Controller 구현</li>
</ul>
<h1 id="📍-개발">📍 개발</h1>
<h2 id="글쓰기-model과-dto-작성">글쓰기 model과 dto 작성</h2>
<h3 id="note-domain-작성">Note domain 작성</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/domain/Note.java</code></p>
<pre><code class="language-java">@Data
@Entity
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "note")
public class Note extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long note_id;

    private String title;

    private Boolean status; // 노트 완결 여부

    @Size(max = 4)
    private Set&lt;String&gt; contents = new HashSet&lt;&gt;();

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "id")
    private User user;

    @OneToMany(mappedBy = "note", cascade = CascadeType.ALL)
    private Set&lt;NoteHashtag&gt; noteHashtags = new HashSet&lt;&gt;();
}
</code></pre>
<ul>
<li><code>Note</code> 와 <code>InnerNote(현 contents)</code> 에 대한 고민<ul>
<li>InnerNote 를 테이블을 분리할까 고민했는데, 내부 리스트 컬럼으로 둬도 될 것 같다.</li>
<li>해시태그와 달리 외부에서 InnerNote 만 조회할 일은 없을 것 같고, 괜히 연산만 복잡할 것 같았다.</li>
</ul>
</li>
</ul>
<h3 id="hashtag-domain-작성">Hashtag domain 작성</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/domain/Hashtag.java</code></p>
<pre><code class="language-java">@Data
@Entity
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "hashtag")
public class Hashtag {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long hashtag_id;

    private String name;

    @OneToMany(mappedBy = "hashtag", cascade = CascadeType.ALL)
    private Set&lt;NoteHashtag&gt; noteHashtags = new HashSet&lt;&gt;();

}
</code></pre>
<ul>
<li>hashtag 를 note 에 컬럼으로 추가할 수도 있었지만, 추후 커뮤니티버전에서 확장성을 위해(태그당 조회수 조회) 테이블로 분리했다.</li>
</ul>
<h3 id="notehashtag-domain-작성">NoteHashtag domain 작성</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/domain/NoteHashtag.java</code></p>
<pre><code class="language-java">@Data
@Entity
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "note_hashtag")
public class NoteHashtag {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long note_hashtag_id;

    @ManyToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "note_id")
    private Note note;

    @ManyToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "hashtag_id")
    private Hashtag hashtag;

}</code></pre>
<ul>
<li><p>개발을 진행하던 중 JPA 에 대한 공부가 부족해서 <code>cascade</code> 설정 관련 오류가 발생했다. 자세한 내용은 다음 문서를 참고하자.</p>
<ul>
<li><a href="https://velog.io/@otyvs1109/Error-Cannot-invoke-org.hibernate.mapping.ToOne.getReferencedEntityName-because-toOne-is-null-%ED%95%B4%EA%B2%B0">[JPA Error] Cannot invoke "org.hibernate.mapping.ToOne.getReferencedEntityName()" because "toOne" is null 해결</a></li>
<li><a href="https://velog.io/@otyvs1109/JPA-Error-object-references-an-unsaved-transient-instance-save-the-transient-instance-before-flushing">[JPA Error] object references an unsaved transient instance - save the transient instance before flushing</a></li>
</ul>
</li>
<li><p>마찬가지로 Hashtag 테이블과 맵핑할 때, NoteHashtag:Hashtag = N:1 관계라는 생각에 Hashtag 를 Set&lt;&gt; 형식으로 맵핑하여 오류가 발생했다. <strong>근데 그게 아니다!!</strong></p>
</li>
<li><p><code>NoteHashtag</code> 는 데이터 테이블로써 독립적으로 존재하고, <code>Hashtag</code>의 속성을 여러개 가지는 게 아니라 종합적인 정보를 담은 튜플을 여러 개 가지는 형식이 된다.</p>
</li>
<li><p>정확히 말하자면 <strong>데이터 타입은 맵핑한 테이블인 Hashtag여야 한다.</strong></p>
<ul>
<li><a href="https://velog.io/@otyvs1109/Error-targets-an-unknown-entity-named-java.util.Set">[JPA Error] targets an unknown entity named 'java.util.Set' 해결</a></li>
</ul>
</li>
</ul>
<h3 id="notesaverequest-dto-작성">NoteSaveRequest dto 작성</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/dto/request/NoteSaveRequest.java</code></p>
<pre><code class="language-java">@Data
public class NoteSaveRequest {

//    @NotBlank(message = "내용을 입력해주세요.")
    private Set&lt;String&gt; contents;

    private Boolean status;

    private String title;

    private Set&lt;String&gt; hashtagNames;
}
</code></pre>
<h3 id="notesaveresponse-dto-작성">NoteSaveResponse dto 작성</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/dto/response/NoteSaveResponse.java</code></p>
<pre><code class="language-java">@Data
@AllArgsConstructor
public class NoteSaveResponse {
    private Long note_id;
}</code></pre>
<h2 id="글쓰기-repository-구현">글쓰기 Repository 구현</h2>
<h3 id="noterepository-구현">NoteRepository 구현</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/infrastructure/NoteRepository.java</code></p>
<pre><code class="language-java">@Repository
@Transactional(readOnly = true)
public interface NoteRepository extends JpaRepository&lt;Note, Long&gt; {
}</code></pre>
<h3 id="hashtagrepository-구현">HashtagRepository 구현</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/infrastructure/HashtagRepository.java</code></p>
<pre><code class="language-java">@Repository
@Transactional(readOnly = true)
public interface HashtagRepository extends JpaRepository&lt;Hashtag, Long&gt; {
    @Transactional
    Boolean existsByName(String name);
}</code></pre>
<h3 id="notehashtagrepository-구현">NoteHashtagRepository 구현</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/infrastructure/NoteHashtagRepository.java</code></p>
<pre><code class="language-java">@Repository
@Transactional(readOnly = true)
public interface NoteHashtagRepository extends JpaRepository&lt;NoteHashtag, Long&gt; {
}
</code></pre>
<ul>
<li><code>NoteHashtag</code> 또한 명확하게 분리된 하나의 기능이기 때문에 별도의 리포지토리를 생성한다.</li>
</ul>
<h2 id="글쓰기-service-구현">글쓰기 Service 구현</h2>
<h3 id="노트-내용-최대-개수-작성">노트 내용 최대 개수 작성</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/NoteConstants.java</code></p>
<pre><code class="language-java">public class NoteConstants {
    public static final int CONTENTS_MAX_SIZE = 4;
}</code></pre>
<h3 id="noteservice-구현">NoteService 구현</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/application/NoteService.java</code></p>
<pre><code class="language-java">@Slf4j
@Service
@RequiredArgsConstructor
public class NoteService {
    private final NoteRepository noteRepository;
    private final NoteHashtagRepository noteHashtagRepository;
    private final HashtagRepository hashtagRepository;

    @Transactional
    public NoteSaveResponse save(NoteSaveRequest request) {
        // 제목과 내용이 모두 없는 경우
        if (request.getContents().isEmpty() || request.getTitle().isBlank()) {
            throw new NoteSaveInvalidException(ErrorType.NOT_BLANK_ERROR);
        }

        // 기승전결 외 5개 이상의 문서 저장 요청이 온 경우
        if (request.getContents().size() &gt; CONTENTS_MAX_SIZE) {
            throw new NoteSaveInvalidException(ErrorType.CONTENTS_MAX_SIZE_4);
        }


        /**
         * 노트 저장
         */
        var note = Note.builder()
                .contents(request.getContents())
                .status(request.getStatus())
                .title(request.getTitle())
                .build();

        var savedNote = noteRepository.save(note);


        /**
         * 해시태그 저장
         */
        for (String name : request.getHashtagNames()) {
            if(!hashtagRepository.existsByName(name)) {
                var hashtag = Hashtag.builder()
                        .name(name)
                        .build();
                hashtagRepository.save(hashtag);
            }
        }

        /**
         * 노트 해시태그 저장
         */
        for (String name : request.getHashtagNames()) {
            Hashtag hashtag = Hashtag.builder()
                    .name(name)
                    .build();
            var noteHashtag = NoteHashtag.builder()
                    .note(note)
                    .hashtag(hashtag)
                    .build();
            noteHashtagRepository.save(noteHashtag);
        }

        return new NoteSaveResponse(savedNote.getNote_id());
    }
}
</code></pre>
<blockquote>
<p>추후 해시태그 서비스를 분리하는 것을 고려해보자.</p>
</blockquote>
<h2 id="글쓰기-controller-구현">글쓰기 Controller 구현</h2>
<h3 id="notecontroller-구현-및-사용자-인가-적용">NoteController 구현 및 사용자 인가 적용</h3>
<p><code>src/main/java/ohahsis/dailydirector/note/presentation/NoteController.java</code></p>
<pre><code class="language-java">@RestController
@RequestMapping(path = "/api/notes", produces = MediaType.APPLICATION_JSON_VALUE)
@RequiredArgsConstructor
public class NoteController {
    private final NoteService noteService;

    @PostMapping
    public ResponseEntity&lt;?&gt; createNote(AuthUser user, @RequestBody NoteSaveRequest noteRequest) {
        var response = noteService.save(noteRequest);
        return ResponseDto.created(response);
    }
}</code></pre>
<ul>
<li>AuthUser 를 파라미터에 포함시켜서 인가 resolver 가 작동하도록 한다.</li>
</ul>
<h1 id="📍-결과">📍 결과</h1>
<h3 id="postman-헤더-토큰-설정">POSTMAN 헤더 토큰 설정</h3>
<img alt="Auth 설정" src="https://velog.velcdn.com/images/otyvs1109/post/7b6440c4-1c78-4ba9-a87d-55995a1f39eb/image.png" width="600" />

<img alt="Header 확인" src="https://velog.velcdn.com/images/otyvs1109/post/827fcd9b-05d2-4bed-8ff0-124addf58209/image.png" width="600" />

<ul>
<li>api 요청을 보내기 전, 토큰 상수의 값을 key 로, 로그인 했을 때 응답으로 받은 토큰 값을 value 로 하는 키-값을 헤더에 포함시킨다.</li>
<li>Header 를 보면 설정한 API Key 가 잘 적용된 것을 확인할 수 있다.</li>
</ul>
<h3 id="노트-정상-추가">노트 정상 추가</h3>
<img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/5362699a-ab24-462c-980c-1c2a2c3c4dea/image.png" width="600" />

<h3 id="노트-4개-초과-예외">노트 4개 초과 예외</h3>
<img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/9994a25a-aa38-4712-a383-81a0efe2f3a9/image.png" width="600" />

<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://targetcoders.com/jjwt-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95/">JWT 생성 및 검증하기 (jjwt 사용 방법)</a></li>
<li><a href="https://velog.io/@shawnhansh/Postman-Authorization%EC%97%90-%ED%86%A0%ED%81%B0-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0">Postman Authorization에 토큰 추가하기</a></li>
<li><a href="https://samtao.tistory.com/65">Java에서 JJWT(Java JSON Web Token)를 이용한 JWT(JSON Web Token) 사용방법</a></li>
<li>bearer 제거 등 : <a href="https://velog.io/@haerong22/Argument-Resolver-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-Authorizationwith-JWT">Argument Resolver 를 이용한 Authorization(with JWT)</a></li>
<li>토큰에서 값 추출 등 : <a href="https://jj-yi.tistory.com/21">Spring으로 Token 받기</a></li>
</ul>