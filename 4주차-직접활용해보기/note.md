#### 1) 해보고자 하는 것 : 간단한 내 스프링프로젝트를 도커파일을 통해 이미지 - 컨테이너로 실행해보기
- 이 때 나는 미리 jar파일을 만들어두는것이 아니라 도커파일을 통해 maven clean package까지 진행하도록 설정하여 jar파일을 생성한 후 이를 실행하도록 하고싶었다.
#### 2) 환경 : java11 & maven
#### 3) 구조 : 
![화면 캡처 2021-08-30 112903](https://user-images.githubusercontent.com/62214428/131276976-19d2e356-a814-4258-bd76-f31aff222ce9.png)
- 참고로 여기선 target에 jar가 이미 생성되어있는데 나는 이렇게 하지않고 dockefile을 통해 jar파일을 생성하는것까지 커버한다.

#### 4) 사용한 소스코드 : https://github.com/skarltjr/Springboot-jwt
- 해당 프로젝트를 클론받고 여기에 Dockerfile을 추가로 작성하고 이미지를 빌드
- 이 후 docker container run을 통해 실행해보기
- 그 다음 도커허브에 저장
- 다시 이미지를 내려받고 실행해보기


------------

#### 5) 과정 기록 :

##### 1. 해당 프로젝트를 클론받고 여기에 Dockerfile을 추가로 작성하고 이미지를 빌드
![화면 캡처 2021-08-30 114053](https://user-images.githubusercontent.com/62214428/131277688-d40f0f45-a8a8-41d3-82d6-610b35e2594c.png)
- 해당 부분에서는 3번 구조 그림에 맞춰 path를 잘 설정해야했다
- 이 후 도커파일을 통해 이미지 빌드
- ![화면 캡처 2021-08-30 114203](https://user-images.githubusercontent.com/62214428/131277774-ee7a9e7a-6927-45d3-a8c7-1745b796497e.png)

##### 2. 이 후 docker container run을 통해 실행해보기
- ![화면 캡처 2021-08-30 114307](https://user-images.githubusercontent.com/62214428/131277842-2a7db6d3-6700-47e0-aff0-cf249d0f465c.png)
- 더 아래 8080으로 실행된것 확인
- ![화면 캡처 2021-08-30 114344](https://user-images.githubusercontent.com/62214428/131277891-5d53661f-02d2-4334-948a-59c9089e7a5f.png)
- postman을 통해 동작하는 것 확인

##### 3. 이제 도커허브에 올리기
- 
