### 클러스터 업그레이드
- 앞서 왜 cordon / drain 같은것들을 배웠을까?
- 왜 이런것들이 필요할까?
- 바로 클러스터를 업그레이드할 때 필요하기 때문
- 노드를 업그레이드 하려는데 pod가 생성되거나 하면 안되니까 



### 일단 버전을 확인해보자
- `kubectl version`
- `kubeadm version`
- 나는 지금 둘 다 v1.21.0
- 이제 업그레이드를 해보자
- https://v1-21.docs.kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/



### 1. master 1번 업그레이드
- 일단 pod 다 빼놔야한다 drain 
- `kubectl get pod -A`로 확인해보면 master 1번에서 수행중인 pod들이 있을 것 
- `kubectl drain nks-master-01.kr-central-1.c.internal --ignore-daemonsets`
- 이제 마스터1번 다 비웠으니 업그레이드를 진행하자
```
[1번 노드] #sudo yum	list	--showduplicates kubeadm --disableexcludes=kubernetes   / 받을 수 있는 리스트를 확인
[1번 노드] #sudo yum	install	-y	kubeadm-1.22.1-0	--disableexcludes=kubernetes   / 1.22.1-0을 받고 complete!

[1번 노드] #kubeadm upgrade	plan	                                              /-> 그 전에 root로 sudo su -
→ preflight 수행하여 클러스터가 정상인지 확인, kube-system kubeadm-config 컨피그맵 검사
→ 앞으로 업그레이드하는게 좋을지 알려줌 (root 계정으로 실행)

[1번 노드] #kubeadm upgrade	apply	v1.22.1
 // [1번 노드] #kubectl uncordon master01번 노드 -> 지금 x 마지막에 한 번에
```
- SUCCESS! Your cluster was upgraded to "v1.22.1". Enjoy!
- 1번 uncordon했는지 확인해라!!

### 2. master 2~3번 업그레이드
- 일단 2번 마스터에 접속 ★ // 당연히 1번에서하면 안된다
- 2번도 일단 `drain` ★
- 이 때 `kubectl get node`로 이름 확인하고 // -> 포트얘기가 나오는데 root로 
- nks-master-02.kr-central-1.c.internal 를 drain한다
- 이 후 업그레이드 하자 !! ★


```
// 여전히 루트계정 
#sudo yum	list	--showduplicates kubeadm --disableexcludes=kubernetes       // 리스트 한 번 확인해보고
[2,3번 노드] #sudo yum	install	-y	kubeadm-1.22.1-0	--disableexcludes=kubernetes
[2,3번 노드] #kubeadm upgrade node                     // 이 부분이 1번과는 좀 다르다 -> 공식문서 참고
// [2,3번 노드] #kubectl uncordon master02번     -> 지금 x 마지막에 한 번에
```
- 2번 끝나면 !!!! ★ 3번을 진행 -> 동시에 할 필요가 없다. 천천히 하자.... 

### 3. master 1~3번 kubelet kubectl 
```
[1~3번 노드] #sudo yum	install	-y	kubelet-1.22.1-0	kubectl-1.22.1-0	--disableexcludes=kubernetes
[1~3번 노드] #sudo systemctl daemon-reload	
[1~3번 노드]	#sudo systemctl restart	kubelet
[1~3번 노드] kubectl uncordon osk-master-01.kr-central-1.c.internal	osk-master-02.kr-central-1.c.internal	os
k-master-03.kr-central-1.c.internal  // 모든 마스터 이제 업그레이드 다 완료했으니 uncordon
```













