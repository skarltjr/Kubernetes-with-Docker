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
