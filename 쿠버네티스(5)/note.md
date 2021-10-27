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
```











