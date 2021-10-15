### 쿠버네티스 명령어와 yaml
- 쿠버네티스의 모든 명령어가 다 잘 먹는건 아니다.
- 그래서 종종 파일로 관리할때가 유리할 수 있다
```
kubectl 명령어 종류 
ex)
kubectl get 
kubectl delete
kubectl run 
...
```

#### 1. POD생성 - 명령어로 파드를 생성하기
ex)
```
kubectl run test --image=nginx

run = 파드를 생성하겠다
test = 파드의 이름
--image = 파드를 생성하려면 이미지가 필수
```

#### 2. POD생성 - yaml로 생성하기
```
kubectl run test --image=nginx --dry-run=client -o yaml > kiseok.yaml

그러면 파드가 생성되는게 아니라 해당 파드를 생성할 수 있는 kiseok.yaml이 생성된것
이를 통해

kubectl apply -f kiseok.yaml
```

- yaml살펴보기
- ![화면 캡처 2021-10-15 181316](https://user-images.githubusercontent.com/62214428/137463492-8a5898e3-1ed7-40b4-b748-b83951140a2f.png)
- `중요한 점` 우리는 이 yaml을 조작할 수 있다
- 그래서 명령어보다 좀 더 세심하게 설정한 후 이를 `apply`해서 `pod`를 생성할 수 있다.


#### 3. POD 조회 -   -o wide로 자세히 살펴보기
![화면 캡처 2021-10-15 182032](https://user-images.githubusercontent.com/62214428/137464454-18ab6467-b748-4cfa-9d59-7eba87cdedc5.png)
- `kubectl get pod -o wide` 옵션을 추가해서 더 자세히 알아볼 수 있다
- 보면 자동으로 알아서 `워커 노드에 pod를 생성`한 것을 볼 수 있다

#### 4. POD조회 - describe 명령어를 통해 하나의 pod 자세히 확인하기
- 하나의 pod를 자세히 살펴보기 위해선 `describe`명령어를 사용한다
- ![화면 캡처 2021-10-15 182442](https://user-images.githubusercontent.com/62214428/137465012-aaa5a8d7-bf17-4930-86c8-fe148f7b481f.png)


#### 5. POD의 로그를 확인하자
- `kubectl logs 파드이름`
- ![화면 캡처 2021-10-15 182555](https://user-images.githubusercontent.com/62214428/137465177-7da29297-2bff-4079-ae02-427fd54e433f.png)


#### 6. POD에 접속하자
- 도커 컨테이너에 접속할 때 `exec -it`를 생각해보자
- `kubectl exec -it pod이름 -- /bin/sh`
- pod에 접속할게 그런데 bash는 기본 /bin/sh를 사용할게 / 참고로 --를 안붙이면 이제 붙이라는 말을 해준다
- 마찬가지로 `exit`로 나간다
![화면 캡처 2021-10-15 183041](https://user-images.githubusercontent.com/62214428/137465853-c8b4b098-6298-472d-8def-1e2227113c30.png)


-------------

### 쿠버네티스 리소스
![화면 캡처 2021-10-15 183758](https://user-images.githubusercontent.com/62214428/137466875-1922ecc0-5eca-4e48-ac24-62f183f23958.png)
- 실제로 openVPN을 활용하여 우리는 기존에 인터넷을 향하는 인터페이스 + `카카오망을 향하는 인터페이스를` 추가 
- 라우팅까지 설정을 이미 해줬던 것 




### CNI : 컨테이너 네트워크 인터페이스
- `컨테이너 네트워크 인터페이스(CNI)`
- 네트워크로 연결될 파드는 동일 노드에, 다른 노드에 있을 수도 있음. CNI의 역할은 단순히 파드간 연결을 용이하게 만드는 것
   - A 파드는 `워커 노드 1`에 위치하고 B파드는 `워커 노드 2`에 위치할 수 있다. 이 파드들의 연결을 용이하게 해주는 것 CNI ( 우리는 여기서 calico를 사용 )

-  컨테이너 런타임(예:도커)은 CNI 플러그인 실행파일(예:칼리코)을 호출하여 컨테이너의 네트워킹 네임스페이스에 인터페이스를 추가/제거
- ![화면 캡처 2021-10-15 184349](https://user-images.githubusercontent.com/62214428/137467696-9afebf1f-e4f7-4f8c-92ff-79645d76ba23.png)
- `왼쪽 그림`을 보면 알 수 있듯이 서로 다른 노드에 위치한 파드끼리의 연결을 담당하는게 CNI
- ★`오른쪽 그림`을 보자 
     - 먼저 `etcd`와 붙어있는 `삼색고양이 = calico`
     - `etcd`는 쿠버네티스의 모든 내용을 저장한다 / CNI는 이 etcd와 소통하면서 연결정보를 구성
     - 이를 통해 `마스터노드`와 아래의 2개의 `워커노드`가 서로 소통  / 워커노드에도 `CNI = calico`가 있기 때문에  

### ReplicaSet - 컨트롤러 오브젝트
- 우리는 `선언적 구성`을 한다. => 3개를 유지시켜줘! => 3개 중 2개가 삭제되면 알아서 3개로 다시 맞춰준다
- 이걸 수행하는 것이 바로 `replicaset` 
- 노드가 고장 / pod삭제 등 발생 시 파드를 복제하여 개수 유지


### Deployment - 컨트롤러 오브젝트
- `ReplicaSet`관리  - (보통 pod를 일일이 관리하기보다는 deployment 를 서비스 단위로 관리)
- 구 버전에서 신 버전으로 복제
- `Deployment`와 `ReplicaSet`이 유기적으로 소통하는데
- 아래 그림을 보자!! 
- `Deployment`는 name = myapp / replicas = 3   => 그래서 pod를 3개 생성하는데
- ★ 이 파드들을 `Deployment`가 `ReplicaSet`한테 부탁해서 `ReplicaSet`아 pod를 3개 유지해줘!
![화면 캡처 2021-10-15 203321](https://user-images.githubusercontent.com/62214428/137480525-9888030a-9fee-48fd-bca0-eefa83e9a464.png)

### Deployment 명령어
![화면 캡처 2021-10-15 204353](https://user-images.githubusercontent.com/62214428/137481667-f8a9f375-a82e-47ce-9f70-6cb9a06b1f83.png)
- 먼저 `3번`을보자 `deploy-test`라는 이름 +  `replicas는 3개`로  `delpoyment`생성
    - 여기서 봐야할 것은 3개의 파드를 하나씩 `run`으로 만든게 아니라 deployment로 한 번에 관리 이게 (보통 pod를 일일이 관리하기보다는 deployment 를 서비스 단위로 관리)의 의미
- `4번, 5번`으로 `deployment` get / 확인
-  `6번` => 앞에서 replicas=3이라고 했기 때문에 당연히 pod 3개 만들어진것을 확인
-  7번 `3개의 파드 중 하나를 지웠다!!` . 분명히 지웠다
-  `8번` => 그런데 다시 get pod 해보니 그대로 3개???
-  ★바로 replicaset이 동작한 것 => replicas = 3을 유지하기위해 다시 생성해준 것 => 시간을 확인해보면 pod 중 15s전에 태어난게있다

