# 7. Docker Compose
  ![화면 캡처 2021-09-15 143900](https://user-images.githubusercontent.com/62214428/133376872-22f8195a-378f-4074-b6fa-97ee49c10abb.png)
- ex) 

 
Python 기반의 웹 프레임워크 **django**, 관계형 DBMS **Postgresql**, Postgresql 모니터링 **pgAdmin** 이 세 가지를 모두 한꺼번에 컨테이너로 구성하려고 한다. 각각을 컨테이너로 구동해야 되니까... 이미지를 받아오고 `docker run` 을 실행하고...다시 또 실행하고...포트는 어떻게 연결하지? 세팅하려는 모든 컨테이너를 각각 구성하려니 명령어를 어떻게 실행해야 할지, 필요한 계정은 어떻게 설정해야 할지 하나도 감이 잡히지 않습니다. 

그래서 위와 같은 **멀티 컨테이너**를 구동하기 위한 구원투수가 나타났으니 그 이름이 바로 **Docker Compose** . Docker Compose 는 도커 컴포넌트 중의 하나로서, 여러 개의 컨테이너를 정의하고 실행하는 역할. 기존에 학습했던 이미지 빌드용 파일인 `Dockerfile` 과 더불어 `**docker-compose.yml**` 이라는 새로운 설정 파일이 등장하는데 사용법이 간결하고 직관적.

## 7.1 Docker Compose 설치 및 개요

### 7.1.1 Docker Compose 설치

`Ubuntu 가상머신이 아닌 Docker Desktop을 설치한 경우 이미 Docker Compose 가 내장되어 있어 별도로 설치할 필요 없다.`


**1) Docker Compose 다운로드**

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

**2) 실행 권한 부여**

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

**3) 버전 확인**

```bash
docker-compose --version
```


### 7.1.2 docker-compose.yml

`*.yml` 혹은 `*.yaml`  

이는 `YAML Ain't Markup Language` 라는 문구의 약자를 따서 만들어졌다고 합니다. 원래는 `Yet Another Markup Language` 이라는 이름이었으나 이후 의미가 변경되었습니다. `JSON` 이나 `XML` 같이 시스템 간 데이터 교환을 위해 만들어 졌으며, `key-value` 구조를 기본. 

문법적 중요사항은 다음과 같다.
- **대소문자를 구분합니다.**
- **구조를 구분할 때 들여쓰기로 탭 대신 스페이스를 사용합니다.**
- **값으로는 문자열(string), 숫자(number), 불린(boolean) 을 모두 취할 수 있습니다.**
- **`**:` 바로 뒤는 한 칸을 떼고 작성합니다.**
- **값을 나열하기 위해서는 `-` 를 입력한 후 한 칸을 떼고 사용합니다.**
- **주석 표기는 `#` 를 사용합니다.**


### 예시를 통해 구동해보고 알아보자
```yaml
version: "3.9"
services:
  db:
    image: postgres:alpine
    environment:
      - POSTGRES_PASSWORD=nextcloud
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
    restart: always
    volumes:
      - db_data:/var/lib/postgresql/data

  nc:
    image: nextcloud:apache
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_PASSWORD=nextcloud
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
    ports:
      - "80:80"
    restart: always
    volumes:
      - nc_data:/var/www/html
  depends_on:
      - db
volumes:
  nc_data:
  db_data: 
```
**1) yml파일 작성**
- ![화면 캡처 2021-09-15 151807](https://user-images.githubusercontent.com/62214428/133380701-c1826d62-b5ea-4cdd-9958-b91f7219ea92.png)
- 현재 `docker-compose`라는 폴더 안에서 yml작성


**2) `sudo docker-compose up`을 통해 구동**
- `localhost:80"으로 접속해보면 80포트로 띄운 nextcloud
- ![화면 캡처 2021-09-15 151916](https://user-images.githubusercontent.com/62214428/133380868-14cad942-a756-4caf-9071-c9169e86479a.png)


**3) `docker ps`로 구성한 두 개의 컨테이너가 동작하는것을 확인**
- 참고로 이 때 컨테이너가 폴더 이름을 따름

**4) `docker network ls`로 조회**
- 새로운 네트워크의 등장
- ![화면 캡처 2021-09-15 152215](https://user-images.githubusercontent.com/62214428/133381233-12eb30dc-005a-4c5f-8f0e-b167d9cde9a0.png)
- `docker compose`로 구동한 두 개의 컨테이너를 이어주는 네트워크

**5) `docker-compose down` - up을 통해 생성한 리소스 stop & remove**

**6) 그래서 만약 안쓰는 컴퓨터에 nextcloud를 이렇게 구축해놓으면 나만의 드라이브( 구글 드라이브, dropbox 같은..) 구축


## 7.2 Docker Compose 명령어

`명령어는 기본적으로 `docker-compose.yml` 파일이 위치한 곳의 경로에서 실행.`

**1) `up` - 멀티 컨테이너 생성 및 실행**

컨테이너의 이름은 별도로 설정하지 않으면 `[docker-compose.yml 파일이 위치한 디렉토리명]_[서비스명]_[번호]` 의 형태로 정의.
```
옵션
-d, --detach	컨테이너를 백그라운드에서 실행합니다.
—-build	컨테이너를 생성하기 전에 이미지를 빌드합니다.
—-no-build	실행 대상 이미지가 존재하지 않더라도 빌드하지 않습니다.
--abort-on-container-exit	여러 컨테이너들 중 하나라도 종료되면 모두 종료됩니다.
주의❗️—-detach 와 함께 사용할 수 없습니다.
```

**2) `ps` - 컨테이너 조회**

`docker ps` 혹은 `docker container ls` 를 사용해도 실행 중인 컨테이너를 조회할 수 있다. 다만, `docker-compose ps` 를 통해 조회했을 때의 출력 양식에서 약간 차이.

**3) `run` - 컨테이너 내부에서 명령 실행**

명령어를 사용할 때 주의할 점은 `run` 명령어 실행의 인수를 컨테이너명이 아닌 `docker-compose.yml` 에 정의된 서비스명으로 입력해야 하는 것.
- 위의 예시에선 서비스를 db / nc라는 이름으로 지정했다.
```bash
sudo docker-compose run [서비스명] [실행 대상 명령]
sudo docker-compose run db bash
```
**4) `start` - 생성되어 있는 컨테이너 실행**

```bash
sudo docker-compose start
```

**5) `stop` - 생성되어 있는 컨테이너 종료**

```bash
sudo docker-compose stop
```

**6) `down` - 컨테이너 종료 및 삭제**

```bash
sudo docker-compose down
```

## 7.3 docker-compose.yml 작성 - 요소를 하나씩 살펴보자
참고로 두 칸씩 띄운다 


```yaml
version: "3.9"
services:
  db:
    image: postgres:alpine
    environment:
      - POSTGRES_PASSWORD=nextcloud
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
    restart: always
    volumes:
      - db_data:/var/lib/postgresql/data

  nc:
    image: nextcloud:apache
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_PASSWORD=nextcloud
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
    ports:
      - "80:80"
    restart: always
    volumes:
      - nc_data:/var/www/html
    depends_on:
      - db
volumes:
  nc_data:
  db_data: 
```
**1) `version`**

```yaml
version: "3.9"
```

Docker Compose 파일은 최상단에 버전을 정의하도록 되어있다. 각 버전별로 명령어 혹은 표기법이 다르기 때문에 정상적으로 작동하지 않는 경우 버전을 체크해보아야 한다.

더불어 설치된 도커 엔진 버전과의 호환성도 반드시 확인. / https://docs.docker.com/compose/compose-file/compose-file-v3/

**2) `services`**

```yaml
services:
  db:
    ...
  nc:
    ...
```

서비스는 Compose 에서 실행할 컨테이너라고 생각. 각 서비스 별로 그에 맞는 환경의 컨테이너를 구성하기 위해 내부에 다양한 옵션을 추가. 혼동하지 말아야 하는 것은 서비스명과 컨테이너명은 다른 개념. 

만약 원하는 컨테이너명을 설정하고 싶다면 각 서비스 하위에 `container_name` 키와 설정하려는 값을 추가.

**3) `image`**

```yaml
services:
  db:
    image: postgres:alpine
      ...
  nc:
    image: nextcloud:apache
      ...
```

이미지는 말 그대로 컨테이너로 실행할 대상 이미지를 설정


**4) `build`**

```yaml
services:
  db:
    ...
  nc:
    build:
      context: .
      dockerfile: Dockerfile
      ...
```

이미 로컬에 이미지가 있거나 Docker Hub에 이미지가 있다면 이미지명과 태그만으로 쉽게 내려받아 컨테이너를 구성할 수 있다. 하지만 일반적으로 작성한 `Dockerfile` 에서 빌드된 이미지를 기반으로 컨테이너를 실행. 

Compose 에서는 실행할 이미지명 대신에 빌드할 `Dockerfile` 의 정보를 입력하여 이를 빌드하고 바로 이미지로 사용할 수 있다. `dockerfile` 옵션을 사용하면 파일의 이름이 `Dockerfile` 이 아닌 것도 빌드 대상으로 지정할 수 있으며 경로 역시 동일 경로가 아닌 다른 경로를 지정할 수 있다.

context . ==> 경로

**5) `command`**

```yaml
services:
  db:
    ...
  nc:
    build:
      context: .
      dockerfile: Dockerfile
    command: java -jar app.jar
```

생성된 컨테이너에 어떤 명령을 내릴지 세팅. 보통 컴파일러나 특정 언어로 작성된 어플리케이션을 명령어로 실행해야 하는 경우에 사용.

**6) `ports`**

```yaml
services:
  db:
    ...
  nc:
    ports: "80:80"
    ...
```

포트포워딩을 설정하는 항목으로 `docker run -p 80:80` 와 동일한 기능. 다만, yaml 파일에서는 `XX:YY` 의 형식이 시간값으로 해석될 수 있기 때문에 안전하게 따옴표 처리를 하시는 것을 권장.

**7) `depends_on`**

```yaml
services:
  db:
    ...
  nc:
    depends_on:
      - db
      ...
```

특정 서비스가 먼저 시작되면 이어서 시작할 수 있도록 설정하는 명령어. 위 예제 소스를 보면 `nc` 라는 서비스에 `db` 가 `depends_on` 으로 걸려있는데, 이는 `db` 서비스가 시작되면 `nc` 서비스가 시작되도록 순서를 정하는 것. 다만, `db` 가 완전히 초기화 되어 리스닝 상태까지 도달 했는지는 확인하지 않는다. 단순히 시작이 되었느냐, 아니냐 만을 가지고 서비스를 시작.

즉. 웹 서비스가 먼저 올라가고 db가 좀 늦게 올라간 상태에서 웹 서비스는 db에 연결을 하려고 시도할 것 

-> 이 때 웹 서비스가 연결을 할 수 없으니 fail날 것 

-> 그래서 restart always가 필요하고 

-> depends on이 필요

**8) `environment`**

```yaml
services:
  db:
    ...
  nc:
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_PASSWORD=nextcloud
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
      ...
```

환경변수를 설정하는 항목이며, DB 계정 및 초기 DB 세팅 등에 주로 사용. 이외 필요에 따라 각 컨테이너별 환경변수를 할당할 수 있다. `docker run -e` 와 유사한 기능

참고로 ★ services는 컨테이너단이라고 생각했을 때 db service의 env는 초기 db 세팅  / 여기서 nc service의 env는 nc 서비스에서 db에 연결하기 위한 정보

**9) `volumes`**

```yaml
services:
  db:
    volumes:
      - db_data:/var/lib/postgresql/data
		...
  nc:
    volumes:
      - nc_data:/var/www/html
		...
volumes:
  nc_data: 
  db_data:
...
```

볼륨을 세팅하는 항목이며, `docker run -v` 와 같은 역할. 컨테이너가 삭제되어도 데이터 유실이 되지 않도록 호스트의 일부 영역을 할당하게 되며, `[볼륨명]:[할당할 호스트 경로]` 를 작성.

더불어 `services` 와 같은 위계에 `volumes` 를 작성하고 그 하위에 서비스에 설정한 볼륨명을 작성.

**10) `restart`**

```yaml
services:
  db:
    ...
  nc:
    ...
    restart: always
		...
```

서비스 재시작을 설정할 수 있다. 기본값은 재시작을 하지 않는 것이지만 웹서비스의 경우 재시작을 `always` 로 설정하는 경우가 많다. 이는 `db` 가 완전히 리스닝이 가능한 상태가 되기 전에 웹 서비스에서 Connection을 생성하려는 경우 에러가 발생해 서비스가 비정상 종료가 되기 때문. 재시작을 하다가 `db` 가 Connection 생성이 가능한 상태가 되면 웹 서비스가 정상적으로 실행.

**11) `expose`**

```yaml
services:
  db:
    ...
    expose:
      - 5432
		...
  nc:
		...
```

`Dockerfile` 에도 `EXPOSE` 라는 명령어가 있었다. 실제로 포트포워딩을 수행하지 않고 어떤 포트를 개방해야 하는지 명시하는 역할을 했었는데 `docker-compose.yml` 에서는 조금 다른 역할을 하기 때문에 주의.

`expose` 에서 지정된 포트를 통해 통신을 가능케 하는 것은 맞다. 하지만, 호스트 OS 에서의 접근은 불가능하고 연결된 타 서비스(컨테이너)와의 통신만 가능. Compose 에서 연결된 서비스라는 것은  `docker-compose.yml` 에 작성된 `services` 하위의 항목. 이들은 동일한 네트워크 대역에 위치하므로 기본적으로 통신이 가능한 상태.
