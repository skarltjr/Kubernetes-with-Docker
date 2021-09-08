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









