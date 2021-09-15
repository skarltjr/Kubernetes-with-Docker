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


