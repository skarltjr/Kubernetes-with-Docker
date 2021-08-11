# 3. 도커 엔진

도커 엔진은 이미지 및 컨테이너를 생성하고 실행하는 코어 기능을 수행하는 컴포넌트. 흔히 우리가 도커라고 부르는 것은 '도커 엔진'.

## 3.1 도커 기본 명령

리눅스 환경에서 도커는 과거 TCP port 방식에서 UNIX Socket에 바인딩되는 방식으로 변경된 이후 root 권한을 필요로 합니다. 즉, 모든 docker 명령어 앞에 `sudo` 를 반드시 붙여야 한다는 뜻.
그러나 내 실습환경은 wsl2고 굳이 sudo없이도 동작은 하지만 sudo를 붙인다.

### 3.1.1 도커 버전 확인하기

```bash
$ sudo docker version
```
![화면 캡처 2021-08-11 204831](https://user-images.githubusercontent.com/62214428/129023713-f7149a17-c19d-4023-a99f-6f36cc5c55d1.png)


### 2.1.2 도커 시스템 정보 확인하기

```bash
$ sudo docker system info
```

![화면 캡처 2021-08-11 205023](https://user-images.githubusercontent.com/62214428/129023910-7ea7f8b8-ce8c-420c-bf77-e0aed57a7f63.png)

현재 제가 구동 중인 도커에는 컨테이너나 이미지가 없기 때문에 모두 0. 추후 실습을 진행하면서 어떤 변화가 있는지 살펴보기.

## 3.2 도커 컨테이너

컨테이너를 아는 것이 곧 도커를 아는 것이라 할 정도로 컨테이너는 매우 중요한 개념입니다. 하지만 중요하다고 해서 어렵게 받아들일 필요는 없습니다. 그 구조를 보면 오히려 우리가 사용하고 있는 PC보다 간단하기 때문입니다. 아래 다이어그램을 한번 보겠습니다.

![화면 캡처 2021-08-11 205721](https://user-images.githubusercontent.com/62214428/129024705-0000be58-9060-46c4-9389-4b6cd6c8e14a.png)

컨테이너는 도커가 설정한 네트워크 설정 및 저장장치에 따라 구성됩니다. 애플리케이션은 바로 이 가상의 공간에서 실행되죠. 컨테이너는 완벽하게 애플리케이션 실행 환경에 맞게 구성됩니다. 좀 더 멀리서 컨테이너를 바라보겠습니다.

![화면 캡처 2021-08-11 205748](https://user-images.githubusercontent.com/62214428/129024746-d830e922-d1a5-4168-8704-317b8cee8a45.png)

이 사각형 전체를 컴퓨터라고 생각해봅시다. CPU, 저장장치, LAN 카드, 그래픽 카드 등의 하드웨어가 컴퓨터를 물리적으로 구성하고 있습니다. 여기에 OS를 기반으로 하여 도커 엔진이 구동되고, 컨테이너는 바로 이 도커 엔진의 통제에 의해 움직입니다.

언뜻 보면 VM을 구동하는 것과 비슷해보이지만 컨테이너에는 기본적으로 OS가 없습니다. 애플리케이션을 구동하기 위해 필요한 라이브러리만을 개별적으로 가지고 있을 뿐입니다. OS를 실행할 필요가 없기 때문에 부하가 적고 구동되는 속도 역시 빠릅니다. 또한, 애플리케이션은 각자 독립된 영역을 보장받기 때문에 런타임에서 발생하는 충돌을 피할 수 있습니다.

### 2.2.1 컨테이너 실행하기

도커 엔진에 컨테이너를 실행

**1) 대화형 컨테이너 실행**

첫번째로 컨테이너에 올릴 시스템은 리눅스 베포판 중 하나인 CentOS 입니다.

```bash
sudo docker container run --interative --tty --name centos centos:latest
```

```bash
sudo docker container run -i -t --name centos centos:latest
```

```bash
sudo docker container run -it --name centos centos:latest          
```

⇒ 위 세 가지 명령은 모두 같은 의미이며, 축약형 옵션의 사용여부에만 차이 / 참고로 여기선 centos라는 이름의 컨테이너를 생성하는데 이 때 centos최신 이미지를 받아온다는 :latest 이 후 실행까지

위 명령어를 보면 대충 도커 컨테이너를 실행하는 것 같은데 수상한 옵션이 붙어있습니다. 이 두 가지 옵션은 컨테이너에 설치된 애플리케이션에 콘솔로 명령을 하기 위함이며, 자세한 의미는 다음과 같습니다.

> `--interactive, -i` 표준 입력창을 엽니다.
`--tty, -t` 장치에 tty를 할당합니다.
`--name` 컨테이너의 이름을 세팅합니다. (이 옵션을 사용하지 않을 경우 영어 단어를 임의로 조합하여 랜덤하게 세팅됩니다.)

위 두 옵션을 붙이지 않을 경우 터미널에서 명령어를 통한 컨테이너 제어가 불가능하므로 유의해야 합니다.

![화면 캡처 2021-08-11 210636](https://user-images.githubusercontent.com/62214428/129025744-a82b80e7-2332-4d57-9378-fedfc5ae5e64.png)

명령어를 통해 생성된 컨테이너에 centos 가 설치되었고, 이어 root 권한으로 원격 접속이 된 것 / 즉 컨테이너 내부에 접속 이 후 마치 대화하듯이 명령 그래서 대화형 컨테이너. 
 종료하려면 `exit` -> 해당 컨테이너도 종료 상태

**2) 백그라운드 컨테이너 실행**

두번째로 컨테이너에 올릴 대상은 웹서버인 httpd입니다.

```bash
sudo docker container run -d -p 80:80 --name apache httpd:latest
```

앞에서 봤던 것과 다른 옵션이 등장했습니다. 먼저 각각의 기능부터 살펴보겠습니다.

> `--detach, -d` 백그라운드에서 컨테이너를 실행합니다.
`--publish, -p` 호스트/컨테이너 포트포워딩을 세팅합니다.

명령어를 해석해보면,
컨테이너를 "백그라운드"에서 실행하고 "호스트의 80번 포트를 컨테이너의 80번 포트로 포워딩" 한다. 
실행 대상 이미지는 "httpd"의 "최신 버전"이고 컨테이너의 이름은 "apache"로 붙인다.
라고 할 수 있겠군요.

![화면 캡처 2021-08-11 211654](https://user-images.githubusercontent.com/62214428/129027090-b7967bc8-e536-4b5a-aca5-dcdfb6f4d1a3.png)

왼쪽이 host의 포트 : 오른쪽이 컨테이너의 포트
브라우저를 열고 `http://(컨테이너 IP):80` 를 입력하면 Apache 서버가 설치된 컨테이너의 80번 포트로 포워딩 되어 다음과 같이 정상적으로 접속이 됨을 확인할 수 있습니다.

![화면 캡처 2021-08-11 211737](https://user-images.githubusercontent.com/62214428/129027169-3ba8ff29-385a-400b-98b8-605c28a15ef2.png)
![화면 캡처 2021-08-11 211719](https://user-images.githubusercontent.com/62214428/129027174-bc422222-86aa-4596-99e4-220b00b8528c.png)

컨테이너 내부에서 어떤 일이 일어나는지는 다음과 같이 로그를 조회하는 명령어로 확인이 가능합니다.

```bash
sudo docker container logs apache
```

`logs` 뒤에 로그를 조회하고자 하는 컨테이너 명을 입력하면 됩니다. 로그는 다음과 표시됩니다.

![화면 캡처 2021-08-11 212007](https://user-images.githubusercontent.com/62214428/129027434-255094e3-6d4c-445f-95fc-156234b1684a.png)

### 3.2.3 컨테이너 생명 주기

'생명 주기'라는 용어는 프로그래밍 학습에서 자주 등장합니다. 가장 친숙한 예를 들어보자면 Java 쓰레드의 생명 주기가 있겠네요. 도커에서 제어하는 컨테이너도 이러한 생명 주기를 갖습니다. 다음의 그림은 컨테이너의 생명 주기를 나타냅니다.

![화면 캡처 2021-08-11 212827](https://user-images.githubusercontent.com/62214428/129028592-cfc215cf-3a4e-4933-887d-377c16e8d7cb.png)

우리가 앞에서 살펴본 명령어인 `run` 이 보입니다. 컨테이너를 생성함과 동시에 실행을 수행했었죠. `create` 는 컨테이너를 생성만 하고 시작은 하지 않습니다. 시작을 위해서는 별도로 `start` 명령을 내려야합니다. `rm` 은 컨테이너를 삭제하는 명령어로서, 정지된 컨테이너에 한해 삭제가 가능합니다. 만약 실행 중인 컨테이너를 삭제하고자 한다면 강제 삭제를 위한 옵션을 추가로 입력해야 합니다.

아래 이어서 컨테이너를 다루는 명령어를 살펴보겠습니다.

### 3.2.4 컨테이너 명령

컨테이너 명령어를 실습하기 전에 각각의 옵션 명세를 먼저 전달드립니다. 대표적으로 많이 쓰이는 것들 위주로 정리하였으며, 전체 옵션에 대해서는 도커 공식 문서 링크를 남겨드립니다.

- 도커 공식 문서

    [https://docs.docker.com/engine/reference/run/](https://docs.docker.com/engine/reference/run/)

**1) `run` - 컨테이너 생성 & 실행**

```bash
sudo docker container run -d -p 80:80 --name apache httpd:latest
참고로 run은 이미지까지 받아서 실행시키는 것 / 따라서 만약 이미지가 존재하고 컨테이너 중지된 상태에선 당연히 run x / restart
```

**2) `stop` - 컨테이너 중지**

```bash
sudo docker container stop apache
```

**3) `start` - 컨테이너 시작**

- 옵션
--detach, -d	컨테이너 생성 후 백그라운드에서 실행한다.
--interactive , -i	표준 입력창을 엽니다.

```bash
sudo docker container start apache
```

**4) `restart` - 컨테이너 재시작**

```bash
sudo docker container restart apache
```

**5) `stats` - 컨테이너 구동 확인**

- 옵션
--all, -a	실행/중지 된 모든 컨테이너의 상태를 확인합니다.
--format	출력 포맷을 설정합니다.

```bash
sudo docker container stats apache
```

```bash
sudo docker container stats apache --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

**6) `ls` - 컨테이너 목록 조회**

```bash
sudo docker container ls
```

**7) `attach` - 구동 중인 컨테이너 연결**
- 백그라운드로 현재 apache서버를 구동 중 
```bash
sudo docker container attach apache
```
- 현재 백그라운드로 구동중인 컨테이너에 붙을 수 있는데
- attach 실행 후 다시 접속해보면 로그가 프롬프트에 남는것을 확인할 수 있다
![화면 캡처 2021-08-11 214913](https://user-images.githubusercontent.com/62214428/129031412-4bda84d9-862f-46ea-87e7-e4edfe530ac6.png)
- 주의할 점 : 여기서 ctrl+c로 종료를 하는데 이 때 !! 컨테이너도 함께 종료된다. 이 점을 고려하고 행동할 것 


**8) `exec` - 구동 중인 컨테이너에서 프로세스 실행**

- 옵션
- --detach, -d	컨테이너 생성 후 백그라운드에서 실행한다.
- --interactive , -i	표준 입력창을 엽니다.
- --tty , -t	장치에 tty를 할당합니다.
- --user, -u	컨테이너 실행 시, 사용자명이나  지정합니다.

- 현재 apache 컨테이너가 동작 중 
```bash
sudo docker container exec -it apache /bin/echo "Hello, Docker!"
```
![화면 캡처 2021-08-11 215521](https://user-images.githubusercontent.com/62214428/129032290-08cebe60-3f05-4c74-bf38-dbddf9d6edbd.png)

```bash
sudo docker container exec -it apache bash
```
![화면 캡처 2021-08-11 215632](https://user-images.githubusercontent.com/62214428/129032482-90d986d2-7022-40e0-97c0-384f4ce50252.png)
- 즉 외부에서 컨테이너 내부 프로세스를 실행하는 방법

**9) `top` - 컨테이너 내부에서 구동 중인 프로세스 확인**
- 현재 apache컨테이너가 백그라운드로 실행 중
- 백그라운드로 실행되기 때문에 외부에선 어떻게 진행되는지 모른다
- 그래서 내부에서 실행중인 프로세스를 확인하고자 할 때 
```bash
sudo docker container top apache
```
- ![화면 캡처 2021-08-11 220407](https://user-images.githubusercontent.com/62214428/129033634-18e1227a-dc64-48d7-92df-6dbb06273f34.png)
- 보면 아 이 컨테이너 내부에서 httpd라는 웹 서버가 돌고 있구나를 알 수 있다.
**10) `rename` - 컨테이너 이름 변경** 

```bash
sudo docker rename apache apache_server
```

**11) `cp` - 컨테이너 내부 파일 복사**

```bash
sudo docker container cp apache:/usr/local/apache2/htdocs/index.html /tmp/index.html
```

```bash
sudo docker container cp /tmp/index.html apache:/usr/local/apache2/htdocs/index.html
```

**12) `diff` - 컨테이너 변경 사항 확인**

```bash
sudo docker diff apache
```
