## 실제로 적용해보자
- https://github.com/skarltjr/Springboot-jwt 결과물
- https://www.baeldung.com/spring-boot-postgresql-docker


### 1. 우선 여기선 jar파일을 미리 만드는 쪽으로 
- jar파일을 생성한 후 src/main/docker에 cp
- Dockerfile / docker-compose.yml 모두 해당 디렉토리

### 2. docker-compose 파일이 있는데 왜 Dockerfile이 있나?
- 실제로 우리가 서비스를 만들 때 애플리케이션에 관한 이미지 어디서 받아오는게 아니라 만들어서 사용을 많이한다.
- 그래서 Dockerfile이 여기서도 존재
- 그럼 이 도커파일이 하는 /  여기 이 프로젝트에서의 역할을 생각해보자
### 2-1 생각 먼저 해보기
- `docker-compose.yml`을 통해서 하고자하는것은 여러 컨테이너를 한 번에 구축하고 서로 소통하도록 
- 그 컨테이너 중 하나가 바로 애플리케이션일 것 
- 해당 애플리케이션을 구축하기 위한 전용 이미지를 `Dockerfile`로 
- 그럼 웹 애플리케이션에 관한 dockerfile을 작성 -> 해당 도커 파일을 통해 여러 컨테이너를 다룰 yml 중 application 컨테이너를 구축하기 위한 이미지로 사용 

### 3. Dockerfile을 작성하자
```
FROM adoptopenjdk:11-jre-hotspot  # 자바 11을 사용하고 
ARG JAR_FILE=*.jar
COPY ${JAR_FILE} application.jar  
ENTRYPOINT ["java", "-jar", "application.jar"] # jar파일을 실행
```
- 잊지말 것!! src/main/docker에서 docker-compose up을 수행해야한다 / 왜냐? jar파일이 거기에 있으니까

### 4. docker-compose.yml작성
```
version: "3.9"
services:
  postgresdb:
    container_name: postgresdb
    image: postgres:latest
    ports:
    - "5432:5432"
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    restart: always

  application:
    container_name: jwt_app
    build: .
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgresdb:5432/postgres
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=password
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
    ports:
    - "8080:8080"
    restart: always
    depends_on:
      - postgresdb
```
- ★가장 고생한 부분 -> db connection 
- 우선 properties에 `spring.datasource.driverClassName=org.postgresql.Driver`를 통해 postgresql을 사용함을 알린다
- application.environment를 통해 db 연결 정보★를 작성하는데 잘 살펴봐야한다. 안그럼 연결안돼서 고생을 한다.


---------------
## 이 프로젝트를 이제 push

### 5. 해당 프로젝트를 clone하고 실행해보자
- -> git clone ~~
- -> src/main/docker로 가서
- -> sudo docker-compose up 
![화면 캡처 2021-09-15 203758](https://user-images.githubusercontent.com/62214428/133426567-0f05f034-f983-4477-90b4-00e208dacca7.png)
![화면 캡처 2021-09-15 203824](https://user-images.githubusercontent.com/62214428/133426583-3d0086b2-a07a-489b-a517-f74a1b82e159.png)
