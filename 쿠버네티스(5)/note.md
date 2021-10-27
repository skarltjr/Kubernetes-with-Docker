### [Compute] Managing State With Deployments, Scheduling

-----  

### 1. 복습해보기
### 1-1. pod / replicaset
![화면 캡처 2021-10-27 172453](https://user-images.githubusercontent.com/62214428/139028631-e794bc9b-5869-4e17-a5fe-f29bee8abdd2.png)
```
고민해보기
deployment로 레플리카 셋을 만들어서 파드가 현재 3개인 상황

여기서 만약 deplouyment를 지우면 레플리카 셋, 파드들은 어떻게?
```
![화면 캡처 2021-10-27 172654](https://user-images.githubusercontent.com/62214428/139028981-6aa1ec74-bf13-4351-a9d7-139dae09381c.png)

- `deployment`로 다뤄진 `replicaset`과 `pod`들은 `deployment`가 delete되면 모두 함께 delete된다.

### 1-2. deployment
```
kubectl create deployment kiseok_deployment --image=nginx --dry-run=client -o yaml > deployment.yaml
-> kiseok_deployment라는 이름의 deployment를 만들게요
-> 그런데 이미지는 nginx쓸거구요
-> 바로 생성하지 않고 yaml로 뺄거예요
```

--------------

### 2. Scheduler
![화면 캡처 2021-10-27 173639](https://user-images.githubusercontent.com/62214428/139030477-cb1f7770-07f4-4ec3-b82c-9c2070f0487e.png)
```
파드가 생성되는 과정을 살펴보자
1. 파드를 생성하라는 명령어가 api server를 통해 전송된다
2. etcd에 이 내용을 기록
3. `Scheduler`가 나설 차례.
   - 어떤 노드에 파드를 생성해야 올바를까? 결정
   - 3번 워커노드가 좋겠어!
4. 각 노드의 kubelet들은 apiserver를 쭉 지켜보다가
   - 3번 워커노드의 쿠블렛은 아! 내꺼네 생성해야겠다
```

### 2-1 스케쥴링 프로세스
```
스케줄러가 파드를 배치하는 주요 기준
1) 사전 조건
- 파드가 특정 노드에 적합한지 확인.
- ex) 파드에서 요청한 메모리 양을 노드가 감당할 수 있을지
- 혹은 유저가 node selector로 지정한 경우는 지정한 노드에 생성

2) 우선순위
- 파드가 노드에서 실행할 수 있다고 가정 후, 상대적 가치를 판단
- ex) 이미지가 이미 존재하는 노드에 가중치를 부여 : 빠른 시작 가능
- 동일한 Service 내 있는 파드라면 spreading 시켜놓음 = 한 노드에 몰리지 않도록 여러 노드에 분배하자!

※ 스케줄링은 한 순간에만 최적이므로, 이후 충돌 등 여러 이유로 실행중인 파드를 이동할 수 도 있음. 이 경우 해당 pod를 삭제하고 재 생성
즉 파드가 이미 실행되었는데 다른 곳에 옮기는 것이 효율적이라면 
삭제하고 다른곳에서 생성시킬 수 있다. 누가? 스케쥴러가
```

### 3 policy 정책
- `스케쥴러`의 방식말고 내가 직접 스케쥴링을 하고 싶은 경우
![화면 캡처 2021-10-27 174757](https://user-images.githubusercontent.com/62214428/139032249-131beb98-fc0d-4e5b-82a5-1080b7f33af1.png)
```
1. node selector
  - ex) 난 disktype이 ssd인 곳에만 pod를 생성할거야
  - 명시적으로 어디만 가능한지 표기
2. node affinity(선호)
  - ex) 난 A,B 둘 다 가능한데. A를 선호해 되도록이면 A에 만든다
 
우선 여기서 그림을 잠깐 보면 기본적으로 마스터 노드로는 안보낸다.
 
그런데!!
3. Taint/toleration
마지막 3번을 보면 Toleration : node-role.kubernetes.io/master:NoSchedule라는  라벨이 들어감
이 친구는 toleration. 마스터에 보내도 상관이 없다 나는~ 
그래서 마스터노드로도 화살표가 있다. 가능하다

일반적으로 taint(제약)을 걸어놓으면 스케쥴러는 그 쪽으로 배포를 안한다.
여기서는 그 예시로 마스터노드가 taint를 걸어놓은 것
그런데 toleration으로 나는 상관없다~ 아무곳에나 보내라 -> 제약에 상관 x
그래서 master노드로도 갈 수 있다라는 설명
참고로 taint는 워커 노드에도 설정할 수 있다
```

### 3-1 policy 자세히 살펴보기 - Node selector
1. node selector
![화면 캡처 2021-10-27 180041](https://user-images.githubusercontent.com/62214428/139034421-9220e9bc-2b8a-4f18-85bd-1e44595c73a0.png)
```
label - 라벨을 추가하겠다
kubernetes-foo-node-1.c.a-robinson.internal - 이라는 노드에
disktype=ssd - 라는 라벨을
```

2. 노드에 label추가
![화면 캡처 2021-10-27 180536](https://user-images.githubusercontent.com/62214428/139035211-7cc4eca5-12fc-430e-a007-29b295cb8bbe.png)

```
1. 노드들 확인 / 이름 확인
2. 워커노드 1에 라벨 추가
3. describe로 해당 노드 자세히 살펴보면
4. label이 추가되었다!
```

3. 새로 생성할 파드에 label을 지정해주면 진짜 위 노드(1번워커)에 생성될까?
- 그러기 위해 우선 pod생성 할 yaml 생성
- ![화면 캡처 2021-10-27 181110](https://user-images.githubusercontent.com/62214428/139036157-e1799b77-7250-4e89-bc5b-73db29763c33.png)
- yaml을 수정해서 label을 추가해주자
- ![화면 캡처 2021-10-27 181353](https://user-images.githubusercontent.com/62214428/139036682-3a7144f3-205e-4a32-a98e-bff3f729e113.png)
- 이제 진짜 pod를 생성해보고 결과를 확인해보자 
- 과연 워커노드 1에 배치될까?
- ![화면 캡처 2021-10-27 181615](https://user-images.githubusercontent.com/62214428/139037094-27be6bc8-8395-4d35-b914-0ce63bd60428.png)
- 짜잔~
#### 참고로 라벨은 구분을 위한 그저 키/쌍 => 이걸로 무슨 데이터를 지정하거나 하는게 아니다

### 3-2 policy 자세히 살펴보기 - Node Affinity
- `Node Selector`는 선택에 대한 요구사항을 이야기하는 사전 조건 
- 복잡한 선택에는 Affinity가 유연. (ex) 레이블 foo이면 A 또는 B 노드를 선택해주세요 - `유연함`

1. pod를 생성할 yaml을 만들어보자
![화면 캡처 2021-10-27 184217](https://user-images.githubusercontent.com/62214428/139041454-10074c5d-06a8-4189-8b9f-24febba4cff8.png)
- `affinity`를 설정
- 여기선 nodeSelector가 없다
- 그런 상황에서 
- `key: disktype`이 `values:ssd`인 쪽을 선호한다고 명시해준 것!!
- 그럼 어디에 생성될까?
- ![화면 캡처 2021-10-27 184238](https://user-images.githubusercontent.com/62214428/139041709-563b372b-5ad7-425c-ac11-1fe1ca752dac.png)
















