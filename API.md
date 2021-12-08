## 1. 회원가입,소셜로그인(카카오,구글,네이버),이메일 인증(비밀번호 찾기)

처음 로그인 구현 시에는 프론트에서 api호출을 했을때 백에서 passport로 소셜 api를 통해 로그인 정보(회원가입정보)를 가져와서 db에 저장한 후 토큰을 만들어 프론트에 보내주는 방식이었습니다. 그런데 막상 프론트와 연결을 했을때 local 로그인은 잘 되었지만 소셜 로그인이 되지 않은 불상사가 일어났습니다.. 계속 CORS에러가 뜨면서 접근이 거부되고 하루종일 매달려봤지만 해결하지 못하였습니다. 그래서 결국 프론트에서 소셜 API를 통해 토큰을 가져와서 우리가 그 토큰을 다시 소셜에 요청해서 유저 정보를 받아오는 방식으로 바꾸었습니다. 

인증 순서
- 프론트에서 로그인시 access token 과 refresh token 을 header에 담아 던져준다.
- 백단에서 받아와 유효한 토큰인지 검사한다. 유효한 토큰이면 access token으로 소셜에 정보를 요청해서 받아온다.
- 요청한 정보를 DB에 저장하고 access token 과 refresh 토큰을 새로 만든다. refresh 토큰은 DB에 저장한다.
- 새로 만든 access token 과 refresh 토큰을 프론트에 전달한다.
- 프론트에서 이 두 token을 어떤 api요청시마다 백에 던져준다.
- 백에서 두토큰을 받으면 refresh token이 만료 되기 전까지는 access token이 만료가 되더라도 새로 갱신을 해준다.

비밀번호 찾기 시에 이메일 인증
- Nodemailer라이브러리를 통해 메일 을 보내고 authcode를 프론트에 보낸다.
- 프론트는 사용자가 입력한 authcode와 비교해서 맞으면 재설정 권한을 준다.


## 2. 채팅,유형별 채팅방(E,i,F,T) 
![image](https://user-images.githubusercontent.com/88120776/145155654-2b513e70-e3c0-4848-a5b9-6bd0d47e17ef.png)
- 유형별 채팅공간 구성
- socket.io라이브러리를 활용해 채팅 구현
- 접속중인 유저목록
- 비속어 필터링 

## 3. 컨텐츠 검색 (Elastic search)
![image](https://user-images.githubusercontent.com/88120776/145156084-43521ce4-4f73-4f3c-92c1-81dbb7051899.png)
- 저희는 다양한 컨텐츠를 모아서 사용자에게 제공하는 서비스로 다양한 컨텐츠 속에서 사용자가 원하는 컨텐츠를 정확하고 다양하게 제공해야 한다고 생각했습니다.
하지만, mySQL에서 Like 연산자는 우선순위 없이 검색어가 포함된 document만 결과로 내려주며, FULL TEXT INDEX와 ngram은 json type의 데이터는 검색불가능 했으며, 불필요한 정보가 함께 검색되는 문제가 있었습니다. 따라서, Nori 한글 형태소 분석기와 다양한 tokenfilter를 제공해주는 elasticsearch를 도입하게 되었고, 고객에 needs에 맞춘 검색기능을 구현할 수 있게 되었습니다.

## 4. 상황별 MBTI 반응 컨텐츠(좋아요, 조회수, 댓글, 공유)
![image](https://user-images.githubusercontent.com/88120776/145156447-8696df11-7f8d-41b8-a31d-0d3425e4899a.png) ![image](https://user-images.githubusercontent.com/88120776/145156474-54449531-a13d-475f-8336-2596e4ab7e7d.png)
 
- 좋아요
- 조회수(조회수중복방지) 
- 댓글 조회


## 5. 밸런스게임 MBTI별 투표 컨텐츠(투표, 조회수, 댓글, 좋아요, 공유)
![image](https://user-images.githubusercontent.com/88120776/145156642-e02fda6c-ecf3-4143-bed2-27c388c2c7ac.png)
- 좋아요
- 조회수 ( 조회수 중복밪지 기능 )
- 댓글
