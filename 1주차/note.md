#### 1. 도커의 기초 - https://github.com/skarltjr/Memory_Write_Record/issues/21
#### 2. 서버를 구축하는 방식
   - 크게 2가지의 방식이 존재한다
      1) onpremise = 자체적으로 데이터센터나 서버실을 운영하는 것
      2) 클라우드 = 클라우드 사업자 ( aws , gcp , azuer...)등을 사용하는 방식
      - 서비스 환경에 따라 선택
#### 3. 서버 구축방식의 비교   // 하늘색 부분은 사용자 혹은 관리자가 직접 작업을 해야하는 파트
##### ✔︎  onpremise
![화면 캡처 2021-08-04 213718](https://user-images.githubusercontent.com/62214428/128181857-08bf9d89-7d98-4332-b4dc-9d3e0800fcbb.png)
   - 많은 파트를 직접 작업해야한다.
   - 따라서 서버의 규모가 커질수록 기업이 부담해야하는 부분이 기하급수적으로 증가한다.

##### ✔︎  cloud  // 주황색 부분이 클라우드 사업자에서 담당, 처리해주는 파트
   - 1. iaas / infra as a service
     - ex) aws의 ec2
     - ![화면 캡처 2021-08-04 214028](https://user-images.githubusercontent.com/62214428/128182252-284ade73-6bb1-4957-b29b-d3e144a53845.png)
   - 2. paas / platform as a service
     - ![화면 캡처 2021-08-04 214312](https://user-images.githubusercontent.com/62214428/128182661-fc726c8f-cb60-4ce6-86b3-83b503ceda4f.png)
   - 3. saas / software as a service
     - ![화면 캡처 2021-08-04 214451](https://user-images.githubusercontent.com/62214428/128182847-96ae9e31-303e-405c-92f7-99c06f3c2e13.png)

 
- 이렇게 다양한 방법의 서버 구축방식이 존재한다.
- 그러나 서비스는 유저가 많아지고 서비스가 성장, 고도화되면서 필연적으로 이주( migration )가 필요하다. 
- ✔︎ 이러한 이유로 바로 Docker가 급부상하게 된 것
- ✔︎ 도커는 컨테이너 기반으로 실행 -> 어느 환경으로 이주하든 의존성에 영향을 받지 않고 migration이 가능하다!  
- ✔︎ 즉 이식성!이 높다.

#### 도커와 VM(가상머신)의 가장 큰 차이는 무엇인가?
- ![화면 캡처 2021-08-04 215521](https://user-images.githubusercontent.com/62214428/128184309-3aa3ec76-c2b8-4f03-ba61-dd2d14460435.png)
- 가상머신은 개별 어플리케이션마다 os가 별도로 실행된다. 당연히 그만큼 자원이 소모되며 속도,성능이 저하된다.
- ✔︎ 도커는 어플리케이션이 요구하는 환경✔︎을 만들어주는 것이지 가상의 컴퓨터 환경을 구축하는것이 아니다. -> vm보다 가볍고 뛰어난 성능

#### MSA / microservice architecture
- 앞서 살펴본 컨테이너별로 구동되는 application을 통합하여 하나의 대규모 시스템을 구축하는 것 
