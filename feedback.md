[readme로-돌아가기](https://github.com/fungap/fungap-back)

## Feedback

### CI/CD란 뭔가?

기본개념은 지속적인 통합 지속적인 배포를 말합니다.<br>
저희는 CI/CD툴로 젠킨스를 사용하였으며 git push 가 일어나면 서버에 자동적으로 통합 배포 파이프라인을 구축하였습니다.

### 왜 scale out 을 안하고 scale up을 하였는지

기본적으로 프리티어 aws에서는 jenkins 조차 제대로 돌아가지 않는 현상이 발생
scale out을 한다고 하더라도 결국엔 manager node인 서버에서는 모니터링 컨테이너 등을 운용해야 하는데
프리티어인 서버성능으로는 운용하기 힘들다고 판단하였고
AWS 프리티어 1코어 1GIB 메모리 말고 무료로 사용할 수 있는 GCP 2코어 4GIB메모리를 사용하지 않을 이유가 없다고 생각했습니다.

### GCP와 AWS 서비스를 함께 사용한 이유는?

처음에는 AWS ec2 와 RDS를 사용하였으나 성능테스트를 하였을때 ec2서버의 응답시간이 400명일때 30초 가까이 나옴으로 인해서
scale up을 결정하게 되었고 무료로 사용할수 있는 GCP 서비스를 배포서버로 선택하였습니다.

### AWS 서비스를 사용하다가 GCP 서비스를 사용했는데 AWS에서도 충분히 가능한 설정이라고 생각된다. 다른 이유가 있나?

EC2의 성능을 올리기 위해서는 비용이 발생하는데 GCP는 3개월간 무료로 더 좋은 스펙의 서버를 이용할수 있었습니다.

### ELK만 사용해도 모니터링이 충분히 가능하다고 생각된다. 프로메테우스랑 그라파나까지 사용한 이유는?

그라파나는 ELK보다도 매트릭 정보를 다루는데 더 좋다고 판단하여 사용하였습니다.

- #### Grafana
 + Grafana는 시스템 관점(cpu, 메모리, 디스크)의 메트릭 지표를 시각화하는데 특화되어 있다.<br>
   다양한 애플리케이션 메트릭 스토리지 데이터 소스를 지원하고 모니터링 및 즉각적인 실시간 경고를 <br>
   이메일 슬랙등, 알림기능을 무료로 제공한다.<br>
   Grafana는 데이터를 기반으로 한 분석, 패턴 식별 및 예측을 위해 시계열 데이터로 효과적으로 작동한다. <br>
   데이터 모니터링을 위해 소규모 및 대규모 조직 모두에게 탁월한 선택이다.<br>

 + 키바나는 elasticsearch에 묶여 있지만, 그라파나는 다양한 데이터베이스를 선택할 수 있다. <br>
   kibana는 알림기능에 XPack 유료플랜이 필요하다. kibana는 로그 분석에 중점을 두고 기존 
   원시 로그 데이터에서 메트릭을 추출하는 방법을 사용자에게 제공하긴 하지만 따로 설정이 필요하다. <br>

- #### Kibana
 + Kibana는 주로 로그 메시지 분석에 사용된다.<br>
   다양한 표현방법은 Kibana가 압도적으로 우세하고, 로그 데이터의 탐색 및 분석을 관련 필터를 <br>
   쿼리하고 적용하여 시각화를 얻고 자신의 방식으로 표시할 수 있다. 로그 분석에 중점을 두고 <br>
   기존 원시 로그 데이터에서 메트릭을 추출하는 방법을 사용자에게 제공한다.<br>

 + Kibana 사용자는 수집된 로그에 사용자는 데이터 파이프라인을 생성하기 위한 다양한 플러그인을 제공하는 <br>
   서버 측 프레임워크인 Logstash와 함께 Kibana를 사용할 수도 있다. <br>
   Elasticsearch를 사용하여 데이터를 관리하는 네트워크가 있는 모든 규모의 조직에 탁월한 선택이다.<br>

> 두 도구 모두 데이터 시각화 및 분석을 위한 인상적인 기능 세트를 가지고 있지만 주로 다른 목적으로 사용된다. <br>
 Grafana는 InfluxDB 및 Graphite와 같은 시계열 데이터 소스를 사용하여 비즈니스 메트릭을 분석하는 반면 <br>
 ELK 스택의 일부인 Kibana는 로그 데이터를 탐색하는 데 사용됩니다. 조직에서는 둘 다 모니터링 인프라의 <br>
 일부로 사용하기도 합니다. <br>

### 프로메테우스랑 그라파나는 AWS에 구성하기 쉬운 서비스 형태로 있는데 직접 구성한 것처럼 보여진다. 이유는?

도커스웜은 모니터링을 지원하지 않았기 때문에 여러 툴중에서 오픈소스이면서 연동성이 좋은 그라파나와 프로메테우스를 사용하였습니다.<br>
직접 구성한 이유는 서비스 형태로 가져다가 쓰기보다는 직접 세팅하면서 세부적인 설정등을 조금 더 디테일하게 이해하고 스터디 하고 싶었기 때문입니다.

### 오토 스케일링은 무엇인가? 스케일 업과 아웃의 차이는 무엇인가?

자동적으로 스케일을 조정해주는 기능입니다. 예를 들면 쿠버네티스는 CPU사용률 기반으로 디플로이먼트로 실행된 pod의 개수를 늘리거나 줄입니다. <br>
스케일 업은 서버 그 자체를 증강하면서 처리 능력을 향상 시키는 것 프로세서 자체를 고성능 모델로 옮겨놓는것!<br>
예) 1CPU 1GIB 메모리 -> 2CPU 4GIB메모리 <br>
스케일 아웃은 접속된 서버의 대수를 늘려 처리 능력을 향상시키는 것이다.

### 하나의 서버에서 여러개의 도커를 실행시키고 도커 스웜으로 관리하는 것이 가능한데 도커 인 도커를 한 이유는 무엇인가?

현재 저희가 도커 스웜을 구축한 방식을 살펴보면 1개의 GCP 서버가 있고 그 안에서 docker 로 컨테이너 3개가 실행 중입니다. 매니저1 워커1,워커2 이 컨테이너들이 클러스터가 됩니다.<br>
이 컨테이너들 안에서 저희 웹서버가 컨테이너로 실행 되고 있습니다. 제가 도커인 도커 방식을 이해한 개념은 도커 안에서 도커가 실행 한다는 것입니다.
따라서 저는 도커 인 도커방식을 취하고 있다고 이해하고 있었는 데 도커인도커를 잘못 이해 한 내용이 있는 것 같습니다..<br>
그렇기 때문에 docker in docker보다는 저희가 어떻게 1개의 서버에서 도커스웜을 구축했는지에 대한 것을 알아주셨으면 합니다.

### 검색 성능을 위해서 MySQL 대신에 Elastic Search를 사용했는데 얼마나 빨라졌나?
현재 검색 기능은 더 빠르게 사용자들에게 결과를 전해주는것 보다 검색어에 연관된 데이터를 더 정확하도 다양하게 <br>
제공하는데 초점을 두고 개발에 임하였기 때문에, 어느것이 더 빠른가에 대한 데이터는 준비하지 못했습니다. <br>
추후에 따로 비교 분석한 데이터를 github에 최신화 시켜놓도록 하겠습니다!! <br>
감사합니다.<br>

### Elastic Search를 사용해서 사용자의 needs를 맞출 수 있다고 했는데 사용자의 needs는 무엇이고 어떻게 알아낸 것인지?
검색단어나 저장된 데이터를 어떤 tokenizer와 tokenfilter로 나누었는지에 따라 같은 검색단어를 입력했을 때에도 <br>
다른 결과를 보여주게 됩니다. 따라서 각 게시물의 조회수, 좋아요 수, 분석엔진을 통해 사용자들이 어떤 검색어를 <br>
많이 입력했는지를 집게하고, 정리된 데이터로 최적의 검색 알고리즘을 고찰해 볼 수 있습니다.<br>
이러한 점은 단순히 ngram으로 검색단어나 DB에 저장된 데이터를 정해진 글자수로 자르기만 해서는 구현할 수 없다고 판단했습니다. <br>
리뷰는 직접적으로 사용자의 의견을 얻을 수 있겠지만 실재 리뷰를 남기는 사람은 그 중 일부에 불과 합니다. <br>
따라서 앞에서 언급한 데이터들의 분석으로 대다수 사용자의 간접적인 needs를 얻어낼 수 있다고 생각했습니다. <br>

### 해당 서비스를 사용하는 유저가 많아지면 채팅로그 2만개는 순식간에 쌓일 것 같은데 채팅 기록을 확인해야 하는 상황이 오면 어떻게 할 것인가?

현재 DB에 채팅이 과도하게 쌓이지 않도록 일정 갯수가 넘어가면 일단 chatlogBackup 테이블로 옮기고 chatlog에서 삭제되도록 로직이 구현되어 있습니다. 기록을 확인하려면 백업테이블에서 확인하면 됩니다.

### 채팅 로그를 몇 주 또는 몇개월 전의 로그를 확인해야할 일은 없다고 생각하는 것인지?

이 부분도 제가 너무 데이터에 대해 안일하게 생각했던 것 같습니다. 현재는 백업테이블에 저장해서 필요에 따라 확인할 수 있도록 하였습니다.

### 정확한 로깅을 위해서 욕설이라도 채팅은 전체 데이터를 DB에 저장하는 것이 좋아보인다. 욕설이 필요없는 데이터라고 생각한 이유는 무엇인가?

- 좋은 답변 감사합니다. 욕설 데이터도 나중에 경고메세지 라던지 제재를 가한다던지 하는 데이터로 쓸 수도 있고 어떻게 활용하냐에 따라 좋은 정보가 될 수 있는데 제가 미처 생각을 못했습니다.
  현재는 욕설 필터링에 걸리면 그 부분은 채팅로그(조회가 일어나는)에 저장하는 것이 아니라 백업채팅로그에 저장하도록 하여서 사용자에게 보여주지는 않되 데이터는 보존하는 형태로 구현을 하였습니다.

### Helmet이 어떤 모듈인가

Express.js 사용시 Http 헤더 설정을 자동으로 바꾸어주어 잘 알려진 몇가지 앱의 취약성으로 부터 앱을 보호 할 수 있는 패키지이다.<br>

- Helmet는 다음과 같은 미들웨어로 이루어 져 있다.<br>
  helmet.contentSecurityPolicy(options) : XSS(Cross site scripting) 공격 및 기타 교차 사이트 인젝션 예방

* [Content Security Policy (CSP) - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

helmet.crossOriginEmbedderPolicy() : Cross-Origin-Embedder-Policy 헤더를 require-corp로 설정

- [Cross-Origin-Embedder-Policy - HTTP | MDN](https://developer.cdn.mozilla.net/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)

helmet.crossOriginOpenerPolicy() : Cross-Origin-Opener-Policy 헤더를 설정

- [Cross-Origin-Opener-Policy - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy)

helmet.crossOriginResourcePolicy() : Cross-Origin-Resource-Policy 헤더를 설정

- [Consider deploying Cross-Origin Resource Policy.](https://resourcepolicy.fyi/)
- [Cross-Origin-Resource-Policy - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy)

helmet.expectCt(options) : Expect-CT 헤더를 설정하여 SSL 인증서 오발급을 예방

- [Certificate Transparency - Web security | MDN](https://developer.mozilla.org/en-US/docs/Web/Security/Certificate_Transparency)
- [Expect-CT - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expect-CT)

helmet.referrerPolicy(options) : Referrer-Policy 헤더를 설정하여 민감한 정보 유출 예방

- [Referer - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)
- [Referer header: privacy and security concerns - Web security | MDN](https://developer.mozilla.org/en-US/docs/Web/Security/Referer_header:_privacy_and_security_concerns)
- [Referrer-Policy - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)

helmet.hsts(options) : Strict-Transport-Security 헤더를 설정하여 보안연결(HTTPS) 강제

- [Strict-Transport-Security - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)

helmet.noSniff() : X-Content-Type-Options 헤더를 nosniff로 설정하여 MIME 스니핑 예방

- [MIME types (IANA media types) - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types#MIME_sniffing)
- [X-Content-Type-Options - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)

helmet.originAgentCluster() : Origin-Agent-Cluster 헤더를 설정하여 오리진간 문서를 별도 에이전트 클러스터로 분리

- [HTML Standard](https://whatpr.org/html/6214/origin.html#origin-keyed-agent-clusters)

helmet.dnsPrefetchControl(options) : X-DNS-Prefetch-Control 헤더를 설정하여 DNS 프리패칭을 조절

- [X-DNS-Prefetch-Control - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-DNS-Prefetch-Control)

helmet.ieNoOpen() : X-Download-Options 헤더를 설정하여 ie8 이상에서만 사용할 수 있도록 함

- [IE8 Security Part V: Comprehensive Protection](https://docs.microsoft.com/en-us/archive/blogs/ie/ie8-security-part-v-comprehensive-protection)

helmet.frameguard(options) : X-Frame-Options 헤더를 설정하여 clickjacking 공격을 예방

- [Clickjacking](https://en.wikipedia.org/wiki/Clickjacking)
- [CSP: frame-ancestors - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)
- [X-Frame-Options - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)

helmet.permittedCrossDomainPolicies(options) : X-Permitted-Cross-Domain-Policies 헤더를 설정하여 크로스도메인 컨텐츠 정책을 설정

- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)

helmet.hidePoweredBy() : X-Powered-By 헤더를 제거하여 웹앱의 프레임워크를 특정할 수 없도록 함

- [Security improvement: don't reveal powered-by by madarche · Pull Request #2813 · expressjs/express](https://github.com/expressjs/express/pull/2813#issuecomment-159270428)

helmet.xssFilter() : X-XSS-Protection 헤더를 0으로 설정하여 크로스사이트 스크립트를 이용한 공격을 예방

- [X-XSS-Protection: header should be disabled by default · Issue #230 · helmetjs/helmet](https://github.com/helmetjs/helmet/issues/230)
- [X-XSS-Protection - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)

### http와 https의 차이점은 무엇인가.

HTTP(Hyper Text Transfer Protocol)란 서버/클라이언트 모델을 따라 데이터를 주고 받기 위한 프로토콜이다.

즉, HTTP는 인터넷에서 하이퍼텍스트를 교환하기 위한 통신 규약으로, 80번 포트를 사용하고 있다. 따라서 HTTP 서버가 80번 포트에서 요청을 기다리고 있으며, 클라이언트는 80번 포트로 요청을 보내게 된다.
HTTP는 애플리케이션 레벨의 프로토콜로 TCP/IP 위에서 작동한다. HTTP는 상태를 가지고 있지 않는 Stateless 프로토콜이며 Method, Path, Version, Headers, Body 등으로 구성된다.
![image](https://user-images.githubusercontent.com/46738141/144450689-10b95e59-5945-4d7e-be9e-6e28a9b83547.png)

하지만 HTTP는 암호화가 되지 않은 평문 데이터를 전송하는 프로토콜이였기 때문에, HTTP로 비밀번호나 주민등록번호 등을 주고 받으면 제3자가 정보를 조회할 수 있었다. 그리고 이러한 문제를 해결하기 위해 HTTPS가 등장하게 되었다.
<br><br>
HTTPS는 HTTP에 데이터 암호화가 추가된 프로토콜이다. HTTPS는 HTTP와 다르게 443번 포트를 사용하며, 네트워크 상에서 중간에 제3자가 정보를 볼 수 없도록 공개키 암호화를 지원하고 있다.

HTTPS는 SSL과 같은 프로토콜을 사용하여 공개키/개인키 기반으로 데이터를 암호화하고 있다. 데이터는 암호화되어 전송되기 때문에 임의의 사용자가 데이터를 조회하여도 원본의 데이터를 보는 것은 불가능하다.

그렇다면 이제 서버는 클라이언트가 요청을 보낼 때 암호화를 하기 위한 공개키를 생성해야 하는데, 일반적으로는 인증된 기관(Certificate Authority) 에 공개키를 전송하여 인증서를 발급받고 있다. 자세한 과정은 다음과 같다.

A기업은 HTTP 기반의 애플리케이션에 HTTPS를 적용하기 위해 공개키/개인키를 발급함
CA 기업에게 돈을 지불하고, 공개키를 저장하는 인증서의 발급을 요청함
CA 기업은 CA기업의 이름, 서버의 공개키, 서버의 정보 등을 기반으로 인증서를 생성하고, CA 기업의 개인키로 암호화하여 A기업에게 이를 제공함
A기업은 클라이언트에게 암호화된 인증서를 제공함
브라우저는 CA기업의 공개키를 미리 다운받아 갖고 있어, 암호화된 인증서를 복호화함
암호화된 인증서를 복호화하여 얻은 A기업의 공개키로 데이터를 암호화하여 요청을 전송함

![image](https://user-images.githubusercontent.com/46738141/144450339-d05bfb82-0991-4d77-8df9-ff46efa8bee9.png)

출처: https://mangkyu.tistory.com/98 [MangKyu's Diary]

### 단순하게 https를 사용한다고 해서 보안이 더 나아지는가?

### node와 mysql을 조합하여 사용했는데, 이에 대한 단점이 무엇이라고 생각하시나요?1

### node의 특징과 장점

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=silver889&logNo=220204778189

### node와 MySQL이 어떤 상황에 사용되어야 유리할까요?
