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


**3) 구성도**
![화면 캡처 2021-09-29 211819](https://user-images.githubusercontent.com/62214428/135266928-586bb8ce-c969-4fd9-a278-8c63aa058247.png)
- 네트워크를 떠올려보자
- `공인 아이피 : 내부 사설 아이피` - 한정된 ip의 한계를 극복하기 위해 내부의 사설 ip를 사용하고 이를 매핑하기 위해선 NAT을 활용
- 당연히 위에서 생성한 노드들이 갖고 있는 ip가 바로 이러한 사설 아이피
- 그런데 카카오 클라우드에선 아직 공인 ip할당이 안된다고 했다/ 그럼 외부에서 아예 접속을 할 수 없는가?
- ![화면 캡처 2021-09-29 212628](https://user-images.githubusercontent.com/62214428/135268077-dd138dbc-6c82-43dc-b239-5001636ab54e.png)
- `bastion host` - 프록시 서버를 활용 + openVPN으로 여기에 접근
- ![화면 캡처 2021-09-29 212741](https://user-images.githubusercontent.com/62214428/135268252-4958f5eb-78bc-4607-aa16-d527d584b166.png)
- 즉 ` -> ( openVPN 공인 아이피 : bastion host 공인 아이피 ) -> 내부 사설아이피를 가진 노드로 접근`
- 어떻게? 
   - 1. openVPN 공인 아이피 : bastion host 공인 아이피  openVPN 공인 아이피 내부에 카카오 사설 대역을 받은 pc가 있다 / 즉 사설 대역을 가진 pc를 openVPN공인 아이피로 감싸줬다고 생각
   - 2. 공인 ip를 통해 bastion host를 만나서 이 ip는 이제 사용했으니 벗겨내고, 벗겨내면 내부에 카카오 사설 대역을 받은 pc가 존재
   - 3. 이 사설 대역을 받은 pc로 이제 내부 노드들에 접근  
   - 그러기 위해선 ` 그림 오른쪽 아래처럼 vpn을 설치하고 .ovpn 파일을 활용한다.`
   - ![화면 캡처 2021-09-29 213955](https://user-images.githubusercontent.com/62214428/135270047-bdf9affb-ab5b-429f-9734-11b0145a657b.png)
   - 모르면? 구글링


