# 4. 도커 이미지

도커 이미지는 애플리케이션 자신 뿐만아니라 실행에 필요한 모든 것들을 담고 있다. OS, DBMS, 웹서버 등 시스템 상에서 이용되는 대부분의 요소를 이미지로 생성할 수 있다.

## 4.1 도커 이미지 개요
```bash
sudo docker container run -it --name ubuntu ubuntu:latest
```
- 기본적으로 로컬에는 필요한 도커 이미지가 존재하지 않는다.
- 여기서 마지막 ubuntu를 보자
- ![화면 캡처 2021-08-18 213458](https://user-images.githubusercontent.com/62214428/129899063-4a6c1c76-5225-4842-ae12-97adcf962540.png)
- 로컬에 이미지가 존재하지 않는 경우 docker hub서 이미지를 받아온다.
- 그러나 컨테이너를 실행할 때 단일 이미지만을 이용하는 경우는 거의 없다. 겉으로 보기에는 단일 이미지이지만 내부를 살펴보면 다중 이미지로 구성. 결국 실제 운영환경에서 실행되는 이미지는 베이스가 되는 이미지와 설정 파일, 그리고 명령들을 한 데 모아 빌드한 것이고, 그렇게 빌드된 이미지로 컨테이너를 실행. 여기서 등장하는 것이 바로 Dockerfile. 

## 4.2 도커 이미지 관리


**1) `pull` - 도커 이미지 받기**

```bash
sudo docker image pull ubuntu
sudo docker image pull ubuntu:focal
sudo docker image pull ubuntu:latest

sudo docker image pull 이미지명[:태그]
```

이미지를 가져올 때는 저장소명(repository)을 사용하며, 필요에 따라 **태그**를 추가로 작성. 태그는 저장소에 저장된 이미지를 버전별로 관리하는 데에 사용.
![화면 캡처 2021-08-18 215603](https://user-images.githubusercontent.com/62214428/129901862-4d46f3a8-62d4-404c-bad9-e98d11a3d11a.png)
위에 작성한 세 줄의 명령어는 사실 모두 같은 이미지를 가리키고 있다. 태그를 명시하지 않은 첫번째 명령은 `latest` 태그가 붙은 것과 같은 의미. 태그의 경우 복수로 설정이 가능하기 때문에 보통 릴리즈 연도, 숫자형식의 버전, 영문 별칭 등이 다양하게 활용.

여러 태그 중 가장 중요한 것은 `latest`. 일반적으로 `latest` 태그는 장기 지원 버전(LTS)이나 안정화(stable) 버전에 붙인다. Ubuntu의 경우 20.04 버전이 가장 최신의 장기 지원 버전이기 때문에 `latest` 태그가 붙어있는 것. 이외 버전을 설치해야 하는 경우 반드시 태그를 명시.

![화면 캡처 2021-08-18 215700](https://user-images.githubusercontent.com/62214428/129901978-fa1366c1-f84f-461e-abda-6f4b0c63fddf.png)
이미지를 받아오는 과정을 들여다 보면 다이제스트라는 긴 문자열이 등장. 이는 Docker Hub에서 이미지를 식별하기 위한 고유한 값. 같은 이미지는 같은 다이제스트

```bash
**SHA256 - Secure Hash Algorithm**
해싱 알고리즘 중 하나인 SHA256은 데이터를 암호화 하는 데에 사용.
256이라는 숫자는 총 256bit의 길이로 해시 값을 이룬다는 의미이며, $2^{256}$의 경우의 수가 존재.
```

**2) `ls` - 이미지 목록 조회**

- 옵션
    - --all, -a	숨겨진 중간 단계 이미지까지 모두 조회합니다.
    - --format	출력 포맷을 설정합니다.
    - —-digests	다이제스트를 포함하여 이미지를 조회합니다.
    - —-quiet, -q	이미지 ID만 조회합니다.
```bash
sudo docker image ls
```
![화면 캡처 2021-08-18 223857](https://user-images.githubusercontent.com/62214428/129908215-178b950c-5e93-46e7-99c2-929dff7c15aa.png)
20.04는 현재 latest버전 따라서 latest 이미지와 이미지ID가 동일 ★ 그러나 18.04 이미지와는 id가 다른걸 볼 수 있다. 즉 그냥 서로 다른 이미지다.

**3) `inspect` - 이미지 정보 조회**

```bash
sudo docker image inspect ubuntu:18.04
```
![화면 캡처 2021-08-18 223908](https://user-images.githubusercontent.com/62214428/129908452-932302d5-7861-4ced-891e-e8851bc351eb.png)

이미지 정보를 조회해보면 굉장히 많은 내용이 JSON 형식으로 출력. 이 모든 내용에서 사용자가 원하는 내용만 조회하기 위해서는 다음과 같이 출력 포맷을 설정해주면 된다.

```bash
아래는 여러 정보 중 RepoTags 파트만 출력하고자 
sudo docker image inspect --format="{{ .RepoTags }}" ubuntu:18.04
```

**4) `tag` - 이미지에 태그 설정**

```bash
sudo docker image tag ubuntu:18.04 ggingmin/ubuntuos:1.0

sudo docker image tag 대상이미지명[:태그] [사용자명/]이미지명[:태그]
```
![화면 캡처 2021-08-18 224346](https://user-images.githubusercontent.com/62214428/129908983-5dfa3c42-e983-41be-a301-171c10107c68.png)
★ 기존 ubuntu:latest에 새로운 이름을 붙여준 것 

태그를 설정한다는 것은 현재 도커 엔진에 저장된 이미지의 복사본을 만들어 새로운 이미지명과 태그를 붙인다는 뜻. 실행 결과를 조회해보면 두 이미지의 ID가 같다(내가 만든 것과 latest). 왜 굳이 실체가 같은 이미지를 복잡하게 다른 이름을 붙여 관리할까?

우리가 생성한 도커 이미지는 Docker Hub를 비롯한 다양한 클라우드 사업자의 레지스트리에 저장할 수 있다. 하지만 각 클라우드 사업자 별로 이미지를 업로드할 때 사용되는 이미지명의 규칙이 달라 이를 맞춰주어야 한다.

**5) `rm` - 이미지 삭제, 이미지 태그 해제**


```bash
sudo docker image rm ggingmin/ubuntuos:1.0
sudo docker image rm ubuntu:18.04

sudo docker image rm 이미지명[:태그]
sudo docker image rm 이미지ID
```
★ 앞에서 말했듯이 복사본을 만들어서 새로운 이미지명과 태그를 붙인것
따라서 복사본은 삭제해도 원본이 삭제되는것이 아니다.
![화면 캡처 2021-08-18 224749](https://user-images.githubusercontent.com/62214428/129909666-26d15b31-59fa-4df2-9a90-91c92f84a7e2.png)

**6) `container commit` - 실행 중인 컨테이너로부터 이미지 생성**

- 옵션
   - —-author , -a	이미지 작성자를 등록합니다.
   - —-message, -m	커밋 메세지를 등록합니다.

```bash
sudo docker container run -d -p 80:80 --name apache httpd # 사전에 실행
sudo docker container commit -a "ggingmin" apache ggingmin/apacheweb:1.0
 -a [사용자명] [컨테이너명] [생성할 이미지명]
```
![화면 캡처 2021-08-18 225225](https://user-images.githubusercontent.com/62214428/129910485-6961a1af-ff3e-4f37-8368-4d9487b9020b.png)
이미지를 생성하는 명령어이지만 특이하게 `container` 명령을 사용.
`commit`은 실행 중인 컨테이너에 수행되는 명령으로서, 컨테이너를 실행하는데에 사용했던 이미지와는 다른 이미지를 만들어 낸다.
컨테이너의 당시 상태를 그대로 본뜬 스냅샷을 이미지로 만들었기 때문.
![화면 캡처 2021-08-18 225306](https://user-images.githubusercontent.com/62214428/129910596-e614c2dd-f580-4b70-9265-39dd4db489a6.png)

**7) `container export` - 실행 중인 컨테이너로부터 파일 생성**

```bash
sudo docker container export apache > apache.tar

sudo docker container export 컨테이너명 > [경로/]파일명
위는 경로지정을 안했기때문에 루트에 저장된다.
```

`export` 역시 `container` 와 함께 사용되는 명령어. 
현재 컨테이너를 본떠 파일로 내보내는 역할을 하고 리눅스에서 많이 사용되는 파일 압축 형식인 `.tar` 를 사용. 이렇게 생성된 파일은 import 명령어를 사용하여 이미지를 생성할 수 있다.

**8) `import` - 파일을 이미지로 생성**

```bash
sudo docker image import apache.tar kiseok/kiseok-apache2:1.0
```
![화면 캡처 2021-08-18 225645](https://user-images.githubusercontent.com/62214428/129911259-b58ac686-3e71-4c3f-bf66-8410682152a3.png)

## 4.3 Dockerfile
![화면 캡처 2021-08-18 233801](https://user-images.githubusercontent.com/62214428/129918197-49bc83bf-f2fd-494e-8d74-7d53aae76cff.png)
Dockerfile은 새로운 이미지를 빌드하는 데에 필요한 이미지와 설정들을 작성한 파일...! 어떤 이미지를 사용할지, 어떤 포트를 열어놓을지 등에 대해서 정의가 완료되면 이 내용을 Dockerfile의 양식에 맞게 작성하기만 하면 된다. 
```docker
FROM ubuntu:18.04

RUN apt-get -y update && apt-get -y upgrade
RUN apt-get -y install nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

위 명령은 ubuntu에 nginx 웹서버를 설치하고 80 포트를 열어놓는 내용을 담고 있다. 이 파일의 내용을 기반으로 빌드가 진행되면 그 결과물로 새로운 이미지가 나오게 된다. Dockerfile로 인프라를 사전에 빌드해 놓는다면 어플리케이션 개발자는 이를 받아서 컨테이너를 실행하기만 하면 되니 매우 간편

![화면 캡처 2021-08-18 233935](https://user-images.githubusercontent.com/62214428/129918496-12a2c504-2dfd-403e-992a-8c49b1b27bac.png)
FROM 으로 시작되는 명령어에서는 베이스 이미지를 설정. 이후 실행되는 명령은 모두 이 베이스 이미지를 기반으로 실행.베이스 이미지는 보통 OS와 관련

![화면 캡처 2021-08-18 233944](https://user-images.githubusercontent.com/62214428/129918659-1bd4dc9d-bb13-492d-8a3b-6e808b8f8bd9.png)
각각의 명령줄은 하나의 이미지 레이어를 구성. 즉, 각 명령 스텝별로 이미지가 생성되는 것. 이 과정에서 생성되는 중간 이미지는 캐싱되어 다른 이미지를 생성할 때도 사용. 

Dockerfile에서 사용할 수 있는 명령어는 용도에 따라 아주 다양합니다. 작성 시 대소문자의 구분은 없으나 통상 대문자로 작성하게 됩니다. 그러면 위의 Dockerfile이 각각 어떤 작업을 수행하는지 나눠서 보겠습니다.

```docker
FROM ubuntu:18.04
```

FROM 절은 베이스 이미지를 세팅. 빌드할 이미지의 베이스를 'ubuntu'로 설정한다는 의미. 여기까지만 본다면 ubuntu의 이미지를 그대로 복제한 것과 같다.

```docker
RUN apt-get -y update && apt-get -y upgrade
RUN apt-get -y install nginx
참고로 여기서 RUN은 이미지를 만드는 명령어 / 즉 컨테이너 생성 후 run과는 관계없는 것 / 이미지를 만드는 한 단계 한 단계를 위한 명령어
```

RUN 절은 이미지를 생성에 필요한 미들웨어나 어플리케이션을 세팅하기 위한 명령을 실행. 여기서는 `apt-get` 명령어를 통해 'nginx'를 설치. 

```docker
EXPOSE 80
```

'nginx' 웹서버를 통해 외부와 통신하기 위해서는 기본적으로 포트가 열려있어야 한다. HTTP 방식의 통신을 위해 '80'번 포트를 열어주었다. 포트는 역할별로 지정되어 있는 것들이 있는데, 이는 하나의 약속으로 이메일을 송수신 하거나 서버 접속방식 등에 포트가 할당 되어있습니다.

```docker
CMD ["nginx", "-g", "daemon off;"]
```

nginx 설치 및 포트설정이 완료된 후에는 웹서버를 구동. CMD 명령은 생성된 이미지를 기반으로 구동된 컨테이너에서 명령을 실행하는 것이며, 하나의 Dockerfile에서 한 번의 명령만 유효. 복수의 CMD 명령이 있을 경우 마지막 CMD 명령만 실행.

여기까지 웹서버 구축을 위한 Dockerfile 예시 /  실제로 컨테이너를 띄우려면 빌드의 과정이 필요

즉 도커파일을 작성한 상태 -> 이제 이 도커파일을 바탕으로 이미지를 만드는 "빌드"할 차례

### 2.3.3 Dockerfile 빌드

'빌드' 라는 용어는 프로그래밍 전방위에서 접하게 됩니다. '모바일 앱을 빌드한다.', '스프링 프로젝트를 빌드한다.' 등등 소스 코드를 컴파일 한다는 의미로 주로 사용되죠. 도커에서 빌드는 Dockerfile을 기반으로 이미지를 생성한다는 의미를 가지고 있습니다. 위에서 작성한 Dockerfile을 기반으로 이미지를 만들어 봅시다.

```bash
mkdir docker && cd docker
touch Dockerfile
vi Dockerfile
```

Dockerfile을 생성한 후, nano 혹은 vi를 통해 위에서 확인했던 명령어를 작성.

```bash
sudo docker build -t sample:1.0 /home/(USER)/docker
```
![화면 캡처 2021-08-18 234634](https://user-images.githubusercontent.com/62214428/129919711-7ce5e164-f472-42a2-853f-216dfc055fa5.png)

`sudo docker build -t` 는 도커 이미지를 생성하기 위한 기본 명령어 입니다. `sample:1.0` 은 태그라고 부르는데 이름표를 붙인다고 생각하시면 되고 `[이미지명]:[버전]` 의 형태로 작성합니다. 마지막 부분에는 경로를 입력하며, Dockerfile이 있는 경로로 이동해서 현재 경로를 나타내는 `.` 를 사용하거나 절대 경로, 상대 경로를 입력하는 것 모두 가능합니다.


## 4.4 Dockerfile 명령어
- Dockerfile 명령어
    - `FROM`	베이스 이미지를 할당합니다.
    - `RUN`	이미지를 생성하기 위한 명령을 실행합니다.
    - `CMD`	컨테이너 내부에서 명령을 실행합니다.
    - `LABEL`	라벨을 설정합니다.
    - `EXPOSE`	포트를 할당합니다.
    - `ENV`	환경변수를 설정합니다.
    - `ADD`	파일 혹은 디렉토리를 추가합니다.
    - `COPY`	파일을 복사합니다.
    - `ENTRYPOINT`	컨테이너 내부에서 명령을 실행합니다.
    - `VOLUME`	볼륨을 마운트합니다.
    - `USER`	특정 사용자를 지정합니다.
    - `WORKDIR`	작업할 디렉토리를 세팅합니다.
    - `ARG`	Dockerfile 내부의 변수를 설정합니다.
    - `ONBUILD`	빌드가 완료된 후 명령을 실행합니다.
    - `STOPSIGNAL`	시스템 콜 시그널을 설정합니다.
    - `HEALTHCHECK`	컨테이너의 상태를 체크합니다.
    - `SHELL`	컨테이너에서 사용할 기본 쉘을 설정합니다.

**) 전반적인 과정을 먼저 살펴보면 
- 1) dockerfile 작성 
    - <img width="837" alt="Screen Shot 2021-08-19 at 9 29 01 AM" src="https://user-images.githubusercontent.com/62214428/129989046-16b0b54c-16d2-41c0-b151-e7d201084daa.png">
- 2) 작성한 도커파일을 기반으로 이미지 생성
    - <img width="891" alt="Screen Shot 2021-08-19 at 9 29 58 AM" src="https://user-images.githubusercontent.com/62214428/129989111-120669c0-7be5-4674-9b0c-a86f5f3c552b.png">
- 3) 생성한 이미지를 통해 run
    - <img width="649" alt="Screen Shot 2021-08-19 at 9 33 12 AM" src="https://user-images.githubusercontent.com/62214428/129989385-96aa8275-f803-4f32-9fc4-a3a99acd07ef.png">
    - <img width="2560" alt="Screen Shot 2021-08-19 at 9 33 39 AM" src="https://user-images.githubusercontent.com/62214428/129989449-205b409a-a9e5-47cf-ab95-91ab8307f01a.png">




----------


**1) `FROM` - 베이스 이미지 설정**

```docker
FROM ubuntu:bionic // 18.04
```
베이스 이미지를 세팅하는 명령으로서 기본적으로 Docker Hub에서 이미지를 탐색해 빌드에 사용. 하지만 상황에 따라 사용자가 직접 만든 베이스 이미지를 통해 빌드하는 경우도 있다.

**2) `RUN` - 이미지를 빌드할 때 실행할 명령 설정**
- 참고로 다시 말하지만 여기서 RUN은 이미지를 빌드할 때 !! 컨테이너가 생성되기 전!!

```docker
RUN apt-get -y update                          # Shell 형식
RUN ["/bin/bash", "-c", "apt-get -y update"]   # Exec 형식
```
`RUN` 을 통해 작동하는 명령은 아직 컨테이너가 작동하는 상태가 아니다. 베이스 이미지에 추가적으로 명령을 실행해 필요한 패키지나 미들웨어를 설치하기 위해 많이 사용.

명령은 위 예시처럼 Shell 형식, 혹은 Exec 형식으로 작성할 수 있다. Shell 형식에서 `RUN` 뒤에 바로 명령어를 작성하는 것은 `/bin/sh -c` 이 붙어있는 것과 마찬가지라고 이해.

두 가지가 혼용되는 경우가 많지만 실행할 셸 혹은 프로그램을 지정한다는 측면에서 Exec 형식이 권장.

**3) `CMD` - 이미지를 통해 생성된 컨테이너 내부에서 실행되는 명령**
- RUN이 이미지를 빌드할 때 ! 라면 cmd는 컨테이너가 생성된 후 ! 컨테이너 내부에서 실행될 명령어

```docker
FROM ubuntu:18.04

CMD echo "Hello, Docker!"             # Shell 형식
CMD ["/bin/echo", "Hello, Docker!"]   # Exec 형식
```

컨테이너는 `docker container run` 명령어를 통해 실행됩니다. 이렇게 컨테이너가 실행될 때 수행할 명령은 `CMD` 를 통해 세팅이 가능. 유의할 점은 Dockerfile 전체에서 `CMD` 명령은 단 하나만 유효하고, 여러 개의 명령이 있다면 마지막 것만 실행이 된다는 점.

`CMD` 명령 역시 `RUN` 과 같이 Shell, Exec 형식 모두 사용 가능.

**4) `ENTRYPOINT` - 이미지를 통해 생성된 컨테이너 내부에서 실행되는 명령**
- CMD와 역할은 같지만 동작방식!!!이 다르다 
- cmd는 변경될 수 있는 사항에 적용 // 컨테이너가 생성되고 변경할 수 있는 내용에 사용
- entrypoint는 무조건 이대로 동작해야하는 경우 // 디폴트를 설정할 때 


```docker
FROM ubuntu:18.04

ENTRYPOINT echo "Hello, Docker!"             # Shell 형식
ENTRYPOINT ["/bin/echo", "Hello, Docker!"]   # Exec 형식
```

뭔가 이상한데... 라는 생각과 함께 위에 `CMD` 명령을 다시 올려다보셨을 겁니다. `ENTRYPOINT` 의 역할이 `CMD` 와 같게 쓰여져 있기 때문이죠. 실제로 이 둘은 컨테이너가 실행된 후 그 내부에서 명령을 실행한다는 동일한 기능을 가지고 있습니다. 하지만 이 둘 사이에는 중요한 차이점이 있습니다. 다음의 명령을 함께 보겠습니다.

```docker
FROM ubuntu:18.04

ENTRYPOINT ["/bin/echo"]
CMD ["Hello, Docker!"]
```

```bash
sudo docker image build -t hellodocker .
sudo docker container run -it hellodocker
sudo docker container run -it hellodocker 'Hi, Docker!'
```

<img width="785" alt="Screen Shot 2021-08-19 at 9 25 02 AM" src="https://user-images.githubusercontent.com/62214428/129988786-ac9d3cba-4a13-4617-9227-e111634d6890.png">

`ENTRYPOINT` 의 명령은 사용자가 어떤 인수를 명령으로 넘기더라도 Dockerfile에 명시된 명령을 그대로 실행. 반면 `CMD` 는 컨테이너를 실행할 때 사용자가 인수를 넘기면 기존에 작성된 내용을 덮어 쓴다. 이러한 특성을 이해한다면 좀 더 효율적인 이미지 생성이 가능.

- 여기서 `ENTRYPOINT ["/bin/echo"]`는 무조건 수행하려고 하는 명령어  
- 반면 `CMD ["Hello, Docker!"]`는 `sudo docker container run -it hellodocker 'Hi, Docker!'`처럼 변경해도 되는 경우에 적용

**5) `ONBUILD` - 이미지 빌드가 완료된 후에 실행되는 명령**

- 1) 여기선 우선 base (parent) 이미지를 생성 Dockerfile
- <img width="887" alt="Screen Shot 2021-08-19 at 1 36 14 PM" src="https://user-images.githubusercontent.com/62214428/130008772-b733e30a-2e7d-4fcd-baa2-6605a528fc75.png">
- <img width="771" alt="Screen Shot 2021-08-19 at 1 51 32 PM" src="https://user-images.githubusercontent.com/62214428/130009799-435565ff-cc76-4887-a60e-58cc9b9a28ef.png">



- 2) 자식 이미지를 생성 : 이 때 자식 도커파일 Dockerfile.child의 베이스를 parent 이미지 ( base:1.0 )으로  Dockerfile.child
- <img width="262" alt="Screen Shot 2021-08-19 at 1 45 37 PM" src="https://user-images.githubusercontent.com/62214428/130009255-b4fbe887-78a7-4f3b-a284-7588cab2d0f2.png">
- <img width="624" alt="Screen Shot 2021-08-19 at 1 54 06 PM" src="https://user-images.githubusercontent.com/62214428/130010034-98488502-e481-48c6-adf0-06a10822fc8a.png">


**6) `HEALTHCHECK` - 컨테이너의 작동상태 체크**

- 옵션
--interval	컨테이너 체크 간격을 설정합니다.
--timeout	설정한 시간에 정상작동 하지 않으면 타임아웃 처리합니다.
--retries	재시도 횟수를 설정합니다.
```docker
# ./docker/Dockerfile
FROM httpd

RUN ["apt-get", "update"]
RUN ["apt-get", "-y", "install", "curl"]

HEALTHCHECK --interval=3s --timeout=5s --retries=3 CMD curl --fail http://localhost:80/ || exit 1
```

```bash
sudo docker image build -t web-healthcheck .
```

```bash
sudo docker container run -d -p 80:80 --name=apache-hc web-healthcheck
```
컨테이너를 작동시킬 때, 이 컨테이너가 정상적으로 작동하고 있는지 체크를 해야하는 경우가 있습니다. Dockerfile을 작성할 때 `HEALTHCHECK` 명령을 추가하면 이를 컨테이너 내부에 로그로 남길 수 있습니다. 컨테이너에서 이 명령어가 잘 작동하는지는 `docker ps` 명령어를 통해 `STATUS` 항목을 살펴보면 됩니다. 구동 시간 옆에 (healthy) 라는 문구가 보입니다. 더 정확하게 로그를 보려면 `docker container inspect <컨테이너명>` 을 통해 "Health" 에 해당하는 값을 확인하면 됩니다.

**7) `ENV` - 환경변수 설정**

**8) `WORKDIR` - 작업 디렉토리 할당**

![화면 캡처 2021-08-19 223212](https://user-images.githubusercontent.com/62214428/130077496-16c84989-f12e-4ecb-a57a-8723b371f65e.png)

- 순서대로 살펴보면 `ENV DIRPARENT /parent`를 보면 /루트하위 parent폴더를 지정 DIRPARENT란 이름으로  
- 그럼 현재 루트하위 parent폴더 내부 
- `ENV DIRCHILD child`으로 parent폴더 내부에 child폴더 지정
- `WORKDIR $DIRPARENT/$DIRCHILD` -> $(환경변수)를 통해 작업할 디렉토리를 직접 지정 -> 여기선 그럼 루트 하위 parent/child 내부에서 동작
- expect 컨테이너 실행해보면 parent/child폴더 내부일것
- ![화면 캡처 2021-08-19 223942](https://user-images.githubusercontent.com/62214428/130078691-44b53dbe-b176-47f6-8af3-4dae81dda48c.png)
- 추가로 연습한 내용은 child 내부에 hello.txt파일 생성까지
- ![화면 캡처 2021-08-19 224323](https://user-images.githubusercontent.com/62214428/130079272-a74639bc-8fdc-4b4d-aa71-28e99230dd78.png)
- ![화면 캡처 2021-08-19 224307](https://user-images.githubusercontent.com/62214428/130079290-d38e39fd-6f05-47b4-baeb-cbd7065d4607.png)

환경변수를 설정하고 이를 이용해 작업 경로를 세팅하였는데, `ENV` 에서 설정된 환경변수는 `RUN`, `CMD`, `ENTRYPOINT` 명령에서 모두 사용 가능. 선언된 환경변수를 사용하기 위해서는 `$` 

`WORKDIR` 는 리눅스의 `cd` 와 유사. Dockerfile 내부에서 경로를 이동할 때도 쓰일 뿐만 아니라, 이렇게 빌드된 이미지를 대화형 컨테이너로 실행할 때 프롬프트의 최초 위치를 결정.

**9) `USER` - 유저 할당**

**10) `LABEL` - 이미지 버전 정보, 작성자 등 레이블 정보 등록**

**11) `ARG` - Dockerfile 내부의 변수 할당**

![image](https://user-images.githubusercontent.com/62214428/130079693-ece804e9-8564-4948-aa9f-a06460792907.png)
- 순서대로 살펴보면 
- `label`은 정보를 표기하기 위함으로 inspect로 살펴볼 수 있다.
- `ARG`는 변수선언으로 볼 수 있다. `ARG MESSAGE="complete"`로 MESSAGE라는 변수를 생성했고 마지막에 `RUN echo $MESSAGE` $변수 -> complete라는 메세지가 출력된다.
- `sudo docker image inspect 해당이미지` 를 통해 확인해보면 
- ![화면 캡처 2021-08-19 225023](https://user-images.githubusercontent.com/62214428/130080341-f90c525a-6db9-4540-9644-0a3951c343fe.png)
- 해당 이미지로 컨테이너를 띄워서 유저를 확인해보면
- ![화면 캡처 2021-08-19 225119](https://user-images.githubusercontent.com/62214428/130080535-8d4912e2-959b-4e65-a90b-753fcf0c59af.png)


**12) `EXPOSE` - 포트 번호 설정**

```docker
# ./docker/Dockerfile

FROM ubuntu:18.04

RUN apt-get -y update && apt-get -y upgrade
RUN apt-get -y install nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```bash
sudo docker image build -t web-server .
sudo docker container run -d -p 80:80 --name=web-server web-server
```

앞서 컨테이너로 웹서버를 띄우는 실습을 진행할 때 포트를 명령어에 같이 작성해주었던 기억이 납니다. 이미지에 포트를 작성하면 따로 컨테이너 실행 시에 `-p` 옵션으로 포트를 지정할 필요가 없는걸까요?

결론부터 말씀드리면 이미지에 작성된 `EXPOSE` 명령어는 단순히 문서의 성격만 가집니다. 실제로 호스트에서 컨테이너의 포트와 통신하도록 listening 상태를 만들어주지 않습니다. 외부로 포트가 노출되지 않는 것이죠. 그저 이미지를 빌드하고 컨테이너를 실행하는 개발자에게 어떤 포트로 어떤 방식을 이용해야 하는지 알려줄 뿐입니다. 컨테이너에서 호스트의 통신 요청에 응답하기 위해서는 반드시 `container run` 단계에서 `-p` 옵션을 통해 포트를 설정해주어야 합니다.

**13) `ADD` - 파일 복사(URL 포함)**

**14) `COPY` - 파일 복사**
- 기본적으로 copy는 현재 도커엔진이 작동하는 // 나의 경우 내 노트북 우분투에 있는 파일을 이미지로 옮긴다
- add는 추가적으로 url을 통해서도 수행이 가능하지만 보통 copy를 많이쓰고 안전
- 현재 내 우분투에 index.html이라는 파일 하나 생성
- ![화면 캡처 2021-08-19 225358](https://user-images.githubusercontent.com/62214428/130081001-ea356e64-f49b-45d3-a04a-29457c50e909.png)
- 이 후 도커파일 빌드
- ![화면 캡처 2021-08-19 225440](https://user-images.githubusercontent.com/62214428/130081083-2807f053-b5ef-4e30-a5d0-f31fcb5af491.png)
- 1) 도커파일을 보면 `WORKDIR /html`을 통해 루트 하위에 html폴더 지정(없으면 생성해서)   `ADD index.html .`를 통해 index.html을 추가 -> 컨테이너 동작시켜서 ls로 확인해보기
- 2) `WORKDIR /inside-htmlCOPY`  `index.html .` 동일하게 확인해보기
- ![화면 캡처 2021-08-19 225730](https://user-images.githubusercontent.com/62214428/130081647-8f4d86ee-87e3-4307-966d-d0db5150cad2.png)
- ![화면 캡처 2021-08-19 225759](https://user-images.githubusercontent.com/62214428/130081657-e9e1d286-fbc3-4ebe-9860-e33f3eaab3d9.png)

`ADD`와 `COPY` 는 Dockerfile이 위치한 경로에 있는 파일을 이미지로 복사해오는 명령어 입니다. 겉보기에는 기능상에 큰 차이가 없어 보이고 위의 결과도 그렇게 보이는 듯 합니다. 하지만 `ADD` 추가적으로 두 가지의 기능을 가지고 있습니다.

1. URL을 통한 파일 복사
2. 로컬에서 압축파일 복사 시, 해제하여 복사
    - 웹에서 받은 압축파일은 압축해제만 되고 tar 형태는 유지됨

로컬에 있는 파일 뿐만 아니라 웹상에 있는 파일도 이미지를 빌드할 당시에 추가할 수 있는 것입니다. 또한 압축을 자동으로 해제하기 때문에 `tar`나 `tar.gz` 을 복사하는 경우 유의해야 합니다.

