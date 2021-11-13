## Troubleshooting
#### 과제 
- pod생성이 막힌 상황에서 쿠버네티스 구조를 떠올리며 log를 통해 왜 생성이 안되는지 원인을 찾고 해결해보자
- ![화면 캡처 2021-11-13 151840](https://user-images.githubusercontent.com/62214428/141608361-ff6b3aa3-9122-4382-b02e-185a4617870b.png)
- 원인을 찾기 위해 describe 찍어보자
- ![화면 캡처 2021-11-13 152114](https://user-images.githubusercontent.com/62214428/141608475-59b5279e-9204-4907-a91a-d3ff7c68f932.png)
- 노드에 taint가 묻어있는 것 같다. 확인을 해보자
- 동작할 pod가 생성될 곳은 worker / worker describe해보자
-  ![화면 캡처 2021-11-13 154803](https://user-images.githubusercontent.com/62214428/141609152-95c0b8ab-e394-4361-8ad5-6cfd2100c1a2.png)
- 앉아서 6시간동안 해본 결과 나는 당연히 taint때문인줄 알고 taint delete를 여러번시도해봤지만 이게 아니였다
#### 해결법 - 
```
정말 중요한 내용이다. 
쿠버네티스의 구조를 pod 생성 과정과 함께 고려해보자 
1. api server를 통해 pod생성 요청을 받을 것이다
2. APIServer는 etcd에 Node에 할당되지 않은 Pod이 있음을 업데이트
3. Scheduler는 etcd의 변경사항을 API Server를 통해 watch하고 Pod을 실행할
 Node를 선택해 API Server에 해당 Node에 Pod을 배정하도록 업데이트
--- 
★여기까진 control plane에 대한것이다
결국 pod가 생성되는 곳은 워커노드
4. kubelet이 아 이 요청 내꺼구나 처리해야겠다 ★를 인지해야하는데!!!!
```
- ![화면 캡처 2021-11-13 203212](https://user-images.githubusercontent.com/62214428/141642252-9222c85e-fe0e-48c6-b907-7f15c86ce295.png)
- 문제의 원인을 드디어 찾았다.
- ★ 클러스터와 워커노드를 이어주는 kubelet이 죽어있으니 당연히 생성되지 않을 것
- `sudo systemctl start kubelet`으로 살려놓으니
- 정상동작

#### 확인 
- 참고로 원래 마스터노드엔 noSchedule이다. 
- ![화면 캡처 2021-11-13 203417](https://user-images.githubusercontent.com/62214428/141642282-fd7f5e05-58f2-4960-8de7-ef41e6088de9.png)
- ![화면 캡처 2021-11-13 203429](https://user-images.githubusercontent.com/62214428/141642286-f5e46903-a8eb-45ee-b626-f69826869425.png)
- 해결완료!
----------------


## 백업 및 복구
#### 과제
- 제공된 백업파일로 복구하기
- `kubectl describe pod -n kube-system etcd-nks-master-01.kr-central-1.c.internal`
- `etcd`정보 확인
- 이를 토대로 아래 구성하여 
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
- 이 후 etcd와 같은 static pod들은 `/etc/kubernetes/manifest`설정 정보에 따라 운용되기때문에 들어가서
- `vi etcd.yaml`
- volume path 설정


#### 나의 과정
1. sudo su -
2. tmp에 복구할 파일 wget으로 받고 // 혹은 실제라면 백업파일을 tmp에 저장해놨다는 상황
3. `kubectl describe pod -n kube-system etcd-nks-master-01.kr-central-1.c.internal`
4. 위에서 describe한 내용 맞춰서 아래 작성 ( ex)  --data-dir /var/lib/etcd-kiseok-backup 처럼 따로 공간할당 - 이는 본인 맘대로 지정)
   - 아래는 ip, data-dir, name 등 변경
   - 아래 명령어를 수정한 후 (이건 지금 내 상황에 맞춰 수정한것) 복사하여 명령어 적용!
   - 아래 명령어는 마지막줄을 보면 알 수 있듯이 restore한다 - 이 내용에 따라 etcd를 설정하기 위해 567을 수행하는 것 
```
ETCDCTL_API=3 etcdctl --endpoints=https://172.30.4.244:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --name=nks-master-01.kr-central-1.c.internal \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     --data-dir /var/lib/etcd-kiseok-backup \
     --initial-cluster=nks-master-01.kr-central-1.c.internal=https://172.30.4.244:2380 \
     --initial-cluster-token etcd-cluster-1 \
     --initial-advertise-peer-urls=https://172.30.4.244:2380 \
     snapshot restore /tmp/snapshot-pre-boot.db
```
5. static pod - etcd.yaml volume path 설정 변경을 위해 `/etc/kubernetes/manifest`
6. etcd 부분에서 --data-dir etcd-kiseok-backup변경
  - ![화면 캡처 2021-11-14 001006](https://user-images.githubusercontent.com/62214428/141648936-d44dbc97-c596-414c-8f67-cf320b47eeb6.png)
7. volume 두 부분 /var/lib/etcd-kiseok-backup으로 path 변경
  - ![화면 캡처 2021-11-14 001026](https://user-images.githubusercontent.com/62214428/141648939-1c55277d-dca1-4676-b817-90ce34d4d4ef.png)
- ![화면 캡처 2021-11-14 000916](https://user-images.githubusercontent.com/62214428/141648890-c9c7ea7b-f022-4673-9cce-1dfa40a1849d.png)
- 그랬더니 강사님이 만들어둔 환경으로 동작!!


## 정리
1. 1번 과제는 쿠버네티스의 구조를 파악해볼 수 있었다.
  - 클러스터와 노드를 연결해주는 것이 무엇일까?
  - 바로 kubelet이다
  - kubelet이 작동하지 않으면 pod가 생성되지 않는다

2. 2번 과제는 만약 내가 백업파일을 만들어뒀거나 누군가가 백업파일을 제공했을 때 이를 활용하여 복구할 수 있었다.
  - 먼저 /tmp 등 위치에 백업파일이 존재하고
  - `kubectl describe pod -n kube-system etcd-nks-master-01.kr-central-1.c.internal`으로 etcd의 정보를 확인한 후
  - 이에 맞춰 아래 명령어를 수정한 후 고대로 복사하여 명령어를 입력한다
```
ETCDCTL_API=3 etcdctl --endpoints=https://172.30.4.244:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --name=nks-master-01.kr-central-1.c.internal \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     --data-dir /var/lib/etcd-kiseok-backup \
     --initial-cluster=nks-master-01.kr-central-1.c.internal=https://172.30.4.244:2380 \
     --initial-cluster-token etcd-cluster-1 \
     --initial-advertise-peer-urls=https://172.30.4.244:2380 \
     snapshot restore /tmp/snapshot-pre-boot.db
```
  - 그 후 static pod에 의해 설정되는 etcd를 위해 `/etc/kubernetes/manifest`에서 etcd.yaml에 들어가서
  - etcd 부분에서 --data-dir 변경
  - volume 두 부분 /var/lib/etcd-kiseok-backup으로 path 변경
  - 완료!













