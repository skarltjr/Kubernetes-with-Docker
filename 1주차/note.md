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

   
