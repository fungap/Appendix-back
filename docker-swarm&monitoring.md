

## 도커스웜과 모니터링

  <br>
  - 기존에 배포 하던 방식<br>
 
  ![image](https://user-images.githubusercontent.com/88120776/144154863-c920298b-26b5-4db0-8f57-9ccbf1f19d4f.png)<br><br>
  ### 기존 서버의 문제점 <br><br>
  아키텍쳐로 볼수 있듯이 기존에는 AWS 단일 서버 단일 노드에 서비스를 배포 하였습니다.  
  이후 아파치 jmeter로 부하테스트를 진행하였습니다. <br>
  쓰레드 (사용자수 ) : 100~400<br>
  시간 단위 : 10초<br>
  루프 카운트 : 1 <br>
  동시접속자수가 500까지는 이상없이 잘되야한다고 생각했으나 테스트한 성능은 400명이 10초간 1번 access할때 <br>
  평균이 34000ms 최대가 40298ms을 찍게 되었습니다.
    
  ![image](https://user-images.githubusercontent.com/88120776/144164699-19d34c8f-81e7-4a1a-b5a5-001b6a5824fb.png)

  이에 성능의 심각성을 느꼈고 코드에서 최대한 성능적으로 변화 가능할 만한 이슈들을 찾기 시작했습니다.<br> 
  그중 독릭접인 비동기적으로 작동하는 코드들을 한꺼번에 병렬적으로 처리하는 방식으로 바꾸는 작업을 하였습니다. <br>
  promise all 처리방식으로의 코드리팩토링을 한 후에 아래 와같이 268/sec 의 처리량 을 300/sec 의 처리량으로 성능을 개선 시킬수 있었습니다. 

  ![image](https://user-images.githubusercontent.com/88120776/143976089-d5d576b0-cd3b-4200-8008-2f658a5f0633.png)
  
  ![image](https://user-images.githubusercontent.com/88120776/143976055-d57cc528-3783-49ba-bd6c-3131e7dbcaae.png)

  이런 코드수정작업 들을 마쳤으나 여전히 부하테스트에서는 성능 자체에 문제가 많다고 여겨질만큼의 데이터를 확인 할 수 있었습니다.<br>
  이것은 근본적인 EC2 프리티어 인스턴스의 스펙에서 오는 것이라 판단하여 scale-up을 생각하게 되었습니다.
  
  ### docker container 방식으로 배포 전환 
  
  요금제가 조금은 다르지만 스펙을 확실히 늘릴 수 있는 구글 클라우드 플랫 폼으로 이전을 결정하였습니다.<br>
  ec2 프리티어의 스펙은 cpu 1 메모리(GIB)1 -> google cloud computer engin cpu 2 메모리 (GIB)4 
  서버를 이전하는 과정에서 EC2 에서는 잘되던 젠킨스 호환이 GCE 에서는 잘 안되는 현상이 발생되었습니다. 문제는 젠킨스가 ssh 키를 이용 해서 <br>
  서버로 접속하는 과정에서 생겼는데 이는 EC2 와 GCE의 조금은 다른 접속방법의 차이였습니다.<br> 
  스펙 말고 거의 똑같은 ubuntu 서버에 새로 똑같은 서비스를 런칭 하는 것 임에도 여러모로 까다롭다고 생각했습니다. <br>
  이에 확장성이 매우높은 docker container로 서버를 구축 한다면 어떨까 하는 생각이 들었습니다. <br>
  docker를 사용한다면 어느 서버에서도 동일한 환경에서 안정적인 서비스를 구축하기 쉬울 것입니다. 
  
  ![image](https://user-images.githubusercontent.com/88120776/143981870-4095d03d-f2f8-414e-b9d1-82581d8f4332.png)

  
  ### 컨테이너 오케스트레이션<br>
  
  컨테이너로 배포하면서 자연스레 컨테이너를 관리할수 있는 솔루션인 컨테이너 오케스트레이션에 눈이 갔습니다. 
  
  |구분|도커 스웜|메소스|노매드|쿠버네티스|
|:------:|:---:|:---:|:---:|:---:|
|설치난이도|쉬움|매우어려움|쉬움|어려움|
|사용편의성|매우좋음|좋음|매우좋음|좋음|
|세부설정지원|거의없음|있음|거의없음|다양하게 있음|
|안정성|매우안정적임|안정적임|안정적임|매우안정적임|
|확장성|어려움|매우잘됨|어려움|매우잘됨|
|정보량|많음|적음|적음|매우많음|
  
  오버엔지니어링이라고 생각했지만 기술적 챌린지로 쿠버네티스를 해보려고 하였습니다.<br>
  로컬에서 vagrant와 virtual 머신으로 쿠버네티스 테스트 환경을 구축하였습니다.<br>
  테스트를 마치고 실제로 GCE에 클러스터를 구축하는 데에 아뿔사 쿠버네티스 1.20버전 이후로는 <br>
  docker를 컨테이너 런타임으로 지원하지 않는다는 사실을 알게되었습니다. <br>
  따라서 다른 컨테이너 오케스트레이션으로의 전환을 하려했고<br>
  쿠버네티스에 쏟은 시간이 컸던 지라 남은 시간이 없었기 때문에 설치 난이도와 사용 편의성에서 좋은 도커 스웜을 적용시키기로 하였습니다.
  
  ### docker in docker vs node 별 인스턴스<br>
  GCE 에는 총 cpu2 메모리 4GIB 스펙의 인스턴스 를 3개월간 3대를 무료로 사용할수 있습니다.<br>
  저희는 배포서버 뿐만아니라 1개의 jenkins 서버 와 1개의 테스트서버가 필요하다고 생각햇습니다.<br>
  (jenkins서버를 따로 구축해놓은이유는 젠킨스의 메모리 사용률이 커서 서버가 같이 다운되었던 적이 많았기 때문입니다.)<br>
  따라서 배포용 서버의 스펙을 cpu2메모리4로 할당하고 테스트 서버도 마찬가지로 cpu2메모리4 를 두었을때 jenkins서버까지 돌리면 따로 새로운 인스턴스에서 node 를 돌릴
  여유가 되지 않았습니다.<br>
  그래서 docker in docker로 개별 node를 도커 내 컨테이너로 구현을 해서 도커스웜의 기능을 온전히 사용 하되 서버는 1개를 유지하는 전략을 사용하였습니다.<br>
  
  외부저장소로는 docker hub를 사용하였습니다. docker hub는 비공개 계정으로 하나의 repository만 사용할 수 있으나 저희가 운용하기에는 무리가 없어보입니다.<br>
  다만 후에 비공개 폐쇄망을 이용하게 될 수도 있으므로 내부적으로 registry도 구현 하였습니다.<br>
  
  ![image](https://user-images.githubusercontent.com/88120776/143992140-3ba934cd-4e33-4f48-a6f8-76c14758fcbe.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ![image](https://user-images.githubusercontent.com/88120776/143992519-82d4e079-d082-4a05-816b-047865aa8592.png)
  ![image](https://user-images.githubusercontent.com/88120776/143992523-8d5ac8db-bc17-4036-be78-00b87586a18a.png)
   ![image](https://user-images.githubusercontent.com/88120776/143992523-8d5ac8db-bc17-4036-be78-00b87586a18a.png)
   ![image](https://user-images.githubusercontent.com/88120776/143992523-8d5ac8db-bc17-4036-be78-00b87586a18a.png)
  
  ![image](https://user-images.githubusercontent.com/88120776/143990487-e8fc48c0-4235-4ab7-802f-f4d11ec77013.png)
  
  ### 아래 docker-compose.yml 파일로 docker manager,worker01,worker02,registry생성, 후에 manager 에서 docker swarm init 후 join토큰으로 간단하게 dockerswarm 서버 환경을 구축 할 수 있습니다.
```
  version: "3"
services:
    registry:
        container_name: registry
        image: registry:2.6
        ports:
            - 5000:5000
        volumes:
            - "./registry-data:/var/lib/registry"
    manager:
        container_name: manager
        image: docker:18.05.0-ce-dind
        privileged: true
        tty: true
        ports:
            - 8000:8000
            - 9000:9000
            - 7000:7000
        depends_on:
            - registry    
        expose:
            - 3375
        command: "--insecure-registry registry:5000"    
        volumes:
            - "./stack:/stack"
    worker01:
        container_name: worker01
        image: docker:18.05.0-ce-dind
        privileged: true
        tty: true
        depends_on:
            - manager
            - registry  
        expose:
            - 7946
            - 7946/udp
            - 4789/udp
          
        command: "--insecure-registry registry:5000"    
    worker02:
        container_name: worker02
        image: docker:18.05.0-ce-dind
        privileged: true
        tty: true
        depends_on:
            - manager
            - registry
        expose:
            - 7946
            - 7946/udp
            - 4789/udp
            
        command: "--insecure-registry registry:5000"    
 ```
 ### 도커 스웜을 사용한 컨테이너 관리
  도커 스웜의 편리한 노드 관리 기능 에는 스케줄링, 클러스터링, 서비스 디스커버리, 로깅, 롤링업데이트, 등을 사용 할수 있습니다.<br>
  
  트러블 슈팅<br>
  .env파일 mount시 위치에 따르는 롤링업데이트가 안되는 문제 발생<br>
  저희 코드는 .env파일을 app.ts 파일과 같은 위치에 두었고 서비스 생성 문에서 mount 옵션으로 usr/src/app 의 위치를 공유 하도록 설정하였습니다.
  (--mount type=volume,src=env,target=/usr/src/app) <br>
  usr/src/app 은 소스코드가 전부 복사되는 루트 폴더 였고 .env파일을 sorce지점의 env폴더에서 직접 생성을 하여 공유 하였습니다.
  그런데 첫 배포(잘 동작함) 이후 update를 이용한 롤링업데이트가 되지 않는 현상이 발생하였습니다. <br>
  해결과정은 다음과 같습니다.<br>
  첫번째 이미지파일이 문제 있는지 확인 -> 로컬에서 이미지만 받아서 실행시 문제 없음<br>
  두번째 롤링 업데이트에 이상이 있는지 확인 ->  업데이트시에 로그를 받아서 확인 시에 문제가 없음<br>
  세번째 실질적으로 코드가 수정이 되었는지 확인 -> 컨테이너 안으로 직접들어가 코드를 뜯어보니 수정이 안되어 있었음!!<br>
  -> 이것은 공유 볼륨이 업데이트 되기 전의 소스였기 때문에 업데이트가 되었어도 공유 볼륨이 강제적으로  이전 소스로 되돌리는 문제 였던 것<br>
  해결 하기 위해 mount 옵션을 전체 코드가 있는 usr/src/app 으로 설정하지 않고 usr/src/app/env 로 설정해 .env 파일만 env 폴더에서 공유만 함으로써<br>
  업데이트가 될때 소스코드는 볼륨에 포함되지 않으므로 다시 이전으로 되돌아가는 현상을 방지 하였습니다. <br>
  
  
  ![image](https://user-images.githubusercontent.com/88120776/143999928-1df0340e-0c98-44e0-8c44-73f9bd38648e.png) .env 파일 env폴더로 이동 ![image](https://user-images.githubusercontent.com/88120776/144002776-ae26ede3-9823-42a9-b50b-17e399c94d02.png)
  
  ### 도커스웜과 오토스케일링
  도커스웜은 오토스케일링을 지원하지 않습니다. 따라서 모니터링에 신경을 더 써줘야 할 것 같다고 생각했습니다. <br>
  그라파나에 CPU가 85퍼센트가 이상일때 슬랙에 알람을 주게 설정을 합니다.<br>
  알람을 받으면 직접 스케일 아웃을 해줍니다. <br>
  docker service scale 명령으로 스케일 수를 지정해주면 도커 스웜이 알아서 스케쥴링 해줍니다.<br><br>
  ![image](https://user-images.githubusercontent.com/88120776/144160609-d95c876c-773a-482c-8c2e-33110a04efb0.png)
  ![image](https://user-images.githubusercontent.com/88120776/144156486-9befc28c-b555-4d73-ab8b-ae9783e09c12.png)

  

  ### 그라파나 와 프로메테우스 
  도커스웜에서는 모니터링을 제공해주지 않기 때문에 직접 구축을 해주어야 했습니다. 
  모니터링 툴로는 오픈소스이기도 하고 서로 연동성이 좋은 그라파나와 프로메테우스를 선택했습니다. 
  
  ### 매니저노드 
  ![image](https://user-images.githubusercontent.com/88120776/144006144-f7d4eeb5-277d-4089-88db-150f03278532.png)
  ### worker노드
  ![image](https://user-images.githubusercontent.com/88120776/144006606-37339d02-c0ce-4d9d-b539-b90fe7871082.png)
  ### 전체적인 리소스사용량
  ![image](https://user-images.githubusercontent.com/88120776/144008474-78b8c04d-d310-4e30-8749-a7cae78d5950.png)

  
  기본적인 노드별 가동되는 컨테이너 수, 리소스 사용량과 전체적인 리소스 사용량을 모니터링 할 수 있습니다. 
  이 모니터링만을 보고서는 의미있는 데이터라든가 결과 값을 알수가 없었습니다. 따라서 부하를 주는 테스트와 함께 모니터링을 같이 해보며 의미있는 데이터를 산출해보고 싶었습니다.
  
  ### 부하테스트 
  배포용 서버와 똑같은 테스트용 서버를 만들었습니다. 똑같은 환경을 구축하는데 20분도 걸리지 않았습니다. 할일은 docker와 docker compose를 설치하고 yml파일을 생성해서
  돌려주고 도커스웜으로 조인만 해주면 되었습니다. <br>
  아파치jmeter로 부하를 주고 그라파나로 모니터링 하였습니다.<br>
  부하테스트 시나리오
  1000명 부터 100명씩 10초간의 시간을 두고 1 번씩 access한다. 각 테스트의 간격텀은 3분 <br>
  (시간을 10초로 준이유는 너무 짧은 시간은 그래프에 잘 나타 나지 않아서 최대한 유의미한 그래프를 뽑기 위한 최소한의 시간이라 판단했습니다)
  
  ![image](https://user-images.githubusercontent.com/88120776/144010302-0c0898bb-e61e-42a4-97c0-5fbfd7a9fc41.png)
  ![image](https://user-images.githubusercontent.com/88120776/144035093-fb415a03-d8b3-402d-95df-a47880d1717c.png)
  ![image](https://user-images.githubusercontent.com/88120776/144035453-db7b37fc-1d29-4362-bebf-8cabf87d1302.png)

  ![image](https://user-images.githubusercontent.com/88120776/144034107-274ec94b-6030-4feb-b5d4-02537f7acd3e.png)
  
  2300명 10초간 1번씩 엑세스의 경우 최고 cpu사용률이 99%를 찍었으며 최대 응답값도 2000을 넘게 되었습니다.. 
  cpu사용률 80퍼센트 이하까지를 서버 안정성의  마지노선이라고 봤을때 2100명의 동시접속자가 10초간 1번씩의 access를 했을때까지가 최대한의 부하라고 생각 할수 있었습니다.

 
  
  
  

  
  
 


