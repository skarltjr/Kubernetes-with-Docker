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
```

`export` 역시 `container` 와 함께 사용되는 명령어. 
현재 컨테이너를 본떠 파일로 내보내는 역할을 하고 리눅스에서 많이 사용되는 파일 압축 형식인 `.tar` 를 사용. 이렇게 생성된 파일은 import 명령어를 사용하여 이미지를 생성할 수 있다.

**8) `import` - 파일을 이미지로 생성**

```bash
sudo docker image import apache.tar ggingmin/apacheweb:1.1
```
