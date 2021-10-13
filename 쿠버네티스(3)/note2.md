### 쿠버네티스 설치
### 1. 대장 마스터노드에 로드밸런서 haproxy설치
![화면 캡처 2021-10-13 230732](https://user-images.githubusercontent.com/62214428/137149393-606a62e5-2058-4e4f-8b49-da2da8a7b8fe.png)
- ![화면 캡처 2021-10-13 225052](https://user-images.githubusercontent.com/62214428/137149449-3e2d40bf-3f2a-47ce-a159-f51b21827cae.png)
- 재시동 후 확인해보기
- ![화면 캡처 2021-10-13 225300](https://user-images.githubusercontent.com/62214428/137149500-0db7636d-7c58-4100-aa39-d614f5822783.png)

### 2. 쿠버네티스 설치
• Kubeadm 설치
- https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
• 설치 전,
- 리눅스 기반. 2GB Ram, 2 CPU 이상의 성능
- SWAP 미사용 (카카오 기본 이미지에 SWAP 미설정됨)
- 고유한 hostname, MAC, UUID, 포트 개방, 네트워크 연결
- # hostname 으로 hostname 확인 후 /etc/hosts 에 반영 (worker도)
   - hostname이라는 명령어로 hostname확인 후 
   - `sudo vi /etc/hosts`에서 
   - ![화면 캡처 2021-10-13 230208](https://user-images.githubusercontent.com/62214428/137149888-e31a4bde-375c-4428-8409-10beaf5d1ae9.png)
   - 위 내용 복사해서 다른 노드에도 동일하게 수행 ( 다른노드는 hostname알아볼 필요없다)
   
   
### 3. iptables가 브리지된 트래픽을 보게 하기
https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#iptables%EA%B0%80-%EB%B8%8C%EB%A6%AC%EC%A7%80%EB%90%9C-%ED%8A%B8%EB%9E%98%ED%94%BD%EC%9D%84-%EB%B3%B4%EA%B2%8C-%ED%95%98%EA%B8%B0
- br_netfilter 모듈이 로드되었는지 확인한다. lsmod | grep br_netfilter 를 실행하면 된다
- 아마 아무것도 안 뜰거다. 그럼 이제 설치를 해주자
- 아래를 복사 붙여넣자 
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
- br_netfilter를 불러오고
- 1 => 활성화 즉 활성화시킨다
- 이 과정을 `모든 노드`에 대해 수행

### 4. 런타임 설치
- `파드에서 컨테이너를 실행하기 위해, 쿠버네티스는 컨테이너 런타임을 사용한다.`
- 도커를 사용할거니까 도커를 설치한다 // 모든 노드에 대해
```
• [전체 노드] CRI (Container runtime interface) 설치 필요 : Docker 설치/시작
• # sudo yum	install	-y	yum-utils	device-mapper-persistent-data	lvm2	
• # sudo yum-config-manager	--add-repo	https://download.docker.com/linux/centos/docker-ce.repo
• # sudo yum	install	docker-ce -y
• # sudo vi	/usr/lib/systemd/system/docker.service 들어가서
ExecStart=/usr/bin/dockerd -H	fd://	--containerd=/run/containerd/containerd.sock --exec-opt	native.cgroupdr
iver=systemd

• # sudo systemctl daemon-reload
• # sudo systemctl start docker	&&	sudo systemctl enable docker
• # sudo docker info	| grep -i cgroup
```
- 참고로 `sudo vi	/usr/lib/systemd/system/docker.service` 부분은 `컨테이너 런타임`과 `kubelet`이랑 `cgroup driver`를 맞춰야하기위해 설정
- `sudo vi	/usr/lib/systemd/system/docker.service`를 통해 docker.service에 들어가서 `ExecStart`으로 시작하는 부분에서 설정
- 여기서 오타나면 망한다!!!
























