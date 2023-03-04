## 쿠버네티스 네트워크 구조 2. service

### 1. service란?
- 쿠버네티스의 파드는 유동적이다. 즉 언제든지 삭제, 생성될 수 있으며 어떤 node에서든지 생성될 수 있다.
- 이런 유동적인 pod에 고정적인 연결을 제공하는것이 바로 service다.
- 서비스는 요청을 어떤 pod에 전달할지 결정해주는것
```
이런 문제점을 대체하기 위한 방법이 바로
reverse-proxy다. 즉 pod 앞 단에 프록시을 위치시키고 요청을 해당 프록시가 전달받은 후 뒷단의 
pod로 넘겨주는것.

그리고 쿠버네티스의 프록시가 바로 service다.
```
### 2. 그런데 service는 nic가 없다.
- 네트워크 통신을 위해선 nic가 필요한데 service와 관련된 network device는 ifconfig로 확인해도 없다.
- 심지어 라우팅 테이블에서 검색해도 service에 관한 라우팅 정보는 존재하지 않는다.
- 즉 service는 그저 쿠버네티스 리소스일뿐 


### 3. 그럼 서로 다른 호스트간 컨테이너는 어떻게 service를 통해 통신하는건가. kube-proxy
- kube proxy는 워커노드마다 갖는다.
- kube proxy는 netfilter와 user space에 존재하는 interface인 iptables을 활용한다.


### 4. Kube-proxy가 직접 UserSpace Proxy역할 시
```
먼저 netfilter는 리눅스 커널 기능 중 하나다.
정해진 규칙을 생성하고 그 규칙에 맞는 패킷이 들어오면 정의된 동작을 수행한다.
```

- ![141045833-931586a3-7fcc-498a-b2c6-34eb638409fb](https://user-images.githubusercontent.com/62214428/222901085-bf41479a-0b1e-4f06-a30b-17681b7f4fa0.png)
```
해당 모드에서 쿠베 프록시는 netfilter로 하여금 특정 service로 들어오는 요청을 자신에게 라우팅하도록 설정한다.

```
- pod A내의 프로세스에서 pod B에 가야할 일이있다.
- 그래서 pod B서비스 IP를 통해 접근하려고 요청을 보냄
- 그런데 pod A는 ? 나는 이게 어디로 가야할 지 모르겠어 ;;
- ip네트워크는 보통 자신의 host에서 목적지를 찾지 못하면 상위 게이트웨이로 올려보낸다. 그래서 상위로 올려보내고
- netfilter에 도달 ( netfilter는 물리서버 커널 - 여기선 host인 워커노드 os의 커널에 존재 )
- netfilter는 아 이 서비스로 요청온거는 A쿠베프록시한테 물어보도록 라우팅 되어있으니 10400포트 쿠베프록시에게 접근
- 쿠베프록시는 이거는 ~로 가라고 알려줘서 pod B에 접근

### 5. 그런데 Kube-proxy가 직접 UserSpace Proxy역할 시 문제점이 존재한다.
- 모든 패킷을 user-space에서 kernel-space로 변환해야한다.

### 6. Kube-proxy가 Iptables를 통해 netfilter조작하는 역할 시
```
- 앞선 문제르 해결하고자 쿠베 프록시가 직접 하나의 proxy 역할을 수행하는게 아니라 모두 netfilter에게 맡긴다.
- 해당 모드에서 쿠베 프록시는 그저 netfilter의 규칙을 조작할뿐이고
- service ip를 발견하고 그걸 실제 pod로 전달하는건 모두 netfilter가 담당한다.
```
- 마찬가지로 pod A내 프로세스에서 pod B에 접근하기 위해
- 먼저 pod B Service Ip에 접근
- pod A는 `pod B Service Ip`에 대해 몰라서 상위로 넘기고
- 이때 netfilter가 `pod B Service Ip`를 보고 `아 이건 여기로 가`를 바로 수행
- 호스트는 정보를 받고 pod B로 바로 접근
    - 어떻게 가능한가?
    - 기생성된 파드를 관리하는 컨트롤러 매니저는 새로운 pod가 생성되었거나 replicaset이 생성되었거나 혹은 `pod가 죽어서 새로 생성했는데 그래서 ip도 바뀌었어`등을 알고 관리하는데
    - 이를 계속 쿠베프록시에게 전달해주고
    - 쿠베프록시는 그때마다 이를 netfilter에 전달 ( ★ 이 때 netfilter는 커널이라고 했다. 그러니까 `kubeproxy`가 `iptables`를 껴서 정보를 전달하여 netfilter 커널에 반영시키는 것)
    - 그러니까 결국 kubeproxy도 iptables, netfilter같은 OS와 관련된 것들을 활용하여 변환을 가능하도록한다.


### 7. 근데 어떤 service로 가야해?
- ingress
