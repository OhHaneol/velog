<h2 id="exceptionhandler">@ExceptionHandler</h2>
<ul>
<li><code>@ExceptionHandler</code>는 <strong>Controller 계층</strong>에서 발생하는 에러를 잡아서 메서드로 처리해주는 기능으로, 매우 유연하게 에러를 처리할 수 있는 방법을 제공한다.</li>
<li>Service, Repository에서 발생하는 에러는 제외한다.</li>
<li><code>@ExceptionHandler</code>는 다음에 어노테이션을 추가함으로써 에러를 손쉽게 처리할 수 있다.<ul>
<li>컨트롤러의 메소드</li>
<li><code>@ControllerAdvice</code>나 <code>@RestControllerAdvice</code>가 있는 클래스의 메소드</li>
</ul>
</li>
</ul>
<h3 id="컨트롤러-메소드에-exceptionhandler-추가">컨트롤러 메소드에 @ExceptionHandler 추가</h3>
<pre><code class="language-java">@RestController
@RequiredArgsConstructor
public class ProductController {

  private final ProductService productService;

  @GetMapping("/product/{id}")
  public Response getProduct(@PathVariable String id){
    return productService.getProduct(id);
  }

  @ExceptionHandler(NoSuchElementFoundException.class)
  public ResponseEntity&lt;String&gt; handleNoSuchElementFoundException(NoSuchElementFoundException exception) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(exception.getMessage());
  }
}</code></pre>
<p><strong>코드 설명</strong></p>
<ul>
<li><code>@Controller</code>로 선언된 클래스 안에서 <code>@ExceptionHandler</code> 어노테이션으로 메서드 안에서 발생할 수 있는 에러를 처리할 수 있다.</li>
<li><code>@ExceptionHandler</code>에 의해 발생한 예외는 <code>ExceptionHandlerExceptionResolver</code>에 의해 처리가 된다.</li>
<li><code>@ExceptionHandler</code>는 Exception 클래스들을 속성으로 받아 처리할 예외를 지정할 수 있다. 만약 <code>@ExceptionHandler</code>에 예외 클래스를 지정하지 않는다면, 파라미터에 설정된 에러 클래스를 처리하게 된다.</li>
</ul>
<p><strong>장점</strong></p>
<ul>
<li><code>ExceptionHandler</code>는 <code>@ResponseStatus</code>와 달리 <strong>에러 응답(payload)을 자유롭게 다룰 수 있다</strong>는 점에서 유연하다. 예를 들어 응답을 다음과 같이 정의해서 내려준다면 좋을 것이다.<ul>
<li><code>code</code>: 어떠한 종류의 에러가 발생하는지에 대한 에러 코드</li>
<li><code>message</code>: 왜 에러가 발생했는지에 대한 설명</li>
<li><code>errors</code>: 어느 값이 잘못되어 @Valid에 의한 검증이 실패한 것인지를 위한 에러 목록</li>
</ul>
</li>
</ul>
<blockquote>
<p><strong>파라미터와 반환 타입</strong>
ExceptionHandler의 파라미터로 HttpServletRequest나 WebRequest 등을 얻을 수 있으며 반환 타입으로는 ResponseEntity, String, void 등 자유롭게 활용할 수 있다.
더 자세한 내용은 아래 공식 문서에서 확인 가능하다.</p>
</blockquote>
<ul>
<li>파라미터: <a href="https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html#mvc-ann-exceptionhandler-args">스프링 공식 문서-Method Arguments</a></li>
<li>반환 타입: <a href="https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html#mvc-ann-exceptionhandler-return-values">스프링 공식 문서-Return Values</a></li>
</ul>
<p><strong>주의할 점</strong></p>
<ul>
<li><code>@ExceptionHandler</code>를 사용 시에 주의할 점은 <strong><code>@ExceptionHandler</code>에 등록된 예외 클래스</strong>와 <strong>파라미터로 받는 예외 클래스</strong>가 <strong>동일</strong>해야 한다는 것이다. 만약 값이 다르다면 스프링은 컴파일 시점에 에러를 내지 않다가 런타임 시점에 다음과 같은 에러를 발생시킨다.<pre><code class="language-shell">java.lang.IllegalStateException: No suitable resolver for argument [0] [type=...]
HandlerMethod details: ...</code></pre>
</li>
</ul>
<p><code>@ExceptionHandler</code>는 컨트롤러에 구현하므로 특정 컨트롤러에서만 발생하는 예외만 처리된다. 하지만 컨트롤러에 에러 처리 코드가 섞이며, 에러 처리 코드가 중복될 가능성이 높다. 그래서 스프링은 전역적으로 예외를 처리할 수 있는 좋은 기술을 제공해준다.</p>
<hr />
<h2 id="controlleradvice">@ControllerAdvice</h2>
<p>ControllerAdvice는 여러 컨트롤러에 대해 전역적으로 ExceptionHandler를 적용해준다. 위에서 보이듯 ControllerAdvice 어노테이션에는 @Component 어노테이션이 있어서 ControllerAdvice가 선언된 클래스는 스프링 빈으로 등록된다.
그러므로 다음과 같이 전역적으로 에러를 핸들링하는 클래스를 만들어 어노테이션을 붙여주면 에러 처리를 위임할 수 있다.</p>
<pre><code class="language-java">@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NoSuchElementFoundException.class)
    protected ResponseEntity&lt;?&gt; handleNoSuchElementFoundException(NoSuchElementFoundException e) {
        final ErrorResponse errorResponse = ErrorResponse.builder()
                .code("Item Not Found")
                .message(e.getMessage()).build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
}</code></pre>
<ul>
<li><code>@ControllerAdvice</code>는 <code>@Controller</code>와 <code>handler</code>에서 발생하는 에러들을 모두 잡아준다.</li>
<li><code>@ControllerAdvice</code>안에서 <code>@ExceptionHandler</code>를 사용하여 에러를 잡을 수 있다.</li>
</ul>
<h3 id="controlleradvice-이점">@ControllerAdvice 이점</h3>
<p><code>@ControllerAdvice</code>를 이용하면 다음과 같은 이점을 누릴 수 있다.</p>
<ul>
<li>하나의 클래스로 모든 컨트롤러에 대해 전역적으로 예외 처리가 가능하다.</li>
<li>직접 정의한 에러 응답을 일관성있게 클라이언트에게 내려줄 수 있다.</li>
<li>별도의 try-catch문이 없어 코드의 가독성이 높아진다.</li>
</ul>
<h3 id="controlleradvice-사용시-주의점">@ControllerAdvice 사용시 주의점</h3>
<p>일관된 예외 응답을 위해서는 다음의 사항을 주의해야 한다.</p>
<ul>
<li>한 프로젝트당 하나의 ControllerAdvice만 관리하는 것이 좋다.</li>
<li>만약 여러 ControllerAdvice가 필요하다면 basePackages나 annotations 등을 지정해야 한다.</li>
<li>직접 구현한 Exception 클래스들은 한 공간에서 관리한다.</li>
</ul>
<h1 id="📍-스프링의-예외-처리-흐름">📍 스프링의 예외 처리 흐름</h1>
<p>다음과 같은 예외 처리기들은 스프링의 빈으로 등록되어 있고, 예외가 발생하면 순차적으로 다음의 Resolver들이 처리가능한지 판별한 후에 예외가 처리된다.</p>
<img src="https://velog.velcdn.com/images/otyvs1109/post/50b8e282-eea4-4e53-8ca4-f024ff4de34c/image.png" width="700" />

<ol>
<li><p><code>ExceptionHandlerExceptionResolver</code> 동작</p>
<ul>
<li>예외가 발생한 컨트롤러 안에 적합한 <code>@ExceptionHandler</code>가 있는지 검사한다.</li>
<li>컨트롤러의 <code>@ExceptionHandler</code>에서 처리가능하다면 처리하고, 그렇지 않으면 <code>ControllerAdvice</code>로 넘어간다.</li>
<li><code>ControllerAdvice</code>안에 적합한 <code>@ExceptionHandler</code>가 있는지 검사하고 없으면 다음 처리기로 넘어간다.</li>
</ul>
</li>
<li><p><code>ResponseStatusExceptionResolver</code> 동작</p>
<ul>
<li><code>@ResponseStatus</code>가 있는지 또는 <code>ResponseStatusException</code>인지 검사한다.</li>
<li>맞으면 <code>ServletResponse</code>의 <code>sendError()</code>로 예외를 서블릿까지 전달되고, 서블릿이 <code>BasicErrorController</code>로 요청을 전달한다.</li>
</ul>
</li>
<li><p><code>DefaultHandlerExceptionResolver</code> 동작</p>
<ul>
<li>Spring의 내부 예외인지 검사하여 맞으면 에러를 처리하고 아니면 넘어간다.</li>
</ul>
</li>
<li><p>적합한 <code>ExceptionResolver</code>가 없으므로 예외가 서블릿까지 전달되고, 서블릿은 SpringBoot가 진행한 자동 설정에 맞게 <code>BasicErrorController</code>로 요청을 다시 전달한다.</p>
</li>
</ol>
<hr />
<h3 id="참고-자료">참고 자료</h3>
<ul>
<li><a href="https://velog.io/@kiiiyeon/%EC%8A%A4%ED%94%84%EB%A7%81-ExceptionHandler%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%AC">[스프링부트] @ExceptionHandler를 통한 예외처리</a></li>
<li><a href="https://mangkyu.tistory.com/204">[Spring] 스프링의 다양한 예외 처리 방법(ExceptionHandler, ControllerAdvice 등) 완벽하게 이해하기 - (1/2)</a></li>
</ul>