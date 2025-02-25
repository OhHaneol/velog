<h2 id="📍-문제-상황">📍 문제 상황</h2>
<ul>
<li>로컬에서 upstream 의 develop 브랜치를 pull 한 다음 분기한 feature 브랜치에서 작업을 진행했다.</li>
<li>이후 PR 을 날리니 아래와 같이, 작업한 커밋 이전에 develop 브랜치에 포함되어 있는 커밋들이 딸려온 것을 확인했다.</li>
</ul>
<img src="https://velog.velcdn.com/images/otyvs1109/post/fb0b8d0f-6743-4f33-a631-5705ded2e0b5/image.png" width="400" />


<h2 id="📍-분석">📍 분석</h2>
<ul>
<li>develop 브랜치를 pull 하는 과정에서 merge 가 있던 것으로 추정된다.</li>
</ul>
<h2 id="📍-해결">📍 해결</h2>
<ul>
<li>명령어를 다음과 같이 바꿔서 진행하니 해결되었다.</li>
</ul>
<pre><code class="language-shell">git fetch --all
git reset --hard upstream/develop</code></pre>
<img src="https://velog.velcdn.com/images/otyvs1109/post/af86200a-4fc6-48dc-80a7-e628228c640b/image.png" width="400" />

<ul>
<li>그러나 협업 시, 특히 브랜치를 공유하는 경우 이 방법은 위험하다.</li>
<li>다음과 같이 <code>pull --rebase</code> 명령어를 사용하는 것이 더 안전하다.</li>
</ul>
<pre><code class="language-shell">git pull --rebase upstream develop</code></pre>