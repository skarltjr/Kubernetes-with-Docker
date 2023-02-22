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
deployment로 관리하지 않는 pod
```
- ![화면 캡처 2021-10-27 232927](https://user-images.githubusercontent.com/62214428/139086316-9bfb4fe9-e98a-4ef2-819c-2d391b2fcd45.png)
  - 1번을 통해 접근
  - 2번의 여기 이 4개의 yaml로 생성된 pod들이 바로 `static pod`
- ![화면 캡처 2021-10-27 233229](https://user-images.githubusercontent.com/62214428/139086909-486d2b0b-660b-4e78-9af7-8c62e1dff12b.png)
  - 1번 / 2번 / 3번 / 4번 애들이 바로 그 `static pod`들
  - 5번을 봐보자 
  - static pod들은 다른 애들과 다르게 뒤에 이상한 이름이 붙지 않는다.

### 3-1. 왜 필요해?
- control plane이 관리한다는것은 결국 의도치않은 삭제가 발생할수도있다는것
- 이런 문제점으로 인해 정적으로 관리한다고 생각.


### 4. 도대체 static pod들은 어떻게 관리되는건가?
- `kubectl get deployment -A`
- `kubectl get replicaset -A`
- 을 해봐도 앞선 1,2,3,4번을 관리하는 애들은 없다
- ![화면 캡처 2021-10-27 233451](https://user-images.githubusercontent.com/62214428/139087463-e97db610-ef3b-410a-bab8-032e75ea6250.png)

#### 이 static pod들은 yaml로 관리된다
- `etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml` 이 yaml들로 관리되는 것
- `/etc/kubernetes/manifests`에 존재


### 5. 이 static pod들은 yaml을 통해 kubelet이 관리를 한다
- etcd.yaml을 봐보자
- 권한이 필요해서 sudo vi etcd.yaml
- 이 `static pod`들은 kubelet이 `정적으로`관리


### 6. /var/lib/kubelet/config.yaml  -> kubelet을 살펴보자
- kubelet이 이 static pod들을 관리한다고 했다
- ![화면 캡처 2021-10-27 234409](https://user-images.githubusercontent.com/62214428/139089110-4d83be56-b2a2-4bcb-93f8-072b3985622d.png)


### 7. 정말 중요한 etcd.yaml
- `etcd`는 쿠버네티스의 정보를 담기에 매우 중요한데
- 우선 /etc/kubernetes/manifests에서 etcd.yaml을 확인해보자
- ![화면 캡처 2021-10-27 235814](https://user-images.githubusercontent.com/62214428/139091934-0f730f7b-00e7-4735-8907-19ccb8dc5052.png)
- 표시한 부분이 왜 중요한지 살펴보자

#### ex) 운용도중 문제가 생겼다. 그런데 이를 대비해서 어제까지의 정보가 담긴 백업파일이 있다.
- 이 상황에서 쿠버네티스의 모든 정보를 담은 etcd를 백업파일의 etcd로 갈아끼워야한다.
- 그러니까 지금 문제가 있는 우리의 `etcd`에서 -> 백업 `etcd`로 갈아끼우고자 하는데
-  그 설정이 바로 위 그림에서 표시한 부분을 바꾸는 것
-  즉 /var/lib/etcd =  현재의 etcd인 etcd가 1개 있고
-  /var/lib/etcd-backup 이라는 etcd백업파일이 있는 총 두 개의 etcd가 있는 상황
-  1. 현재 파드 etcd의 path를 백업용으로 갈아주기 위해 hostPath의 path를 `/var/lib/etcd-backup으로 변경
-  2. ★★★ `총괄 etcd 파드`가 1번 변경내용을 알기 위해서 path를 백업용으로 갈아주기 위해 volumeMounts의 mountPath를 `/var/lib/etcd-backup으로 변경 = > 정말중요



### 8. static pod를 만들어보자
`kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 100 > static-busybox.yaml`
- `command` = pod를 실행할 때 이 해당 pod에서만 동작하는 명령어를 추가하고싶다
- 참고로 `--restart=Never` -> static pod니까
- ![화면 캡처 2021-10-28 001232](https://user-images.githubusercontent.com/62214428/139094641-abce95a8-bd07-4004-a009-4abe4fa0fa47.png)
- describe로 자세히 살펴볼 수 있다
- ★★ 여기서 이 `static-busybox.yaml`을 `/etc/kubernetes/manifests에 cp만 해도 어떤일이 생길까?
- ![화면 캡처 2021-10-28 002002](https://user-images.githubusercontent.com/62214428/139095919-e4130bd7-0b05-4242-970d-6af56dc0bfb1.png)
- 2번처럼 cp만 했는데도 1,3번을 비교해보자 -> 새로운 `static pod`가 생겼다

### 9. 그래서
- static pod를 만들 때 위처럼 cp를 해서 만들고
- 지울때도 `/etc/kubernetes/manifests`에서 `rm`으로 지우면 pod도 삭제된다
-  ![화면 캡처 2021-10-28 002311](https://user-images.githubusercontent.com/62214428/139096528-9c235402-6586-40fd-a5e3-416c2fc03ac0.png)
```
1번처럼 이동해서
2번 -> rm으로 yaml을 지웠는데
3번 -> static pod가 삭제되었다!
```
