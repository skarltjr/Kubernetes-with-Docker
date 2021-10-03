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
- 참고로 path 작성줄이기위해 dockerfile위치한곳에서 이미지 올렸음
- ![화면 캡처 2021-08-30 114924](https://user-images.githubusercontent.com/62214428/131278240-27a7d049-b140-4fa7-bdb8-801ba3e229ec.png)
- `sudo docker build -t <계정명>/<저장소명>:[태그명] .`
- 이미지 푸쉬
- ![화면 캡처 2021-08-30 115159](https://user-images.githubusercontent.com/62214428/131278430-c86952ff-e441-4d8f-a227-526dc940b7e6.png)

##### 4. 마지막으로 해당 이미지를 내려받고 실행확인해보기
- 우선 내 도커허브에 올려둔 이미지를 pull받고
- ![화면 캡처 2021-08-30 115557](https://user-images.githubusercontent.com/62214428/131278712-421f743c-961a-477e-ba5b-86df43f2d072.png)
- 이제 마지막으로 실행확인을 해보자
- ![화면 캡처 2021-08-30 115853](https://user-images.githubusercontent.com/62214428/131278996-ccf07f75-4a96-40f7-8658-56a747f3d271.png)

끝.


----------------

#### gradle
```docker
FROM openjdk:8-jdk-alpine as builder
# JDK 1.8 버전을 베이스로 설정한 후 builder로 alias 처리합니다.

COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .
COPY src src
# Spring Boot 프로젝트 내의 gradle 설정 파일과 소스코드를 이미지로 가져옵니다.

RUN chmod +x ./gradlew
RUN ./gradlew bootjar
# gradlew 에 실행권한을 부여하고 프로젝트를 jar 형식의 파일로 빌드합니다.

FROM openjdk:8-jdk-alpine
# 위에서 빌드한 jar 파일을 실행해 주기 위해 다시 JDK 1.8 버전을 베이스로 설정합니다.

COPY --from=builder build/libs/*.jar springboot-sample-app.jar
VOLUME /tmp
EXPOSE 8080
# builder를 통해 생성된 jar 파일을 이미지로 가져옵니다.
# 8080 포트를 공개한다고 명시합니다.

ENTRYPOINT ["java", "-jar", "/springboot-sample-app.jar"]
# 가져온 jar 파일을 실행시킵니다.
```
