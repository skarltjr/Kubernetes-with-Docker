## [Storage] Volumes and data

### 1. Storage 개요
- 쿠버네티스의 스토리지를 말하기 전 it에서의 스토리지에 대해 살펴보자
![화면 캡처 2021-11-03 203312](https://user-images.githubusercontent.com/62214428/140053043-4de0223c-0c0c-4b3b-befe-9813e904fdbb.png)

- `DAS`는 노트북이나 컴퓨터에 하드디스크가 붙어있는 것처럼 직접 붙어있는 형태
   - 지금 그림에서 보면 하나의 서버에 하나의 스토리지 직접 연결 / 일대일
   - 그런데 이게 값이 싸니까 Ceph,GlusterFS같은 오픈소스로 발전
   - 하나에 서버에 여러 디스크를 꽂아서 일대다 / 마치 `NAS`처럼

- `NAS(Network Attached Storage)`는 IP를 기반으로 네트워크 결합 스토리지 - `DAS`의 반대 개념
   - 여러 서버/PC가 스토리지를 `공유`해서 사용가능 / 여러 서버가 같은 스토리지에 접근가능
   
- `SAN(Storage Area Networking)`은 네트워크로 통신하여 저장장치에 연결!
   - 일반적인 스위치와 서버는 아이피 기반으로 길을 찾아다닌다면 
   - `SAN`에서는 추가적으로 뒤에 `WWN`기반으로 연결 - SAN SWITCH
   - 추가로 스위치 - 서버는 LAN으로 연결이라면 / 서버 - SAN SWITCH는 광케이블로

### 2. Storage 개요 - 내부 / RAID
![화면 캡처 2021-11-03 204714](https://user-images.githubusercontent.com/62214428/140054647-c5e28abb-eb3e-4cb8-a3cc-c442ef9925b0.png)
- `RAID`는 여러개의 디스크를 묶어 하나의 디스크 처럼 사용하는 기술
- `RAID 0`의 경우 => a1,a3,a5.. / a2,a4,a6... 으로 구성되어 있는걸 볼 수 이쓴ㄴ데
   - 그렇기 때문에 `Disk 0`이 날아가거나 고장나면 답이없음
   - 하드디스크도 생각해보면 파일 날아가면 날아가버린것.

- `RAID 1`의 경우 => `DISK 0`의 내용을 `DISK 1`에 복제를 해놓는
   - 생각해볼것은 이런경우 복구측면에서 괜찮지만 내가 디스크를 1테라짜리 `2개`사면 사실상 `1개`만 사용하는 것
   - 왜냐면 스토리지 자체에서 하나는 저장 / 하나는 복구를 위해 저장
- `RAID 5`의 경우 => 패리티 비트를 이용하여 0과 1의 중간정도
   - DISK 0 = A1, B1, C1, `DP(parity)`
   - DISK 1 = A2, B2, `CP`, D1
   - DISK 2 = A3, `BP`, C2, D2
   - DISK 3 = `AP`, B3, C3, D3
   - 어떤식으로 활용되는가?
   - EX) D 관점으로 봤을 때 만약 `DISK 1`이 죽으면 DISK 0의 DP에 DISK 1의 D1을 복구
   - EX) C 관점으로 봤을 때 만약 `DISK 0`이 죽으면 DISK 1의 CP에 DISK 0의 C1을 복구... 이런식






