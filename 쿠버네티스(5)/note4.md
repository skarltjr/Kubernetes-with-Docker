### 클러스터 노드 작업
![화면 캡처 2021-10-28 203018](https://user-images.githubusercontent.com/62214428/139247416-6cae8a30-675b-4f83-ba9a-39ebe8b03075.png)


### 1. cordon
```
지금까지 A노드에 파드가 있는 건 괜찮아 ok
근데 앞으로 새로 만들어질 파드는 A노드에 배치하지마! 스케쥴링 하지마!
```
- `kubectl cordon 노드명`
- ex) 'kubectl cordon nks-worker-01.kr-central-1.c.internal`  => `node/nks-worker-01.kr-central-1.c.internal cordoned`
- ![화면 캡처 2021-10-28 204041](https://user-images.githubusercontent.com/62214428/139248800-a9c83c6d-3347-4210-a89f-debaf50351a4.png)
```
cordon 명령어로 워커노드 1번을 cordon했다.
describe로 살펴보니 taint에 noschedule이 추가된 것!
그러니 toleration을 추가하지 않은 일반적인 모든 노드들은 워커1번에 스케쥴링 되지 않는다.
```
- 해제 = `kubectl uncordon 노드`
### 2. drain
```
현재 노드에 문제가 있으니 이 노드의 파드들을 다른곳에 옮기자!
당연히 먼저 cordon을 수행시켜 고장난 이 노드로 스케쥴링 되지 않도록 막고
그 다음 파드들을 옮기는, 빼내는 drain수행
```
- ![화면 캡처 2021-10-28 204234](https://user-images.githubusercontent.com/62214428/139249043-f4b439c9-cf9c-4d92-8cd6-cd481a560492.png)
- 현재 1번노드에 pod가 존재
- `kubectl drain 워커노드 1번`
- ![화면 캡처 2021-10-28 204604](https://user-images.githubusercontent.com/62214428/139249577-ab0f77b6-2fbd-4ccf-a662-acff41f1b393.png)
- 에러에 맞춰
- `--ignore-daemonsets` => 추가 / 참고로 `daemonset`은 모든 노드에 하나씩 추가되는 노드
- `--delete-emptydir-data` => 추가
- `--force` 추가
- 그런데 세팅이 필요한것들을 drain하게되면 pod가 사라지기도 한다.. 사라져버림

### 2-1. 왜 사라졌는지 다시 에러를 제대로 확인해보자
```
WARNING: deleting Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet: default/test-pod; 
ignoring DaemonSet-managed Pods: kube-system/calico-node-8lwlj, kube-system/kube-proxy-wk4lp
```
- `pod`를 만들 때 `run`으로 만들었는데 이는 당연히 레플리카셋에 관리를 받는다 / yaml로도 마찬가지
- 위에보면 deployment는 빠져있는데
- deployment로 만들어보자
- ![화면 캡처 2021-10-28 210339](https://user-images.githubusercontent.com/62214428/139251960-5fa3c3b3-ec81-47d5-8cc0-cc9a5fe4863f.png)
- `deployment`로 관리할거야! 그런데 레플리카셋은 3개로 / 확인
- 이제 다시 `drain`을 해보자
- ![화면 캡처 2021-10-28 210633](https://user-images.githubusercontent.com/62214428/139252286-5c01064a-82a6-43f1-ad54-07c63e866179.png)
  - 1번에서 워커1번에 위치한 pod2개 확인
  - 2번에서 1번워커 drain 수행
  - 3번에서 --ignore-daemonsets추가
  - 4번에서 올바르게 잘 옮겨진 것을 확인
  - 마무리로 다시 1번 uncordon

#### run으로 만든 pod는 지워졌지만 deployment 단위로 만든 pod는 잘 drain되었음을 확인했다









