## 백업 및 복구
- 쿠버네티스 클러스터의 정보들을 갖고 있는 etcd.
- Etcd 문제 발생 시, 쿠버네티스 클러스터 전체가 영향을 받게 됨
- `sudo rm -rf /var/lib/etcd`하면 이제 진짜 큰일나는것 
- https://github.com/skarltjr/Kubernetes-with-Docker/blob/main/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4(5)/note3.md 하단 부분을 참고하자

#### etcdctl
- Pod로 구동된 etcd 버젼 확인 후 해당 etcdctl 설치 (https://github.com/etcd-io/etcd/releases)
- 버전 확인은 
- ★오픈소스는 버전 정말 잘 확인해야한다. 확인 제대로 안하면 .... ㅎ
```
1. kubectl get pod -A 로 etcd pod 확인
kube-system     etcd-nks-master-01.kr-central-1.c.internal 확인

2. kubectl describe -n kube-system pod etcd-nks-master-01.kr-central-1.c.internal로 etcd pod 버전확인
containers의 image -> Image: k8s.gcr.io/etcd:3.5.0-0    // 3.5.0-0 확인
```
- 이 후 사진처럼 진행
- ![화면 캡처 2021-11-13 135440](https://user-images.githubusercontent.com/62214428/141606156-b786702a-fbea-4580-82e9-3cb04dbe80cb.png)



### 백업
- port는 위에서 etcd pod describe로 확인가능
```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot save /tmp/snapshot-pre-boot.db
```


### 복구
- 가장 중요한 부분은 바로 
- ` --data-dir /var/lib/etcd-from-backup \`
- https://github.com/skarltjr/Kubernetes-with-Docker/blob/main/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4(5)/note3.md 에서 말한 그 복구받을 etcd파일의 경로를 지정하는건데
- ★ 만약 이거 /varlib/etcd로 해버리면 기존에 망가진 etcd파일에 덮어씌우니까 복구도 못하게된다 주의!!!
```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --name=master \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     --data-dir /var/lib/etcd-from-backup \
     --initial-cluster=master=https://127.0.0.1:2380 \
     --initial-cluster-token etcd-cluster-1 \
     --initial-advertise-peer-urls=https://127.0.0.1:2380 \
     snapshot restore /tmp/snapshot-pre-boot.db
```



















