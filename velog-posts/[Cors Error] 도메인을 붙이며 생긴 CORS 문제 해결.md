<h2 id="📍-문제-상황">📍 문제 상황</h2>
<p>기존 IP 주소를 문자 주소로 변경하면서 다음과 같은 CORS 문제가 생겼다.</p>
<blockquote>
<p>Access to fetch at "IP 주소" from origin '문자 주소' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.</p>
</blockquote>
<h2 id="📍-분석">📍 분석</h2>
<ol>
<li>오류를 해석하자면, “<code>문자 주소</code> origin 으로부터의 <code>ip 주소</code> 접근이 cors 정책에 의해 막혔다”</li>
<li>즉 해당 네임 도메인에 대해 교차 출처 리소스 공유(Cross-Origin Resource Sharing, CORS)를 허용함으로써 신뢰 가능한 다른 Origin 을 읽을 수 있도록 해줘야 한다는 말이다.</li>
<li>기존 로컬 서버 주소인 <code>http://localhost:63342</code> 가 이미 <code>allowedOrigins</code> 에 추가되어 있었으나, 이는 배포 환경에는 영향을 끼치지 않는다. 따라서 기존에 잘 돌아가던 배포 환경에서는 IP 서버 주소가 브라우저의 Origin 과 일치했기 때문에 정상적으로 돌아갔을 것이다.</li>
<li>다음으로 여기서 내 착각이 있었다. 사용자가 아무리 문자 주소를 쓰도록 바꿨다고 한들 이는 배포 서버에 오기 전에 네임 서버(DNS)를 거쳐서 등록된 IP 주소로 변경되어 전달된다. 그래서 나는 문자 주소로 바꾼 것은 영향이 없을 것이라고 생각했다.</li>
<li>그런데 알고 봤더니, DNS와 상관 없이, 브라우저가 판단하는 <strong>Origin</strong>은 <strong>URL에 입력된 요소</strong>를 가지고 판단한다고 한다.(참고로 Origin의 동일함은 두 URL의 구성 요소 중 Protocol, Host, Port 이 세 가지만 동일하다면 동일 출처로 판단한다.)</li>
<li>즉, 아무리 <code>IP 주소</code>를 돌려준다고 해도 브라우저에서는 <code>문자 주소</code> 로 판단하기 때문에 해당 URL 을 다른 Origin 으로 판단, CORS 정책에 따라 블락하는 것이다.</li>
</ol>
<h2 id="📍-해결">📍 해결</h2>
<p>따라서 WebConfig의 addCorsMappings 에서 allowedOrigins 에 해당 문자 주소를 등록해야 한다.
이 때 로컬과 배포에서 Origin 이 상이하기 때문에, 프로파일을 사용하여 <code>@Value</code>를 주입하여 해결하였다.</p>