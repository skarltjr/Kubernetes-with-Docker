### 1. Namespace
- 쿠버네티스는 동일한 물리 클러스터를 기반으로 하는 여러 가상 클러스터를 지원, 이런 가상 클러스터가 네임스페이스
- `구분 단위`
- `	kubectl get	pods	-A`로 파드들의 네임스페이스를 확인할 수 있다.
- `kubectl get namespaces`를 하면 현재 존재하는 namespace들을 확인할 수 있다

### 새로운 구분단위 Namespace를 만들고 namespace를 지정해서 pod를 만들어보자
1. namespace 생성
- `kubectl create namespace kiseok`
- ![화면 캡처 2021-10-27 230828](https://user-images.githubusercontent.com/62214428/139082435-8b6f5118-3d2d-4282-95a2-118a5d35b417.png)

2. pod를 생성 / 이 때 namespace를 지정
- `kubectl run pod이름 --image=nginx --namespace=네임스페이스이름`
- ex) `kubectl run likelion --image=nginx --namespace=kiseok`
- 참고로 -n으로 대체가능

3. kubectl get pod를 하면 생성한 pod가 나올까? ★
- 해봐라 
- 안 나 온 다
- 왜?
- namespace를 지정해줘야한다
- ![화면 캡처 2021-10-27 231114](https://user-images.githubusercontent.com/62214428/139082907-057c9d58-f1c3-4dd9-93d9-0a350ef12c11.png)
### ★ namespace를 지정해주지 않으면 default namespace에 해당된다. 다른 동작도 마찬가지
- 지울때도 ex) `kubectl delete pod -n kiseok likelion`


### 2. Label과 Selector
- 앞에서 `label`로 쿼리가 가능하다고 했다 ★
- 즉 `label`로 특정한 것들만 뽑아낼 수 있다는 것
- 해보고자 하는 것  = pod를 생성하되 label을 부여하고 이 라벨을 통해 해당 pod를 get해보자
1. pod를 생성
- `kubectl run label-pod --image=nginx -l teacher=osk --dry-run=client -o yaml > label.yaml`
- label-pod라는 파드를 만들거야
- `-l` = label옵션으로 teacher = osk라는 라벨을 붙일거야
2. yaml확인
- ![화면 캡처 2021-10-27 231941](https://user-images.githubusercontent.com/62214428/139084453-eb590cb7-8e45-4afe-b893-1201623ab618.png)

3. `kubectl get pod --selector 키 값`
- ex) `kubectl get pod --selector teacher=osk`
- ![화면 캡처 2021-10-27 232112](https://user-images.githubusercontent.com/62214428/139084828-a013dcd5-52df-49cc-b53d-a45ee6dcece0.png)
- 즉 selector를 통해 label이 teacher=osk인 pod만 뽑아올 수 있었다.★
- 참고로 `kubectl get pod -l teacher=osk`로 -l 라벨 옵션으로도 동일하게 동작

4. `kubectl get all -l 키 값`
- 3번에서는 get pod로 해당 label을 가진 pod만 가져왔는데
- `all`은 pod뿐만 아니라 전체 오브젝트에서 해당 label을 가진 모든 것들을 가져온다

### 3. Staticpod
- Static pod(Manifest)는 `kubectl 및 control plane`에서 관리하지 않는 pod
- pod spec을 디스크에 직접 쓸수 있으며, kubelet은 시작과 함께 바로 지정된 컨테이너를 시작
- Kubelet은 지속적으로 변경사항이 있는지 Manifest 파일을 지속적으로 모니터링
```
즉 control plane에서 관리하지 않기 때문에
지워져도 replicaset이 다시 생성한다거나 그러지 않는..
deployment로 관리하지 않는

pod
```







