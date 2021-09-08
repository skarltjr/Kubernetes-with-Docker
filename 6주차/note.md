## Docker network / 볼륨

### 6.1.0 네트워크


##### 네트워크란?
- **네트워크**는 복수의 디바이스가 연결되어 있는 것. 
- 연결된 디바이스 간에 데이터를 주고받는 규칙을 **프로토콜(Protocol)** , 
- 우리가 이미 익숙하게 알고 있는 `http(80)`, `https(443)` 등. 


![화면 캡처 2021-09-08 141414](https://user-images.githubusercontent.com/62214428/132450314-bba867db-f56b-41f5-8cdf-1624e2225bc0.png)

**1) `ens33`**
- 디바이스의 NIC(Network Interface Controller)를 나타내고 있는 부분. 호스트와 네트워크 통신망 사이에서 데이터 송수신 역할을 수행합니다.
- 일반적인 PC에는 물리적인 NIC가 내장. 
- 구버전 리눅스에서는 `eth0`, `eth1` 

**2) `lo`**
- `lo` 는 **Loopback network interface** 에서 비롯된 용어로 `localhost` 를 의미. 디바이스의 관점에서 자기 자신을 가리키는 것. 웹서버를 구동한 PC에서 실제 브라우저를 통해 접속하여 테스트 해볼 수 있는 것이 바로 Loopback 네트워크 덕분. 하나의 디바이스가 클라이언트와 서버의 역할을 겸하는 것.
- OS 내부적으로 호스트의 이름을 IP와 매핑하여 가지고 있는 파일이 있는데, Ubuntu의 경우에는 /etc/ 경로에 hosts 라는 파일로 세팅되어 있다. 127.0.0.1 이나 localhost 로 접속해도 모두 동일한 결과를 볼 수 있는 것은 바로 이 설정 때문.

**3) `inet` `inet6`**

- `inet` 은 IPv4를 의미하며, 우리에게 익숙한 12 자리 숫자체계. 32비트로 구성되어 있으며, 10진수를 4부분으로 분할하여 표기.
- `inet6` 은 IPv6를 의미하며, 32비트의 IPv4가 고갈될 것에 대비한 128비트 체계의 IP주소. 16진수를 8부분으로 분할하여 표기. `0` 값을 가지는 경우 생략이 가능하기 때문에 실제로 조회되는 주소값은 8부분보다 적은 경우가 있다.
- 실제로 모든 디바이스가 공인 IP(Public IP)를 사용하는 것은 아니다. 대신 라우터에 의해 설정된 대역에서 사설 IP(Private IP)를 사용. 이에 따라 하나의 공인 IP를 가지고 여러 디바이스 들이 통신망에 접근할 수 있다. 라우터로 묶인 네트워크 통신망을 LAN(Local Area Network) 이라고 하며 여러 LAN 들이 모여 커다란 WAN(Wide Area Network)을 구성.
![화면 캡처 2021-09-08 142226](https://user-images.githubusercontent.com/62214428/132451124-349f5f7a-db54-4069-a589-576a5b17ab2f.png)


**4) `port`** 
IP는 디바이스가 가지는 주소. 가령 현대오피스텔 202동 을 IP 주소라고 하면 각 1002호, 304호 등과 같은 호수를 Port라고 할 수 있다. 물리적으로는 같은 위치에 있지만 각 호수별로 기능이 나뉘는 것. 우리가 사용하는 컴퓨터도 이렇게 서비스를 Port 단위로 나누고 특정 Port로만 서비스가 통신할 수 있도록 할당.
```
**Well-known Port**
미국의 IANA(Internet Assigned Numbers Authority)는 인터넷 할당 번호를 관리하는 기관으로 특정 서비스에 대한 Port 를 Well-known Port 로 지정하였습니다.

**FTP : 20, 21
SSH : 22
TELNET: 23
SMTP : 25
DNS : 53
HTTP : 80
POP3 : 110
HTTPS : 443**

*출처 : Internet Assigned Numbers Authority (IANA) [https://www.iana.org/](https://www.iana.org/)*
```

### 6.1.1 호스트-컨테이너 네트워크
![화면 캡처 2021-09-08 142601](https://user-images.githubusercontent.com/62214428/132451435-c03e71e8-b619-460b-8b7a-105ccc9cd17f.png)
1. **NIC(Network Interface Controller)**
    - 흔히 우리가 LAN 카드라고 부르는 것이 바로 이 NIC. 호스트와 네트워크간 데이터를 송수신하는 인터페이스 역할을 수행.
    - 물리적으로 구성된 NIC는 ens33으로 인식. 구버전 리눅스 배포판에서는 eth0로
2. **NAPT(Network Address Port Translation)**
    - NAT에서 PORT까지 확장된(포함된) 개념
    - 한정된 공인 ip / 내부에서 사설 ip를 사용하고 공인 ip와 매핑을 통해 극복
    - NAPT는 IP와 Port를 변환하는 기술. 도커 엔진이 컨테이너에 부여한 IP는 사설 IP로서, 호스트에서 직접 접근이 불가능. 접근을 가능케 하기 위해서 Host의 각 포트에 컨테이너의 IP와 포트를 맵핑 시켜주는 것이 NAPT의 역할.

    - `그러니까 즉 내 컴퓨터 host / 여러개의 도커 컨테이너 ( 각 컨테이너는 개별 nic(eth0)를 보유 - 개별 ip )`
3. **docker0**
    - bridge라는 형태의 네트워크이며 도커 엔진을 실행하면 자동으로 생성. 컨테이너를 실행하면 내부의 모든 포트를 `docker0` 라는 가상 bridge에 개방. 호스트는 NIC에서 직접 컨테이너와 연결하는 것이 아니라 이 bridge를 경유. 포트를 통해 각 컨테이너에 접근할 수 있도록 다리를 놔주는 역할을 수행.
4. **vethOOO**
    - 컨테이너를 띄우고 Host에서 `ifconfig` 명령어를 수행하면 `docker0`, `lo`, `ens33` 이외에 `veth`로 시작하는 것이 새롭게 생긴 것을 확인 할 수 있다. 이는 컨테이너가 Host와 통신하기 위한 가상의 NIC.
    - `veth + 난수` 의 형태로 생성.
    - 구조도 내 컨테이너에 `eth0` 이라고 표시된 이유는 Host 입장이 아닌 컨테이너 입장에서 가상 NIC를 `eth0` 라고 인식하기 때문.

### 6.1.2 도커 네트워크

- 호스트와 컨테이너 간 네트워크의 작동 방식에 대해 살펴보다. 지금부터 알아볼 것은 컨테이너 간 네트워크 - 서비스는 보통 단일 컨테이너로만 이루어지지않기 때문에 컨테이너간 네트워크가 중요하다!!
- 만약 API를 통해 데이터를 DB에 등록하려고 하는 서비스를 구축하는 경우, API 서버와 DBMS 서버를 각각 컨테이너에 구축. 컨테이너 생성 뿐만 아니라 이 두 컨테이너를 통신이 가능하도록 설정해주어야 한다.


**1) `bridge`**
![화면 캡처 2021-09-08 150654](https://user-images.githubusercontent.com/62214428/132455281-ab8ce27a-571d-47ae-9ce4-676ecdf18d4d.png)
- 도커 엔진 내부에서 동작하는 `bridge` 네트워크는 위와 같이 구성. 
- `docker0` 를 Gateway로 하여 컨테이너가 생성된 순서대로 IP가 부여. 
- 기본적으로 컨테이너를 구동할 때 별도의 네트워크 설정을 하지 않으면 모두 이 `docker0`라는 `bridge` 네트워크에 연결. 
- 같은 대역의 네트워크에 컨테이너가 구성되었기 때문에 통신도 서로 가능. 만약 커스터마이징한 별도의 `bridge` 네트워크를 구축해서 컨테이너를 구성한다면 기존의 `docker0`에 구성된 컨테이너와는 통신이 `불가능`.
- 물리적인 스위치를 생각하면 좀더 이해가 쉽다. A 스위치와 B 스위치에 각각 구성된 디바이스는 당연히 통신을 할 수 없다. 도커에서는 `bridge` 네트워크가 물리적인 스위치 역할을 한다고 보면된다.

**게이트웨이(Gateway)**
네트워크에 접근하기 위한 출입구. 우리가 스마트폰과 노트북으로 와이파이에 접속하는 것도 모두 게이트웨이를 통해 인터넷 망에 접근하는 것. 이와 같이 서로 다른 네트워크 간 접근을 위해서는 게이트웨이 주소가 필요하며, 기본적으로 IP 마지막 자리를 `1`로 설정.

```
sudo docker container run -d -p 80:80 --name hello1 httpd
sudo docker container run -d -p 8080:80 --name hello2 httpd
sudo docker container run -d -p 8081:80 --name hello3 httpd
```

` sudo docker container inspect --format="{{ .NetworkSettings.IPAddress }}"를 통해 ip 확인 `

![화면 캡처 2021-09-08 151009](https://user-images.githubusercontent.com/62214428/132455622-e79b0fd5-f90e-4667-ada6-8d4a51a6b639.png)

------------------
 ` 참고로 거의 대부분 1번 bridge 방식을 따르고 2,3번은 참조용 `
 
 **2) `host`**
 ![화면 캡처 2021-09-08 151152](https://user-images.githubusercontent.com/62214428/132455805-a8558f40-886e-47dc-8ab1-f9907c6b17c0.png)
`host` 네트워크는 단어 그대로 별도의 경유 없이 도커 엔진이 작동하고 있는 Host의 IP 주소를 그대로 사용하는 것. `bridge`를 통해 NAPT의 과정을 거치지 않아도 되며, 명시적으로 컨테이너의 포트를 명령어에 작성하지 않아도 된다. 다시 말해 컨테이너를 격리된 네트워크로 구성하지 않고 도커 엔진과 한 공간에 두는 것.

` 주의 !! ★ host` 형식의 네트워크는 Host OS가 리눅스인 경우에만 지원하며, Docker Desktop의 형태로 사용하는 경우 지원 x .

```
~$ sudo docker container run -it --network host --name ubuntu-host ubuntu:18.04

/# apt-get update
/# apt-get install net-tools
```

**3) `none`**
`none` 은 말 그대로 컨테이너를 격리된 공간에 차단시켜놓고 어떠한 네트워크와도 연결하지 못하게 하는 것.

```
~$ sudo docker container run -it --network none --name ubuntu-none ubuntu:18.04

/# apt-get update
```


### 6.1.3 도커 네트워크 명령어

**1) `ls` - 네트워크 목록 조회**

```bash
sudo docker network ls
sudo docker network ls --filter driver=bridge

filter를 통해 원하는 것만 조회가능
```
![화면 캡처 2021-09-08 155955](https://user-images.githubusercontent.com/62214428/132461667-7286326c-eeb9-4e62-b399-56657c462f2e.png)

**2) `create` - 네트워크 생성**

```bash
sudo docker network create --driver=bridge new-bridge
```
![화면 캡처 2021-09-08 160123](https://user-images.githubusercontent.com/62214428/132461855-0b05e167-5e42-4c4b-9819-874475b125b4.png)

- 참고 : 보통은 명령어를 입력해서 생성하기 보다는 docker-compose.yaml 내에 세팅하는 경우가 더 많다.

**3) `connect` - 컨테이너를 네트워크에 연결**

```bash
sudo docker network connect new-bridge webserver1
sudo docker container inspect webserver1
```

![화면 캡처 2021-09-08 160320](https://user-images.githubusercontent.com/62214428/132462074-d8c17121-051e-4b07-9b91-f25e772704f8.png)

` sudo docker container inspect 컨테이너이름 `

![화면 캡처 2021-09-08 160426](https://user-images.githubusercontent.com/62214428/132462199-0f234876-856e-4094-b724-15fe685165a4.png)

`현재 hello 컨테이너는 두 개의 네트워크와 연결된 상태 / 원래 bridge에만 연결 -> new-bridge까지 / 하나의 컨테이너는 여러개의 네트워크(ip확인)와 연결될 수 있다 `

```
docker container run` 에 `--net` 옵션을 추가해서 컨테이너를 생성하면 디폴트 네트워크 설정값인 `bridge` 를 연결하지 않고 
명령어에 설정된 `new-bridge` 만 연결

sudo docker container run -d -p 8082:80 --name=webserver4 --net=new-bridge httpd
sudo docker container inspect webserver4
```

**4) `disconnect` - 컨테이너를 네트워크에서 해제**

```bash
sudo docker network disconnect new-bridge hello
```
![화면 캡처 2021-09-08 160811](https://user-images.githubusercontent.com/62214428/132462669-c357afdd-e9c5-49d4-b6fd-297b078ac5c2.png)
- inspect로 확인


**5) `inspect` - 네트워크 상세정보 조회**

```bash
sudo docker network inspect new-bridge
```

**6) `rm` - 네트워크 삭제**

```bash
sudo docker network rm new-bridge
```

## 6.2 도커 볼륨

컨테이너 내부는 어플리케이션을 실행하기 위한 다양한 파일로 구성되어 있다. 이러한 파일들을 관리하기 위해 디스크 드라이브가 내부적으로 작동.

그렇다면 컨테이너 내부의 파일은 영원할까?  컨테이너는 영원한가요?

컨테이너는 언제든지 멈추고 삭제할 수 있다. 컨테이너 내부의 디스크에서 존재하던 파일 혹은 데이터 역시 컨테이너를 삭제하게 되면 함께 사라진다. 영구적으로 혹은 일정 기간동안 데이터를 보관하는 용도로 사용해야 한다면 이를 관리하기 위해 다른 방법을 생각해야 한다.

도커에서는 `Volume`과 `Bind Mount`라는 개념으로 Host의 일부 영역을 할당해 스토리지로 사용할 수 있게 해준다.

**6.2.2 Volume**

Volume은 Bind Mount 이후에 나온 개념으로, 호스트의 경로를 임의로 지정할 수 없다. 대신 도커 엔진에서 관리하는 `/var/lib/docker/volumes/<볼륨명>` 으로만 세팅. 볼륨은 컨테이너를 실행하는 단계에서 생성할 수 있으며 볼륨만 개별적으로 생성하는 것도 가능.
` 참고로 내가 사용한 wsl2 환경에서는 \\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes에 위치 `

```

~$ sudo docker container run -d -it --name volume -v volume1:/volume-test ubuntu:18.04
                                      
~$ sudo docker container attach volume

/# echo "volume" > ./volume-test/volume.txt

~$ cd /var/lib/docker/volumes/volume1/_data
~$ cat volume.txt
```

### 중요
` ~$ sudo docker container run -d -it --name volume -v volume1:/volume-test ubuntu:18.04 `

` 
  volume1:/volume-test ubuntu:18.04 ->  volume1과 컨테이너 내부 volume-test 연결 
  따라서 volume1은 내 컴퓨터 디렉토리에서 접근할 수 있고
  volume-test는 컨테이너 내부에서 접근할 수 있다
`        

- 현재 컨테이너를 위와 같이 실행한 상태 
- \\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes\volume1\ --> 위에서 말한 volume1이 생성
- ![화면 캡처 2021-09-08 173812](https://user-images.githubusercontent.com/62214428/132476046-73f82276-586c-4daa-aa68-4fb71a1ffcc9.png)
- 컨테이너로 접속해보면 volume-test 디렉토리가 생성된것을 확인
- 그럼 확인해볼 수 있는게 컨테이너에서 txt파일을 생성하면 이를 저장하기 위한 공간인 volume에도 생성될 것 
- ![화면 캡처 2021-09-08 174334](https://user-images.githubusercontent.com/62214428/132476932-7b08da63-fcaa-4ffe-8e0a-40ad9c603153.png)



-------

```bash
# 볼륨 개별 생성

sudo docker volume create volume2
```
