## Services
- 여러 레플리카에 트래픽을 분산시키는 로드밸런서 (TCP, UDP 모두 가능)
- 노드의 kube-proxy 를 활용하여 엔드포인트로 라우팅 되도록 처리



![화면 캡처 2021-11-10 124118](https://user-images.githubusercontent.com/62214428/141045833-931586a3-7fcc-498a-b2c6-34eb638409fb.png)

### 1. 기존에 Kube-proxy가 직접 UserSpace Proxy역할 시
- pod A내의 프로세스에서 pod B에 가야할 일이있다.
- 그래서 pod B서비스 IP를 통해 접근하려고 요청을 보냄
- 그런데 pod A는  ? 나는 이게 어디로 가야할 지 모르겠어 ;;
- 그래서 이더넷같은 경우 상위로 올려보내고
- netfilter에 도달 ( netfilter는 물리서버 커널에 존재 )  
- netfilter는 `아 이거는 쿠베프록시한테 물어보자`해서 10400포트 쿠베프록시에게 접근
- 쿠베프록시는 `이거는 ~로 가`라고 알려줘서 pod B에 접근

### 2. Kube-proxy가 Iptables를 통해 netfilter조작하는 역할 시
- 마찬가지로 pod A내 프로세스에서 pod B에 접근하기 위해 
- 먼저 pod B Service Ip에 접근
- pod A는 `pod B Service Ip`에 대해 몰라서 상위로 넘기고
- ★이 떄 이 경우에선 netfilter가 `pod B Service Ip`를 보고 `아 이건 여기로 가`를 바로 수행
- 호스트는 정보를 받고 pod B로 바로 접근
  - 어떻게 가능한가?
  - `controller`는 새로운 pod가 생성되었거나 replicaset이 생성되었거나 혹은 `pod가 죽어서 새로 생성했는데 그래서 ip도 바뀌었어`등을 알고 관리하는데
  - 이를 계속 쿠베프록시에게 전달해주고
  - 쿠베프록시는 그때마다 이를 netfilter에 전달 ( ★ 이 때 netfilter는 커널이라고 했다. 그러니까 `kubeproxy`가 `iptables`를 껴서 정보를 전달하여 netfilter 커널에 반영시키는 것)
  - 그러니까 결국 kubeproxy도 iptables, netfilter같은 OS와 관련된 것들을 활용하여 변환을 가능하도록한다.


### 직접 실습해보자
1. deployment 구성
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80

```
- 사실 위는 쿠버네티스 공식문서의 yaml을 직접 만들필요없이 해당 링크를 apply하면 된다.
- apply한 후 get pod해보면 2개 확인가능 ( replicaset 2였으니)


2. `expose`를 통해 서비스를 만들고 연결하자
![화면 캡처 2021-11-10 125858](https://user-images.githubusercontent.com/62214428/141047351-bd8ef0f1-8f0b-44fd-9715-3466a402f6b5.png)
- `expose`명령어를 활용하면 ex) `kubectl expose deployment my-nginx`
- my-nginx라는 deployment에 대해 서비스를 만들고 연결해주세요! ( --name옵션을 통해 서비스 이름지정가능)
- 


































