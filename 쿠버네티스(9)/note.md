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
- `kubectl describe pod -n kube-system etcd-nks-master-01.kr-central-1.c.internal`
- `etcd`정보 확인
- 이를 토대로 아래 구성하여 백업
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



















