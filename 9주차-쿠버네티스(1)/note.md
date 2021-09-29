### 1. pc 및 vm 세팅 - 쿠버네티스 설치
![화면 캡처 2021-09-29 210612](https://user-images.githubusercontent.com/62214428/135265092-b195ed36-7222-4a10-980d-c6bda4e4507e.png)
**1) 마스터 노드**
  - 3개의 `마스터 노드`(=인스턴스)를 생성 ( 여기서는 cpu 2 / ram 4GB /centOS / volume = 10GB)
     - 참고로 마스터 노드는 홀수개로 구성해야 안정적이다.라는 말이 있다. 
  - 참고로 GCP나 AWS는 공인아이피 / 사설아이피를 할당받아 외부에서 접속 & 내부에서 외부로 가능
  - 그러나 카카오 i 클라우드는 아직 공인 ip할당 기능은 x / 외부러 보내기만 가능 `그래서 나중에 openVPN사용`

**2) 워커 노드**
- 동일하게 구성 but 인스턴스 = 2개 / volume = 20
- ![화면 캡처 2021-09-29 211337](https://user-images.githubusercontent.com/62214428/135266298-ac0f8fce-2694-4fb4-9915-cc1607c9b5e6.png)
