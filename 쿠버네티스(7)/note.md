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


### 직접 서비스를 생성해보자.
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
- ![화면 캡처 2021-11-10 135501](https://user-images.githubusercontent.com/62214428/141052290-4d856caa-b4e2-4ba6-9936-9ef275ee21e0.png)

------

## DNS
- 도메인 네임 시스템(Domain Name System, DNS)은 호스트의 도메인 이름을 호스트의 네트워크 주소로 바꾸거나 그 반대의 변환을 수행할 수 있도록 하기 위해 개발되었다
- 우리가 구글의 ip가 아닌 www.google.com으로 접속할 수 있는 이유
- ![화면 캡처 2021-11-10 140226](https://user-images.githubusercontent.com/62214428/141052970-3063ef2b-e011-474d-8cdf-3dc570d7e1f5.png)

1. 쿠버네티스는 디폴트로 CoreDns를 사용한다
- ![화면 캡처 2021-11-10 140506](https://user-images.githubusercontent.com/62214428/141053228-ffb8b170-d763-4763-a9a7-748bbcc71c4b.png)

2. ★ 이 coreDns가 어떻게 사용되는지 확인해보자
![화면 캡처 2021-11-10 141505](https://user-images.githubusercontent.com/62214428/141054146-c6bdc47c-51dd-4d42-b650-86d758aa7652.png)
- `1번` = 현재 service 출력
- `2번` = 간단하게 pod 생성 
- `3번`★
   - `nslookup` = 리눅스의 기본적인 도메인,ip를 물어보는 명령어
   - 현재 생성된 파드에서 nslokkup을 통해 my-nginx 서비스에 접근 방법을 알아보면
   - 여기서!!★ server와 address를 보면 쿠버네티스 dns인 coreDns로부터 해당 정보를 받아온 것을 알 수 있다.(get svc -A 결과를 비교해서보면)
   - 생각해보자. 원래 dns가 루트 -> 최상위 도메인 ->... ->구체적인도메인까지 접근해서 정보를 받아오고 루트가 이를 나에게 전달해주는 것. 바로 위설명 또한 마찬가지
- 정리★
   - 우리가 찾고자하는것은 my-nginx서비스에 접근하기위한 정보입니다!
   - 그래서 루트에게 물어봤더니 `Name:   my-nginx.default.svc.cluster.local`이고 `Address: 10.100.187.202`라고 알려줬어요 (get svc -A에서 확인한 my-nginx 서비스와 동일한것을 확인가능)
   - 그리고 이 루트는 `10.96.0.10`ip를 가진 `kube-dns`이네요!

### 이게 어떻게 가능한가?
- `deployment`를 생성 후 `expose`를 통해 `my-nginx`라는 서비스를 생성하여 `deployment`와 연결
- 이런 정보들을 kube-dns(coreDns)와 계속 주고 받았기에 
- 나중에 임시적으로 파드하나 띄워서 nslookup으로 서비스 정보를 받아오는게 가능한 것
- 참고로
- ![화면 캡처 2021-11-10 142858](https://user-images.githubusercontent.com/62214428/141055358-f387bbf1-55e6-4634-ba2a-b96d96e03e75.png)
- `cat /etc/resolve.conf`는 리눅스에선 해당 dns서버에서 도메인,ip의 매핑관계 정보를 받아오겠다!
- `10.96.0.10` 어디서본 것 같은데? -> 바로 coreDns 



----------
## Ingress★
- `클러스터 내의 서비스★에 대한` 외부 접근을 관리하는 API 오브젝트이며, 일반적으로 HTTP를 관리함.
- ![화면 캡처 2021-11-10 173856](https://user-images.githubusercontent.com/62214428/141078974-c3d82978-14a8-4714-b671-255f6ced9853.png)
- ![화면 캡처 2021-11-10 174002](https://user-images.githubusercontent.com/62214428/141079122-049fd2fa-e517-4f17-9df3-a2a46bed9f28.png)
- `service` 중 `ClusterIp`는 클러스터내에서의 통신을 위한 서비스로 마스터노드는 클러스터의 구성원이라 curl로 접근가능하지만 크롬으로 해보면 당연히 안된다.  

### 외부에서 Http로 서비스에 접근할 수 있도록 Ingress를 설정해보자
- `minimal-ingress.yaml`
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: my-nginx
            port:
              number: 80
              
```
- ingress는 http를 대상으로 로드밸런싱
- ★path = http~/testpath에 대해서 !!
- ★service/name = my-nginx 서비스로 로드밸런싱하겠다!!

### ★그런데 지금 이 yaml이 로드밸런싱을 하는게 아니다. 이건 로드밸런싱 정보
### ★진짜 로드밸런싱을 수행하는 것은 Ingress Controller
- ![화면 캡처 2021-11-10 175930](https://user-images.githubusercontent.com/62214428/141082272-9b6877dc-7fa3-4631-8017-a8b3c7f57e0d.png)

--------

## Ingress Controller
#### 1. 설치하기
  - https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters
  - kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.4/deploy/static/provider/baremetal/deploy.yaml를 실행
#### 2. 확인하기
  - nodePort를 사용
  - ![화면 캡처 2021-11-10 181027](https://user-images.githubusercontent.com/62214428/141084090-27d348e6-0bf4-4ff8-b742-109e5b720aab.png)
  - ![화면 캡처 2021-11-10 181203](https://user-images.githubusercontent.com/62214428/141084334-555f11e4-7821-4bfa-bb8a-2f43f0d26d3e.png)

#### 3. 정리하고 가보자
  - deployment로 pod 생성했고
  - expose를 통해 my-nginx서비스 생성 후 pod와 연결했고
  - Ingress를 수행할 Ingress Controller 설치완료
  - Ingress정보를 위한 `minimal-ingress.yaml`작성 완료
  - 그러면 이제 Ingress Controller에 minimal-ingress.yaml을 넣으면 yaml에서 지정한대로 외부에서도 ~/testpath로 접근하면 my-nginx서비스로 로드밸런싱이 될 것


#### 4. 그럼이제 minimal-ingress.yaml을 적용해보자
1. 적용
- ![화면 캡처 2021-11-10 181822](https://user-images.githubusercontent.com/62214428/141085338-f1e8322e-6b43-4dd3-b2f1-71a8d1214765.png)
- kubectl apply -f 를 통해 ingress 설정 정보인 minimal-ingress.yaml을 적용했다.

2. 확인
- 그런데 생각해보면 어디로 접근했을 때 ingress가 수행되는걸까?
- 앞에서 Ingress Controller가 `nodePort`를 사용한다고 했다
- `nodePort`는 클러스터내의 `어떤 노드의 IP든` 특정 포트로 들어오면 해당 서비스로  로드밸런싱해준다고 했다
- 그러니까 마스터n이든 워커n이든 특정 노드로만 접속하면 된다는건데

3. 접속
- ![화면 캡처 2021-11-10 182443](https://user-images.githubusercontent.com/62214428/141086356-1e2cbad7-47cc-4f4b-a7a2-f3151c3bd14c.png)
- 위에서 말한 특정 포트가 32261이라는것을 확인★
- 그럼 일단 마스터노드1 ip:32261로 접근해보면
- ![화면 캡처 2021-11-10 182553](https://user-images.githubusercontent.com/62214428/141086515-4b941df0-67b3-4ae0-8dd0-6b4645d6541e.png)
- 워커1 ip:32261로 접근해보면
- ![화면 캡처 2021-11-10 182731](https://user-images.githubusercontent.com/62214428/141086767-293504b9-5498-4c18-9db5-d4c76d184cde.png)
- 접근이 되는것을 확인했다 ★
- 그런데 원한건 nginx페이지이고 우리가 놓친게 있지 않은가?
- 바로 `/testpath`
- 그런데 /testpath해도 안된다??
- ![화면 캡처 2021-11-10 183432](https://user-images.githubusercontent.com/62214428/141087884-d0da7361-87ca-4430-b647-1219719b42b7.png)

4. 해결
- https://kubernetes.github.io/ingress-nginx/
- ![화면 캡처 2021-11-10 183542](https://user-images.githubusercontent.com/62214428/141088088-1bcb06c2-309a-4b1f-a11b-205eeed26103.png)
- 마침 이전에 쿠버네티스를 v1.22로 업그레이드 했었고 해당 버전에서는 ingressclass가 꼭 필요하다고 한다.
- 그리고 우리는 ingress controller가 1개인 상황이고 이 상황에서 공식문서는 아래를 생성하길 추천한다고한다.
```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    app.kubernetes.io/component: controller
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
```
- ![화면 캡처 2021-11-10 184332](https://user-images.githubusercontent.com/62214428/141089453-036246bf-6b71-4295-9437-5b5116dfe574.png)
- 여기서부터 집중 ★
- 1번 = `ingressClass.yaml`생성 후 apply한다.
- 2번~3번 = 이전 설정정보로 적용한 minial-ingress를 지우고
- 4번 = 다시 ingress 설정정보를 생성하여 ingressController에게 넣는다.

5. 재확인
- 그럼 이제 /testpath로 접근해보면
- ![화면 캡처 2021-11-10 184533](https://user-images.githubusercontent.com/62214428/141089777-9a06182f-012f-47da-ba7f-0172cbe0c287.png)

6. 정리
- 1. deployment로 pod를 생성
- 2. expose로 서비스(my-nginx) 생성 후 pod와 연결
- 3. ingress를 위해 ingressController설치
- 4. 버전 이슈로 ingressclass 적용
- 5. ingress 설정정보 (minimal-ingress.yaml)생성 후 적용
- 6. 결과적으로 어떤 노드로 접근하든 http로 특정포트(get svc -A로 확인)로 접근하는 경우 ingress 설정 정보에 따라 특정 service(my-nginx)로 로드밸런싱하고 이 서비스는 특정 파드(deployment로만든)로 로드밸런싱하여 마지막에 우리는 nginx가 동작하는 pod를 ★외부에서 접근할 수 있게되었다
- 외부 ->ingress -> service ->pod 
















