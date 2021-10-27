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
















