<h2 id="1-docker-hub-설정">1. Docker Hub 설정</h2>
<h3 id="repository-만들기">Repository 만들기</h3>
<ul>
<li><a href="https://hub.docker.com/">https://hub.docker.com/</a></li>
<li>repository 생성 후 name 을 기록해둔다.</li>
</ul>
<h3 id="token-발급">Token 발급</h3>
<ul>
<li>우측 상단 프로필 &gt; Account Settings &gt; Personal access tokens &gt; 토큰 생성</li>
</ul>
<h2 id="2-aws-설정하기">2. AWS 설정하기</h2>
<ul>
<li><a href="https://rachel0115.tistory.com/entry/AWS-EC2-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-%EC%83%9D%EC%84%B1-%EB%B0%8F-%EC%84%A4%EC%A0%95">https://rachel0115.tistory.com/entry/AWS-EC2-인스턴스-생성-및-설정</a></li>
</ul>
<h3 id="인스턴스">인스턴스</h3>
<ul>
<li>이름 및 태그 작성</li>
<li>애플리케이션 및 OS 이미지<ul>
<li>Amazon Linux 선택(프리티어 사용 가능)</li>
</ul>
</li>
<li>인스턴스 유형<ul>
<li>t2.micro 그대로 사용</li>
</ul>
</li>
<li>키 페어<ul>
<li>[name]-key 형식으로 생성</li>
</ul>
</li>
<li>네트워크 설정<ul>
<li>이후 EIP(탄력적 IP)를 할당할 것이기 때문에 <strong>퍼블릭 IP 자동 할당</strong>은 <strong>비활성화</strong> 시켜줍니다</li>
<li>보안 그룹을 생성하여 ssh 터미널을 통해 인스턴스에 접근할 수 있도록 ssh유형의 <strong>인바운드 보안 그룹 규칙</strong>을 추가합니다.</li>
<li>필요에 따라, 웹 프로젝트를 위한 (HTTP) 8080 포트나 (HTTPS) 443 포트를 추가로 열어놓습니다.</li>
</ul>
</li>
<li>스토리지 구성<ul>
<li>프리티어의 볼륨 최대값인 30으로 세팅</li>
</ul>
</li>
<li>인스턴스 시작</li>
<li>인스턴스 생성 확인</li>
</ul>
<h3 id="탄력적-ip-할당">탄력적 IP 할당</h3>
<ul>
<li>좌측 카테고리의 네트워크 및 보안 &gt; 탄력적 IP &gt; <strong>탄력적 IP 주소 할당</strong></li>
<li>기본 세팅 그대로 <strong>할당</strong></li>
<li>이후 할당된 EIP를 선택하고 <strong>작업</strong> - <strong>탄력적 IP 주소 연결</strong>을 클릭</li>
<li>하단 토글을 통해 <strong>인스턴스</strong>를 선택하고 <strong>프라이빗 IP 주소</strong>를 선택한 후 <strong>연결</strong> 버튼을 클릭</li>
<li>초록창으로 ‘탄력적 IP 주소 -이(가) 인스턴스에 연결되었습니다.’ 확인</li>
<li>생성한 EC2 인스턴스에서 탄력적 IP 주소가 할당 된 것을 확인</li>
</ul>
<h3 id="인바운드-아웃바운드-규칙-설정">인바운드 아웃바운드 규칙 설정</h3>
<ul>
<li><p>보안그룹에서 설정</p>
</li>
<li><p>인바운드</p>
<ul>
<li>8080 으로 설정한 경우 ip 뒤에 8080 만 들어오게 할 것이다~ 라는 뜻</li>
</ul>
</li>
<li><p>아웃바운드</p>
<ul>
<li>아웃바운드는 요청 사용자가 누구인지 모르니까 다 열어둔다!</li>
</ul>
</li>
</ul>
<h3 id="rds-데이터베이스-생성하기">RDS 데이터베이스 생성하기</h3>
<ul>
<li><p>엔진 옵션</p>
<ul>
<li>mysql 로 생성</li>
</ul>
</li>
<li><p>설정</p>
<ul>
<li>마스터 사용자 이름<ul>
<li>설정 후 메모(이후 mysql 에 같은 유저와 암호로 접근하게 됨)</li>
</ul>
</li>
<li>마스터 암호<ul>
<li>암호 입력</li>
</ul>
</li>
</ul>
</li>
<li><p>스토리지</p>
<ul>
<li>스토리지 자동 조정 활성화 체크 꼭 풀기!</li>
</ul>
</li>
<li><p>연결</p>
<ul>
<li><p><del>EC2 컴퓨팅 리소스에 연결하기(인스턴스에 연결)</del></p>
</li>
<li><p>퍼블릭 엑세스 허용 → mysql 연결해야 하니까!</p>
<ul>
<li>VPC(Virtual Private Cloud)는 DB 인스턴스에 대한 가상 네트워킹 환경이다. 서브넷 그룹까지 모두 기본적으로 되어있는 설정을 유지한다.</li>
<li>퍼블릭 엑세스는 '아니요'를 설정하면 VPC 내부에서만 DB 인스턴스에 접근이 가능하다. 본인의 EC2에서만 연결해서 사용한다면 '아니요'를 선택해도 되지만, 툴을 통해 접근하거나 로컬의 개발환경 등에서 사용하려면 '예'를 선택해야한다.</li>
<li>VPC 보안그룹에는 내가 기존에 EC2에서 사용하던 보안그룹을 추가한다.</li>
</ul>
</li>
<li><p>추가 구성의 데이터베이스 포트는 기본으로 3306 이다.</p>
</li>
</ul>
</li>
<li><p>추가 구성</p>
<ul>
<li>초기 데이터베이스 이름 설정 필수</li>
</ul>
</li>
<li><p><strong>데이터베이스 생성</strong></p>
</li>
<li><p>이후 보안 그룹에 인바운드(3306) 추가</p>
<ul>
<li>현재 db 라는 이름으로 추가되어 있음. 재사용.</li>
</ul>
</li>
<li><p>데이터베이스 백업에서 사용 가능 상태가 되고 수정 버튼 활성화 되면 db 보안그룹 추가(rds 에는 총 ssh, db 이렇게 두 보안그룹 추가)</p>
</li>
</ul>
<h3 id="메모리-스왑">메모리 스왑</h3>
<ul>
<li>스왑 메모리 설정<ul>
<li>블로그에 나온 것 그대로 진행:  <a href="https://kth990303.tistory.com/361">https://kth990303.tistory.com/361</a></li>
<li>ssh 로 ec2 연결하기: <a href="https://5equal0.tistory.com/entry/AWS-EC2-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%EC%97%90-ssh-%EC%A0%91%EC%86%8D-%ED%95%98%EA%B8%B0">https://5equal0.tistory.com/entry/AWS-EC2-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%EC%97%90-ssh-%EC%A0%91%EC%86%8D-%ED%95%98%EA%B8%B0</a></li>
</ul>
</li>
<li>설명<ul>
<li>ec2 프리티어의 ram 이 1기가 라서 out of memory 발생 가능</li>
<li>이런 서버가 터지는 상황을 방지하기 위해 스왑 메모리를 실행해서 2기가 확보하는 것.</li>
<li>왜 2기가? 원래 메모리의 두 배까지가 최대임.</li>
</ul>
</li>
<li>결과<ul>
<li>램 1GB에, 추가로 실제 디스크를 이용한 swap memory 2GB가 생성된 것을 확인할 수 있다.</li>
</ul>
</li>
</ul>
<h2 id="3-mysql-db-생성">3. MySQL DB 생성</h2>
<h3 id="project-data-source-생성">Project Data Source 생성</h3>
<ul>
<li>Host<ul>
<li><del>인스턴스의 public IPv4 DNS 추가</del></li>
<li>RDS 의 엔드포인트 추가</li>
</ul>
</li>
<li>User &amp; Password<ul>
<li>RDS 에 작성한 것과 동일하게</li>
</ul>
</li>
<li>Test Connection 성공 시 Apply &gt; OK</li>
<li>URL 은 스프링 yml 의 datasource:url 에 입력</li>
</ul>
<h2 id="4-git-action-시-실행될-프로세스를-담은-yml-파일-만들기">4. Git Action 시 실행될 프로세스를 담은 yml 파일 만들기</h2>
<ul>
<li><p><code>.github/workflows/ci-cd.yml</code></p>
<pre><code class="language-html">  # This workflow uses actions that are not certified by GitHub.
  # They are provided by a third-party and are governed by
  # separate terms of service, privacy policy, and support
  # documentation.
  # This workflow will build a Java project with Gradle and cache/restore any dependencies to improve the workflow execution time
  # For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-java-with-gradle

  name: **[spring project name]** ci/cd

  on:
    push:
      branches: [ "develop" ]

  # 해당 스크립트에서 사용될 환경 변수
  env:
    ACTIVE_PROFILE: "prod"
    AWS_REGION: ap-northeast-2
    SERVICE_NAME: **[spring project name]**-api

  permissions:
    contents: read

  jobs:

    build:

      # Github의 워크플로에서 실행될 OS 선택
      runs-on: ubuntu-latest

      steps:
        - name: Checkout
          uses: actions/checkout@v3

        # JDK 17, Corretto 17
        - name: Set up Corretto JDK  17
          uses: actions/setup-java@v3
          with:
            java-version: '17'
            distribution: 'temurin'

        # Secret Setup - application-prod.yml
        - name: Inject env-values to application-prod.yml
          uses: microsoft/variable-substitution@v1
          with:
            files: ./src/main/resources/config/application-prod.yml
          env:
            # Database 환경 변수 주입
            spring.datasource.url: ${{ secrets.DATASOURCE_URL }}
            spring.datasource.username: ${{ secrets.DATASOURCE_USERNAME }}
            spring.datasource.password: ${{ secrets.DATASOURCE_PASSWORD }}

        # gradlew 파일 실행권한 설정
        - name: Grant execute permission for gradlew
          run: chmod +x ./gradlew
          shell: bash

        # Gradle build (Test 제외)
        - name: Build with Gradle
          run: ./gradlew clean --stacktrace --info build
          shell: bash

        # Generate Image Tag
        - name: Make image tag
          run: echo "IMAGE_TAG=$ACTIVE_PROFILE-${GITHUB_SHA::7}" &gt;&gt; $GITHUB_ENV # activeProfile-커밋 hash 값

        # DockerHub 로그인
        - name: dockerhub login
          uses: docker/login-action@v2
          with:
            username: ${{ secrets.DOCKER_USERNAME }}
            password: ${{ secrets.DOCKER_TOKEN }}

        # Docker 이미지 빌드
        - name: docker image build
          run: docker build -t ${{ secrets.DOCKER_USERNAME }}/${{ env.SERVICE_NAME }}:${{env.IMAGE_TAG}} .

        #  Docker Hub 이미지 푸시
        - name: dockerHub push
          run: docker push ${{ secrets.DOCKER_USERNAME }}/${{ env.SERVICE_NAME }}:${{env.IMAGE_TAG}}

        # Deploy **[spring project name]** Service
        - name: Deploy and Start Spring Boot Application
          uses: appleboy/ssh-action@master
          with:
            host: ${{ secrets.HOST_PROD }}
            username: ec2-user
            key: ${{ secrets.PRIVATE_KEY }}
            script: |
              sudo echo "IMAGE_TAG=${{ env.IMAGE_TAG }}" &gt;&gt; .env
              echo "${{ secrets.DOCKER_TOKEN }}" | sudo docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
              sudo docker ps
              sudo docker pull ${{ secrets.DOCKER_USERNAME }}/${{ env.SERVICE_NAME }}:${{env.IMAGE_TAG}}
              if [ $(sudo docker ps -q) ]; then sudo docker stop $(sudo docker ps -q); fi
              if [ $(sudo docker ps -aq) ]; then sudo docker rm $(sudo docker ps -aq); fi
              sudo docker run -d -p 8080:8080 --name ${{ env.SERVICE_NAME }}-${{env.IMAGE_TAG}} ${{ secrets.DOCKER_USERNAME }}/${{ env.SERVICE_NAME }}:${{env.IMAGE_TAG}}
              sudo docker image prune -f
            timeout: 30m
</code></pre>
</li>
<li><p>이 파일을 작성 후 Push 하면 git action 에서 jar 파일을 생성한다.</p>
</li>
</ul>
<h2 id="5-dockerfile-작성">5. Dockerfile 작성</h2>
<ul>
<li><p><code>Dockerfile</code></p>
<pre><code class="language-docker">  # 1. jdk setup
  FROM amazoncorretto:17

  # 2. JAR 파일 경로와 파일명
  ARG JAR_FILE=build/libs/*.jar

  # 3. JAR 파일 이미지 내부로 복사
  COPY ${JAR_FILE} app.jar

  # 4. 어플리케이션 실행 및 로그 출력
  #CMD nohup java -jar /app.jar &gt; stdout.log 2&gt; stderr.log &amp;
  ENTRYPOINT ["java","-Dspring.profiles.active=prod","-jar","/app.jar"]</code></pre>
</li>
<li><p>Spring Boot 애플리케이션을 Docker 컨테이너로 실행하기 위한 설정 파일이다.</p>
</li>
<li><p>코드 설명은 다음과 같다.</p>
<ol>
<li>Amazon에서 제공하는 OpenJDK 17 버전을 베이스 이미지로 사용하며, CI/CD yml 파일에서 설정한 JDK 버전(17)과 일치한다.</li>
<li>Gradle 빌드 후 생성되는 JAR 파일의 위치를 지정하며, <code>build/libs/</code> 디렉토리에 있는 모든 JAR 파일을 대상으로 한다.</li>
<li>위에서 지정한 JAR 파일을 컨테이너 내부로 복사하며, 컨테이너 내부에서는 <code>app.jar</code>라는 이름으로 저장된다.</li>
<li>컨테이너가 시작될 때 실행할 명령어를 지정한다. <code>java</code> 명령어로 JAR 파일 실행하고, <code>Dspring.profiles.active=prod</code> 로 prod 프로필 활성화하고, <code>jar /app.jar</code> 로 복사한 JAR 파일을 실행한다.</li>
</ol>
</li>
<li><p>이 Dockerfile은 CI/CD 파이프라인에서 다음과 같이 사용된다.</p>
<ol>
<li><p>GitHub Actions에서 Gradle 빌드로 JAR 파일 생성</p>
</li>
<li><p>이 Dockerfile을 사용하여 Docker 이미지 생성</p>
</li>
<li><p>생성된 이미지를 Docker Hub에 푸시</p>
</li>
<li><p>EC2 서버에서 해당 이미지를 pull 받아 실행</p>
<p>즉, 이 Dockerfile은 Spring Boot 애플리케이션을 컨테이너화하는 "레시피"라고 생각하면 된다.</p>
</li>
</ol>
</li>
</ul>
<h2 id="6-github-secrets-설정하기">6. Github Secrets 설정하기</h2>
<h3 id="secrets-설정">secrets 설정</h3>
<ul>
<li>Settings &gt; Secrets and variables &gt; Actions &gt; <strong>New Repository Secrets</strong></li>
<li>여기서 yml 파일의 secrets 등록</li>
</ul>
<h2 id="7-docker-설정">7. Docker 설정</h2>
<p><a href="https://jinjinyang.tistory.com/46">https://jinjinyang.tistory.com/46</a></p>
<ul>
<li>error<blockquote>
<p>Error: Process completed with exit code 1.Post dockerhub login0sPost Set up Corretto JDK 170sPost Checkout0sComplete job0s</p>
</blockquote>
</li>
</ul>
<ul>
<li>해결<ul>
<li>따로 작업을 추가하지는 않았고, </li>
<li>git pull —rebase upstream develop</li>
<li>git push upstream develop</li>
</ul>
</li>
</ul>
<p>이렇게 해서 커밋 기록을 정리해주니 해결되었다.</p>
<h3 id="shell">shell</h3>
<ul>
<li>도커 설치<ul>
<li>sudo yum install -y docker</li>
</ul>
</li>
<li>도커 실행<ul>
<li>sudo service docker start</li>
</ul>
</li>
<li>도커 내부 컨테이너 확인<ul>
<li>sudo docker ps</li>
</ul>
</li>
</ul>
<h2 id="8-종료">8. 종료</h2>
<ol>
<li>인스턴스 삭제</li>
<li>네트워크 및 보안 &gt; 탄력적 IP &gt; 릴리즈</li>
<li>RDS 디비 삭제</li>
</ol>
<hr />
<h2 id="참고">참고</h2>
<ul>
<li><a href="https://velog.io/@tilsong/%EC%BD%94%EB%93%9C-Push%EB%A1%9C-%EB%B0%B0%ED%8F%AC%EA%B9%8C%EC%A7%80-Github-Actions-Docker">https://velog.io/@tilsong/코드-Push로-배포까지-Github-Actions-Docker</a></li>
</ul>