## 도커 레지스트리
도커 레지스트리는 도커 이미지들을 모아 놓고 필요할 때 사용할 수 있도록 구축한 저장소.   
도커 이미지를 빌드하거나 컨테이너를 실행할 수 있었던 것도 레지스트리의 이미지를 사용했기 때문.

도커라는 플랫폼을 정의했던 문장이 다음과 같다.

`도커는 어플리케이션 및 실행 환경을 정의한 이미지를 생성/공유함과 동시에, 이를 이용하여 컨테이너를 작동할 수 있도록 하는 플랫폼이다.`

레지스트리를 통하여 이미지를 공유하는 여러 가지 방법과 이용 가능한 플랫폼 등을 확인해보자

-----------
- 도커의 기능을 한 눈에 알아볼 수 있는 사진
![화면 캡처 2021-08-29 141752](https://user-images.githubusercontent.com/62214428/131239395-fe6ff6c8-145c-49a3-bbdc-a2c72e71293c.png)

Dockerfile을 통해 빌드된 이미지가 컨테이너를 실행하는 데에 이용되기도 하고, 혹은 레지스트리에 저장되어 여러 사람이 사용할 수 있도록 공개.  
이를 각각 Building, Running, Shipping(Sharing) 이라는 단어로 표현.  

도커 이미지가 공유되는 레지스트리를 크게 3가지로 구분해보면, 디폴트로 이미지를 받아오는 도커의 `도커 허브`,  
사용자가 직접 컨테이너에 구축한 `로컬 레지스트리`, 클라우드 사업자에서 제공하는 `컨테이너 레지스트리`가 있다.

레지스트리 구축전에 레지스트리를 사용하는 목적에 대해 알고 넘어가자.   
이미지를 파일 형식으로도 저장 가능한데 왜 굳이 비용을 지불하면서 레지스트리를 사용하는 걸까? 
![화면 캡처 2021-08-29 142158](https://user-images.githubusercontent.com/62214428/131239534-3564d672-21ec-495f-9907-e3c7d88d9656.png)

첫 번째는 `보안 문제`.  
이미지 내부에 보안상 중요한 파일이나 내용이 필요한 경우에는 더욱 엄격히 관리할 필요가 있다.  
이미지 자체도 암호화 되어야 하고 레지스트리에 접근할 수 있는 권한도 통제.  
이런 일련의 것들을 클라우드 사업자에서는 `IAM(Identity and Access Management)` 라는 형태로 제공. 

두 번째는 `배포 파이프라인 효율화`.  
위 그림에서 보다시피 레지스트리에 저장된 이미지는 단순히 저장에서 끝나는 것이 아니라 CI/CD 라는 과정까지 연속적으로 활용.

**CI(Continuous Integration) : 지속적 통합**
빌드/테스트가 자동으로 수행된 이후 개발자의 수정 사항이 중앙 저장소에 머지되는 것 까지의 과정
일련의 자동화 과정 및 개발 문화가 포괄

**CD(Continuous Delivery) : 지속적 전달**
운영 환경에 배포할 소스코드가 자동으로 세팅
빌드 이후의 변경 사항을 테스트 및 운영 서버에 자동으로 배포
필요에 따라 테스트 서버를 동적으로 생성할 수 있음


지금부터 하고자 하는 것 : 이미지 파일을 저장하는 레지스트리에 이미지 파일을 저장하기

------------

## 5.1 도커 허브
도커 허브는 도커에서 디폴트로 참조하는 레지스트리로서, 각종 공식 이미지를 손쉽게 사용할 수 있도록 구성되어 있다. 

### 먼저 레포지토리 생성
![화면 캡처 2021-08-29 151050](https://user-images.githubusercontent.com/62214428/131240591-d1dbdb91-44e9-45ca-93f1-b1e8ae19da33.png)

### 우분투에서 도커 레지스트리에 올릴 이미지를 빌드 
`sudo docker build -t <계정명>/<저장소명>:[태그명] .`  
빌드 명령어는 규칙으로 잘 기억하자.
![화면 캡처 2021-08-29 151211](https://user-images.githubusercontent.com/62214428/131240614-76053722-659d-4ae0-8b1c-c95faa784970.png)

- 도커 파일은 아래처럼 구성
![화면 캡처 2021-08-29 151348](https://user-images.githubusercontent.com/62214428/131240641-6bcce0ca-f8ad-42cd-907a-89d410b1e6ec.png)
```
FROM nginx:alpine
# alpine은 경량화 리눅스 배포판 입니다. 
# 도커 베이스 이미지로 alpine 리눅스를 활용하는 이유는 이미지 용량을 적게 차지할 뿐만 아니라 처리속도가 빠르다는 이점이 있기 때문입니다. 

WORKDIR /usr/share/nginx/html
# 이 경로는 nginx 웹서버의 index.html 파일이 위치한 곳입니다.
# 브라우저를 통해 웹서버로 접근했을때 로드되는 페이지가 바로 이 곳을 참조하여 렌더링 되는 것입니다.

RUN rm -rf ./*
# 클론한 소스를 이미지에 복사하기 위해 기존 이미지 내의 파일을 전부 삭제합니다.

COPY ./* ./
# Dockerfile이 위치한 경로의 html, css, js 등의 파일을 이미지 내로 복사합니다.    // 도커파일이 위치한 경로의 모든 파일 ./*을 현재 WORKDIR 경로( ./ )로 

ENTRYPOINT ["nginx", "-g", "daemon off;"]
# nginx 웹서버를 기동합니다.

```
### 만든 이미지를 푸쉬할 차례
![화면 캡처 2021-08-29 151721](https://user-images.githubusercontent.com/62214428/131240693-a080ee18-3d9a-4629-9355-37f276ef4162.png)
![화면 캡처 2021-08-29 151748](https://user-images.githubusercontent.com/62214428/131240704-558d39a2-e5fa-4e07-a207-c038ac4d5e9c.png)


`여기까진 도커허브를 통해 이미지를 저장`

----------

## 5.2 로컬 레지스트리 구축
`클라우드 서비스보다 더 보안이 필요한 경우 외부 레지스트리가 아닌 서버내 직접 레지스트리를 구성해서 사용하는 경우가 있다 그런 방법을 실습하자`
`그러니까 서버 혹은 로컬에 직접 도커허브와 같은 레지스트리를 구성하는 경우`

도커를 통해 실행되는 어플리케이션은 컨테이너 단위로 실행. 이미지가 저장되는 레지스트리 역시 컨테이너 상에 구축할 수 있는데, 이는 도커 회사에서 [Registry](https://hub.docker.com/_/registry) 라는 이름으로 제공하고 있다.

**1) Registry 컨테이너 실행**
- 먼저 레지스트리를 구성하기위해 레지스트리 컨테이너를 실행

```bash
sudo docker run -d -p 5000:5000 --restart always --name registry registry:2
# --restart always의 경우 도커 엔진이 재시작되는 경우 자동으로 컨테이너를 재시작하도록 하는 옵션.
```
![화면 캡처 2021-08-29 155335](https://user-images.githubusercontent.com/62214428/131241497-69c935ff-7846-4b5d-837b-49bbe041d784.png)
 
**2) 이미지 태깅**

```bash
sudo docker tag <기존 이미지명>:[태그명] <레지스트리 컨테이너 IP>/<이미지명>:[태그명]
기존 이미지에 다른 이름을 붙여 현재 우분투 서버에서 동작중인 레지스트리 컨테이너에 저장해보기 위해 
```
![화면 캡처 2021-08-29 155616](https://user-images.githubusercontent.com/62214428/131241568-19c33a98-80f4-4d7c-9470-136586262d4c.png)
![화면 캡처 2021-08-29 155638](https://user-images.githubusercontent.com/62214428/131241570-fd7c1662-3f74-4fcb-85cf-01401ccefd33.png)

##### 참고로 sudo docker tag <기존 이미지명>:[태그명] <레지스트리 컨테이너 IP>/<이미지명>:[태그명] 같은 양식은 반드시 지켜야 인식이 가능

**3) 이미지 공유**
![화면 캡처 2021-08-29 155745](https://user-images.githubusercontent.com/62214428/131241597-675a023b-b9af-4f11-a795-91a7bf0ddd00.png)

- 그러면 현재 내 우분투에 로컬레지스트리가 컨테이너로 구동중이고 거기에 push를 통해 localhost:5000/mydocker:1.1이라는 이미지를 올려서 저장해두고 있다.
- docker image ls로 확인해보면 현재는 이미지가 로컬에 있는데 이 이미지를 지우고 레지스트리 컨테이너에서 다시 받아올 수 있지 않을까?
- ![화면 캡처 2021-08-29 155957](https://user-images.githubusercontent.com/62214428/131241650-529a910c-c7cb-49eb-9657-e9167d082cca.png)


## 5.3 GCP Artifact Registry

#### 도커허브 / 로컬 레지스트리 / 그리고 마지막으로 클라우드 레지스트리를 활용해보기

- 기존에 사용하던 gcp계정이 있어서 난 그걸 사용


**1) GCP Console 접근 및 프로젝트 생성**

![화면 캡처 2021-08-29 180354](https://user-images.githubusercontent.com/62214428/131244956-5bced917-31f5-4af6-8439-802969385141.png)

**2) 결제 계정 등록인데 이미 되어있으므로 ..**

**3) 서비스 사용 등록**
![화면 캡처 2021-08-29 180659](https://user-images.githubusercontent.com/62214428/131245063-5c120fcc-c22a-4e3d-b3e7-a5eb3ca0a550.png)

**4) GCP 저장소 만들기**
![화면 캡처 2021-08-29 180815](https://user-images.githubusercontent.com/62214428/131245090-a998b4e5-24e3-4397-a5da-d3330f3189e1.png)

![화면 캡처 2021-08-29 180743](https://user-images.githubusercontent.com/62214428/131245078-d1f84d8c-0da5-4f6a-955e-be053db8c274.png)

**5) 리눅스 도커 보안 그룹 설정**

```bash
sudo usermod -a -G docker [계정명]
해당계정은 도커계정말하는거
```
**6) google cloud SDK 패키지 경로 추가**

```bash
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
```
- 패키지 소스 URI를 추가. 사용하려는 gcloud는 이 경로를 통해 내려받게 된다.


**7) google cloud SDK 공개키 내려받기**

```bash
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
```

**8) google cloud SDK 설치**

```bash
sudo apt-get update && sudo apt-get install google-cloud-sdk
```

**9) google cloud SDK 초기화**

```bash
gcloud init
```

![화면 캡처 2021-08-29 181054](https://user-images.githubusercontent.com/62214428/131245154-fc0a3b4a-985e-49e5-97eb-c6ceaf2f69b1.png)
- 빨간 부분을 크롬에 복사해서 접근하고 설정한 후 나오는 코드를 Enter verification code: 여기에 붙여넣으면 된다
![화면 캡처 2021-08-29 181159](https://user-images.githubusercontent.com/62214428/131245174-4ae9f838-987f-4467-97ec-c99f020a4b89.png)
- 사용하려는 레포지토리 선택

**10) GCP Registry 저장소 인증**

```bash
sudo gcloud auth configure-docker asia-northeast3-docker.pkg.dev
```

**11) 이미지 태깅**

```bash
sudo docker tag skarltjr/mydocker:1.0 asia-northeast3-docker.pkg.dev/[프로젝트ID]/[저장소명]/mydocker(이건 그냥 저장소명과 동일하게)

sudo docker tag skarltjr/mydocker:1.0 asia-northeast3-docker.pkg.dev/skarltjr-docker-registry/mydocker/mydocker
```
참고로 asia~~ 다 칠 필요없이
![화면 캡처 2021-08-29 181414](https://user-images.githubusercontent.com/62214428/131245241-1409a987-108e-42a2-8b5a-29e8f64a0489.png)

- 내 우분투에 만들어둔 이미지를 gcp 레지스트리에 올리기위해 따로 이름을 붙여준 부분이 바로 이 태깅이다
- 이제 다음으로 이 이미지를 올리기위해 push

**12) 이미지 공유**

```bash
sudo docker push asia-northeast3-docker.pkg.dev/[프로젝트ID]/[저장소명]/mydocker

sudo docker push asia-northeast3-docker.pkg.dev/skarltjr-docker-registry/mydocker/mydocker
보면 여기서 마지막에 mydocker이후 태그는 안붙여줬다 그럼 latest로 들어갈것
```
![화면 캡처 2021-08-29 181616](https://user-images.githubusercontent.com/62214428/131245297-983bfe69-1f60-4b52-a42b-a7f3cfa4e2fe.png)

#### 이렇게 GCP 레지스트리에 나의 도커 이미지를 올려서 저장할 수 있다.

----------

### GCP에는 추가적인 기능이 있는데 바로 trigger
- 이전에 GCP & 깃허브 webhook을 통해 레퍼지토리에 변경사항이 있으면 이를 반영하여 자동 배포화를 경험한적이있다.
- 이미지도 동일하게 자동으로 변경사항을 반영하여 저장할 수 있다. 그 때 사용할 수 있는게 트리거
- cloud build api & artifact registry를 사용으로

- 먼저 깃허브 레포지토리와 연결
- ![화면 캡처 2021-08-29 212504](https://user-images.githubusercontent.com/62214428/131250336-476343ff-26bd-4c85-a928-516a31f63a9d.png)
- ![화면 캡처 2021-08-29 212549](https://user-images.githubusercontent.com/62214428/131250339-92408236-6e89-46a8-9f20-0db7809f28c2.png)
- ![화면 캡처 2021-08-29 212607](https://user-images.githubusercontent.com/62214428/131250344-eea1769c-a0eb-43b2-afac-2db3e1bfa6cd.png)
- ![화면 캡처 2021-08-29 212621](https://user-images.githubusercontent.com/62214428/131250347-a22154c2-3fcb-4eac-9f20-2cd8af303728.png)
- ![화면 캡처 2021-08-29 212638](https://user-images.githubusercontent.com/62214428/131250350-e9be1877-7f2c-4380-b6e9-bb88cdac6874.png)


- 이제 트리거를 만들 차례

- ![화면 캡처 2021-08-29 212504](https://user-images.githubusercontent.com/62214428/131250257-83fab733-b388-4453-b87b-193bbcb693cd.png)


참고로 아래 설정도 해줘야한다
- ![화면 캡처 2021-08-29 212932](https://user-images.githubusercontent.com/62214428/131250367-07f0df3f-1b01-4c70-8b6f-d9dfaddd4d66.png)

이 후 변경사항 푸쉬 후 확인해보면
- ![화면 캡처 2021-08-29 213025](https://user-images.githubusercontent.com/62214428/131250382-853e48f6-769a-4340-adee-3c58cecfc899.png)

참고로 클라우드 서비스 과금은 본인이 주의하기
artifact registry는 500mb 무료 / cloud build api는 하루 2시간넘으면 과금
