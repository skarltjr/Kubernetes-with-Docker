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
- `--ignore-daemonsets` => 추가 / 참고로 `daemonset`은 모든 노드에 하나씩 추가되는 노드
- `--delete-emptydir-data` => 추가
- 그런데 세팅이 필요한것들을 drain하게되면 pod가 사라지기도 한다.. 사라져버림










