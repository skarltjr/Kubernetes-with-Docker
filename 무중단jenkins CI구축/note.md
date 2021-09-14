우선 나는 GCP / WSL2 / Docker를 사용

# 0. Gcp에서 jenkins 인스턴스 생성 ( vm )
- 8080 50000포트 열어둘 것 
- 해당 내용은 나의 배포 issue 참고 - https://github.com/skarltjr/Memory_Write_Record/issues/15
- 참고로 여기선 centOs 대신 ubuntu 18.04로 수행했음
![화면 캡처 2021-09-14 150023](https://user-images.githubusercontent.com/62214428/133203192-6d6bacde-ad6f-4945-9583-5ae421f0fbd4.png)
- https://blog.dalso.org/linux/ubuntu-20-04-lts/13118 - 도커 설치
- ![화면 캡처 2021-09-14 150907](https://user-images.githubusercontent.com/62214428/133204154-cbd76e3b-3faa-4baa-a557-547969840a6b.png) - 도커 구동 확인 
- 참고로 centOs와 ubuntu의 차이에 따라 이전 issue랑은 좀 차이가 있었다

# 1. Jenkins 컨테이너 구동 및 플러그인 설치 - vm

 `8080`, `50000` 포트는 사전에 VM 서브넷 수신 규칙에 미리 추가. 

```bash
sudo docker run --name jenkinsci -u root -p 8080:8080 -p 50000:50000 -v $(which docker):/usr/bin/docker -v $HOME/.jenkins/:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts
```
- ![화면 캡처 2021-09-14 151913](https://user-images.githubusercontent.com/62214428/133205239-a3edc4bf-f140-40ff-a165-fce14ff5dac5.png)
- 해당 부분을 어딘가에 기억한 후
- 해당 vm instance 외부 ip:8080으로 접속 ! 하여 붙여넣기
- suggested plugin을 install
- 계정 설정 / 나는 계정명 : kiseok
- 젠킨스 설치 및 사용시작
![화면 캡처 2021-09-14 152410](https://user-images.githubusercontent.com/62214428/133205903-a1ed4c95-493b-438e-9bb8-af1d1f5f7420.png)
- ctrl c로 빠져나온 후 해당 컨테이너(jenkinsci)를 백그라운드로 실행 `sudo docker start jenkinsci`
- 껏다 켰으니 재접속해보고

**Manage Jenkins - Manage Plugins** 페이지로 이동 
- // 한글판으론 jenkins관리 -> 플러그인 관리 
- 설치 가능 탭에서
- docker pipeline install without restart

# 2. 컨테이너 SSH Public/Private Key 등록

1) Jenkins가 Github Repository에 접근하기 위해서는 SSH key가 필요. 컨테이너의 SSH key 생성을 위해 다음의 명령어를 실행.

```bash
sudo docker exec -it jenkinsci ssh-keygen
```
2) 정상적으로 SSH key가 생성 되었다면 Public Key 값을 조회하기 위해 다음의 명령어를 실행. 

```bash
sudo docker exec -it jenkinsci cat /root/.ssh/id_rsa.pub
```
3) 깃허브에서 연동할 레포지토리를 만들고 조회된 키를 Github Repository 에 등록하기 위해 **Settings - Deploy keys** 화면에서 **Add deploy key** 버튼.
- ![화면 캡처 2021-09-14 153329](https://user-images.githubusercontent.com/62214428/133207065-949d3113-eade-4691-9635-b023537add4d.png)

5) 다음은 SSH Private Key 값을 Jenkins Credential 에 추가. 명령어를 실행해서 Private Key 값을 조회.
- RSA 비대칭키 암호화방식

```bash
sudo docker exec -it jenkinsci cat /root/.ssh/id_rsa
```

6) 브라우저의 Jenkins 대시보드에서 **Manage Jenkins** 를 클릭하고 이어 **Manage Credentials** 를 클릭.
7) **Jenkins - Global credentials (unrestricted) - Add Credentials** 를 차례대로 클릭.
- ![화면 캡처 2021-09-14 153554](https://user-images.githubusercontent.com/62214428/133207343-f4d6f90f-4044-448a-a5ce-3712d79c7663.png)
8) Kind 는 SSH Username with private key 로 설정한 후, ID는 식별을 위한 적절한 값을 입력. 하단 Private Key 항목에 터미널에 출력된 SSH Private Key 를 붙여넣고 OK.
- ![화면 캡처 2021-09-14 153839](https://user-images.githubusercontent.com/62214428/133207713-2c64ec69-41d5-4e50-8c1b-cedd72fe80d7.png)

# 3. Docker Hub 계정 등록

1) 브라우저의 Jenkins 대시보드에서 **Manage Jenkins** 를 클릭하고 이어 **Manage Credentials** 를 클릭.
2) **Jenkins - Global credentials (unrestricted) - Add Credentials** 를 차례대로 클릭.
3) Kind 는 Username with password 로 설정한 후, Username 과 Password는 사용하고 있는 Docker Hub 계정정보를 입력/ ID는 인증정보를 식별할 수 있는 값.
![화면 캡처 2021-09-14 155321](https://user-images.githubusercontent.com/62214428/133209623-fd0f39ca-4b4c-40e7-8eeb-f17c70b75d3d.png)
4) 총 2개의 credential 확인
![화면 캡처 2021-09-14 155349](https://user-images.githubusercontent.com/62214428/133209698-92c85c90-c7d7-4976-8644-8d74b6f6a29d.png)

# 4. Github WebHook 등록

1) **Settings - Webhooks** 화면에서 **Add webhook** 버튼.
2) Payload URL에 http://<인스턴스 외부IP>:8080/github-webhook/ 를 입력하고 Content type 은 application/x-www-form-urlencoded 를 선택.
![화면 캡처 2021-09-14 155614](https://user-images.githubusercontent.com/62214428/133210009-c384b4fe-bd36-4e7c-863e-1d529e27f545.png)


# 5. Jenkinsfile 생성
- 우선 도커허브 저장소 하나 생성
- ![화면 캡처 2021-09-14 160124](https://user-images.githubusercontent.com/62214428/133210663-5700087c-b90d-431a-a22e-0ec4effc7414.png)
- 우분투에서 폴더만들고
- git init
- git clone
- 이 후
1) 이미지를 빌드할 로컬 Repository에서 **Jenkinsfile** 을 생성한 후 아래와 같이 작성.
node {
     stage('Clone') {
         checkout scm
     }
     stage('Build') {
         app = docker.build("skarltjr/jenkinsci")
         # 빌드할 이미지명은 자신의 Docker Hub 계정.
				 # app = docker.build("<계정명>/<저장소명>")
     }
     stage('Push') {
         docker.withRegistry('https://registry.hub.docker.com', 'docker_hub') {
             app.push("${env.BUILD_NUMBER}")
             app.push("latest")
         }
     }
 }
2) Jenkinsfile을 Github Repository 에 push.
```
git add -A
git commit -m "Jenkinsfile added"
git push -u origin main
```
