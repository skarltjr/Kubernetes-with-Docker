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

### 4. 런타임 설치 / 도커
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

### 5. kubeadm, kubelet 및 kubectl 설치
- [전체 노드] Redhat 배포판 버젼으로
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

먼저 여기까지 복사붙여넣으면 레포지토리들을 저장
`cd /etc/yum.repos.d/` 으로 확인해보고

===>  permissive 모드로 SELinux 설정(효과적으로 비활성화)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

• [전체 노드] 단, kubelet / kubeadmin / kubectl 은 최신보다 한단계 낮은 버젼으로 설치 (나중에 업데이트해보기 위해)
```
#	sudo yum	info	kubelet --disableexcludes=Kubernetes -y // 그냥 정보확인
#	sudo yum	install	kubelet-1.21.0	--disableexcludes=kubernetes -y // 여기서부터 모든 노드에서 설치
#	sudo yum	install	kubectl-1.21.0	--disableexcludes=kubernetes -y
#	sudo yum	install	kubeadm-1.21.0	--disableexcludes=kubernetes -y
# sudo systemctl enable	--now	kubelet // enable을 해주면 껐다 켜지면 자동으로 재시작알아서
```

- 다음으로
```
앞에서 도커의 cgroup driver를 systemd로 설정했기에, 쿠버도 마찬가지로 설정
#	sudo vi	/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf	

그림참조
[Service]	
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd --runtime-cgroups=/systemd/system.slice --kubeletcgroups=/systemd/system.slice"


이 후 
sudo systemctl daemon-reload
sudo systemctl restart kubelet
이제 필요한 패키지는 모두 설치 완료. 클러스터를 만들어야 함
```
![화면 캡처 2021-10-14 003043](https://user-images.githubusercontent.com/62214428/137165497-ada30c91-2c52-4897-85ec-776306964c45.png)

### 6. etcd를 따로 빼지않고 묶어서 사용할것이기 때문에
- 로드밸런싱을 담당하는 첫번째 마스터 노드에만 stack control node를 설치한다
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
![화면 캡처 2021-10-14 014434](https://user-images.githubusercontent.com/62214428/137177282-63cc9b7d-a71b-4a4d-a47b-e76f00628f9a.png)
`Your Kubernetes control-plane has initialized successfully!` 확인

##### 참고로 to start using your clustering~ 부분부터 마지막까지 복사해서 ex) finish.txt만들어서 저장해둬라 날아갈수도있어서
```
To start using your cluster, you need to run the following as a regular user:  // 다른 사람이 실행할 수 있으려면 다른 유저한테 아래 내용을 수행하도록 시켜야합니다~

  mkdir -p $HOME/.kube      // 참고로 .은 숨김파일
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 172.30.4.114:16443 --token fhgojd.2ep3euw248imiqh1 \
        --discovery-token-ca-cert-hash sha256:213c533cc9f2d1323b0f2bd20162ca48085b78f610ebee99f86f200a3e609d69 \
        --control-plane --certificate-key f47c18f18407fd9e0e64c817bc5a81f87855a594a9098fd3b57d9f06874becb6

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.30.4.114:16443 --token fhgojd.2ep3euw248imiqh1 \
        --discovery-token-ca-cert-hash sha256:213c533cc9f2d1323b0f2bd20162ca48085b78f610ebee99f86f200a3e609d69
```

- 그 중 아래 부분은 복사 붙여넣기로 실행 ( 지금 다 마스터 노드 1에서)
``` 
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- 그 다음으로 아래 문구가 있지만
```
Alternatively,	if	you	are	the	root	user,	you	can	run:
export	KUBECONFIG=/etc/kubernetes/admin.conf
```
- 대신하여
- ![화면 캡처 2021-10-14 020653](https://user-images.githubusercontent.com/62214428/137180489-adca7194-f723-4ad9-88cf-72fcb2ce432e.png)
- 이것은 이전에 kube init할 때 전달한 인증서를 루트에도 넣어둬서 인증이 저절로되도록한것


- 다음으로 You should now deploy a pod network to the cluster.
```
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
  그 중 calico = https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises
```
- 그래서 // 여전히 마스터노드 1에서만
![화면 캡처 2021-10-14 021453](https://user-images.githubusercontent.com/62214428/137181438-d4cf4e2f-5198-4f34-9a27-83aad3b88b73.png)

- `kubectl`이 동작하나 확인해보면 / 잘 동작하니까 kubectl apply -f calico.yml
![화면 캡처 2021-10-14 021627](https://user-images.githubusercontent.com/62214428/137181667-bb782738-b41f-4547-bbf2-40a4720d7ba6.png)
```
→ #	curl	https://docs.projectcalico.org/manifests/calico.yaml -O
→	#	kubectl apply	-f	calico.yaml
```

- 참고로 내 `finish.txt`는 cd /로 이동해야 찾을 수 있었다. ~에 넣어둔것이 아니었다


- 다음으로 : `다른 마스터 노드`에서 아래를 수행 ( sudo su - 로 루트에서 진행)  
- You can now join any number of the control-plane node running the following command on each as root:
```
   kubeadm join 172.30.4.114:16443 --token fhgojd.2ep3euw248imiqh1 \
 --discovery-token-ca-cert-hash sha256:213c533cc9f2d1323b0f2bd20162ca48085b78f610ebee99f86f200a3e609d69 \
 --control-plane --certificate-key f47c18f18407fd9e0e64c817bc5a81f87855a594a9098fd3b57d9f06874becb6
        
        
           
  finish.txt에서 위를 복사하고     
```
- 마스터노드2 들어가서 `sudo su -` -> 루트에서 붙여넣는다
- 참고로 나는!! 마스터노드1을 실수로 최신버전으로 받아서 kubelet kubeadm ..이 다른거와 버전이 안맞고 그래서 에러났다
- 처음부터 다시 다 했다.......

- 이어서 실패했던걸  다시 다 만들고 이번엔 성공해서 빨간표시 수행 
- ![화면 캡처 2021-10-14 220311](https://user-images.githubusercontent.com/62214428/137322684-8591c5e2-4bce-4c4e-86c1-498c41380eb6.png)
- 마스터 2,3번 위 과정 수행했으면 마스터 1에서 확인해보기
- ![화면 캡처 2021-10-14 220749](https://user-images.githubusercontent.com/62214428/137323407-b4189033-f924-46be-b566-85bdc5b5dc29.png)
- 이제 `워커노드` 해줄 차례=>  마스터1의 finish.txt에서 맨 아래 문구 마찬가지로 복사 후 => 워커 1,2에서 `루트`로 실행
```
kubeadm join 172.30.4.244:16443 --token uhl4o2.i89wbakhi813p6c2 \
--discovery-token-ca-cert-hash sha256:1101f3634043690b5e5b2c99b1ed08e2717d00b8aba5fa9dcd7db75878f12015
```

- 확인해보면 마스터1에서 `kubectl get nodes`
- ![화면 캡처 2021-10-14 221505](https://user-images.githubusercontent.com/62214428/137324623-59c6f9ca-cac2-44e6-8493-daaef3490536.png)

---------------------

### kubectl
• Kubectl 은 쿠버네티스 클러스터를 제어하기 위한 커맨드 라인 도구
ü kubectl [command] [TYPE] [NAME] [flags]
- command: 명령을 하려는 ‘동사’. 예: create, get, describe, delete
- TYPE: 리소스 타입
- NAME: 리소스 이름
- flags: 선택적 옵션

- 보라색이 자주 쓰일 수 있는 것들
- ![화면 캡처 2021-10-14 222001](https://user-images.githubusercontent.com/62214428/137325487-9305c6de-e28d-4b62-b80c-438510be6ec0.png)
- `kubectl run`은 파드를 생성하는 것 그런데 `kubectl run 파드이름 --image=이미지 --dry-run=client -o yaml > kiseok.yaml`로 하면 yaml로 만들어준다
- `왜 yaml로 하냐 굳이?` => 쿠버네티스가 다 명령어를 먹지 않을 때도 있다고한다. 그래서  yaml로도 많이 쓴다.
- 그래서 저 yaml은 아직 파드를 만든게 아니고 파드를 만들 준비를 작성해둔것
- `kubectl apply -f kiseok.yaml`로 yaml을 통해 파드 생성

### 관련 키 확인
![화면 캡처 2021-10-14 222656](https://user-images.githubusercontent.com/62214428/137326760-eb0d1662-fcd9-4673-ad50-ada8e62170b2.png)
![화면 캡처 2021-10-14 222627](https://user-images.githubusercontent.com/62214428/137326766-3ca34e25-c61a-4db0-ba32-572aefedd2a9.png)


### 설치완료 후 팁
- tab눌러서 자동완성 적용하면 편리하니까
![화면 캡처 2021-10-14 222832](https://user-images.githubusercontent.com/62214428/137327013-2c5b77da-e0e6-4608-8fa4-40bfe124e28e.png)
![화면 캡처 2021-10-14 223236](https://user-images.githubusercontent.com/62214428/137327737-f0c3127c-cd4e-4f10-bdae-01664b1a1ccd.png)

