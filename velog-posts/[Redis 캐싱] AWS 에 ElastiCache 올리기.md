<h2 id="🫧-클러스터-생성">🫧 클러스터 생성</h2>
<h3 id="elasticache--redis-oss-캐시--redis-oss-캐시-생성"><a href="https://ap-northeast-2.console.aws.amazon.com/elasticache/home?region=ap-northeast-2#/">ElastiCache &gt;</a> <a href="https://ap-northeast-2.console.aws.amazon.com/elasticache/home?region=ap-northeast-2#/redis">Redis OSS 캐시</a> &gt; <strong>Redis OSS 캐시 생성</strong></h3>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/aec6d656-c27e-4b1c-97fa-9c76fb49f06e/image.png" /></p>
<ul>
<li>클러스터 모드는 동적 할당으로 인한 과금 방지를 위해 비활성화 한다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/be2b1d63-5d05-47bc-a282-1e2286aff6df/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/93fbd724-b0cb-4ffe-a4cc-954bd7595e24/image.png" /></p>
<ul>
<li>⚠️ 프리티어라면 노드 유형은 cache.t2.micro 버전을 사용해야하며 복제본 개수는 0개로 지정해주어야 한다. 해당 설정을 따르지 않으면 99% 확률로 과금될 위험성이 있다.</li>
<li>나머지는 기본 설정으로 두고 다음으로 넘어간다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/8225dfcb-ce6f-4ea5-a762-95e5f4aef5e6/image.png" /></p>
<ul>
<li>보안그룹을 생성한 뒤 추가해준다. 이 때 보안그룹 인바운드는 기본 6379 포트를 포함하도록 하고 ec2 에서 사용하는 소스를 추가해줬다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/otyvs1109/post/35d46d9c-0f2c-40a1-ab92-f3a195167e5a/image.png" /></p>
<ul>
<li>이후 클러스터 생성을 한다.(시간 소요)</li>
</ul>
<h2 id="🫧-ec2-에-redis-설치">🫧 EC2 에 redis 설치</h2>
<ul>
<li>블로그 내용 따라하기!<ul>
<li><a href="https://velog.io/@ncookie/AWS-ElastiCache-Redis-%EC%A0%81%EC%9A%A9">https://velog.io/@ncookie/AWS-ElastiCache-Redis-적용</a></li>
</ul>
</li>
<li>단, redis 접속은 내 endpoint 적용</li>
</ul>