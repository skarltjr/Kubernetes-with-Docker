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
   - EX) D 관점으로 봤을 때 만약 `DISK 1`이 죽으면 `DP D2 D3를 활용해서` DISK 0의 DP에 DISK 1의 D1을 복구
   - EX) C 관점으로 봤을 때 만약 `DISK 0`이 죽으면 `CP C2 C3를 활용해서` DISK 1의 CP에 DISK 0의 C1을 복구... 이런식
   - 근데 결국 DISK가 2개 이상 죽으면 복구가 안된다.

### 3. VOLUME
![화면 캡처 2021-11-03 210824](https://user-images.githubusercontent.com/62214428/140057312-92dcf95a-1420-4425-9182-eecc2436064f.png)

- master 01번 노드를 확인해보면
- ![화면 캡처 2021-11-03 211432](https://user-images.githubusercontent.com/62214428/140058158-954b0cb1-e239-4c98-9717-700ebd41ef5b.png)
   - 1번 = 맨 처음에 vm을 만들 때 설정했듯이 마스터 노드는 10Gb 용량이고 
   - 2번 = 이 10GB짜리 디스크가 바로 `/dev/vda1`이고
   - 3번 = 이 디스크의 UUID. 즉 이 디스크의 다른 이름이 이 UUID
   - 4번 = 파일 시스템은 리눅스에서 많이 쓰는 `xfs` 뒤에는 디폴트 옵션

- 정리하자면 `볼륨`도 결국 `스토리지`인데 `하나의 파일 시스템을 갖춘` 
- 참고로 여러개의 작은 용량의 디스크를 묶어서 하나의 볼륨으로 만들어 사용할 수 있다.

### 4. PV / PVC
![화면 캡처 2021-11-03 212305](https://user-images.githubusercontent.com/62214428/140059253-334b7a7f-f813-496f-b236-de245975b7e0.png)
- `PV` = 스토리지
- `PVC` = 스토리지를 원한다는 요청. 클레임

### 5. PV를 생성해보자
- PV 생성: https://kubernetes.io/ko/docs/concepts/storage/volumes/#hostpath
- 그 중 hostpath방법
```
hostpath pv생성을 위한 yaml

apiVersion: v1
kind: Pod      // pv 또한 하나의 pod로 생성하고
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx      // 해당 이미지를 사용하고
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:           // 이 볼륨을 생성하여 위에서 바로 위에서 마운트를 설정
  - name: test-volume
    hostPath:
      # 호스트의 디렉터리 위치
      path: /data
      # 이 필드는 선택 사항이다
      type: Directory
```
- 1. kubectl apply -f hostpath.yaml -> 계속 pod가 생성이 대기중
  - ![화면 캡처 2021-11-03 213547](https://user-images.githubusercontent.com/62214428/140061033-68f1235f-307a-4c88-beec-cf55cb522d00.png)
- 2. kubectl get pod -o wide
  - ![화면 캡처 2021-11-03 213815](https://user-images.githubusercontent.com/62214428/140061399-d12ea746-a38c-4f7e-b729-9a18baaca025.png)
  - 2번 워커노드에 띄우려고 하는데 문제가 있는 것 같다.
- 3. 왜 그런지 알아야하니까 자세히 살펴보자 `kubectl describe pod test-pd`
  - ![화면 캡처 2021-11-03 213707](https://user-images.githubusercontent.com/62214428/140061258-ee59d62e-f45e-4aea-955e-2fabb1106c2e.png)
  - yaml을 다시 확인해보면 `volume`의 path가 /data인 것을 확인할 수 있다.

- 4. `워커노드 2번`에 /data가 있는지 확인을 해보자
  - 없다면 없으니까 실패하고 대기중인것 -> 만들어줘야겠지
  - ![화면 캡처 2021-11-03 214033](https://user-images.githubusercontent.com/62214428/140061688-3dd71fa0-e0f3-48bf-bdc9-34ca513154c2.png)
  - 없다!! 그러니까 볼륨을 못잡고 대기중인것

- 5. /data를 만들어주자
  - 워커노드 2번 루트 하위에 mkdir data로 만들어주면
  - ![화면 캡처 2021-11-03 214145](https://user-images.githubusercontent.com/62214428/140061859-577f2e32-f4c5-43d5-bac4-9faf7f0a4296.png)

### 6. yaml을 자세히 살펴보자
```
apiVersion: v1
kind: Pod      // pv 또한 하나의 pod로 생성하고
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx      // 해당 이미지를 사용하고
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:           // 이 볼륨을 생성하여 위에서 바로 위에서 마운트를 설정
  - name: test-volume
    hostPath:
      # 호스트의 디렉터리 위치
      path: /data
      # 이 필드는 선택 사항이다
      type: Directory
```
- 여기서 자세히봐야할게 volumeMounts / volumes
- 어떤 의미냐면 
- `mountPath` => 이 yaml을 통해 생성된 컨테이너 test-pd안에 `/test-pd`경로의 디렉토리를
- hostPath => `worker 02`의 `path( /data)`에 마운트 했다는 것
- 즉★ `워커노드 02의 /data`와 `test-pd라는 pod 내부 /test-pd`가 연동
- 이를 확인해보려면 
- 1★ 워커노드 02의 `/data`에 파일을 만들어보고 
- 2★ test-pd pod의 /test-pd에 동일한 파일이 있는지 확인해보면 된다

### 7. 확인해보자
1. 워커노드 2번의 `/data`에서 파일생성
- ![화면 캡처 2021-11-03 215442](https://user-images.githubusercontent.com/62214428/140063650-84b37624-485b-419d-bc4b-ad8edf47874d.png)
2. `test-pd` pod에 접속하여 `/test-pd`에 `likelion`이 있는지 확인해보면
- ![화면 캡처 2021-11-03 215604](https://user-images.githubusercontent.com/62214428/140063827-933f5e25-09d9-4d70-9ae4-2a00268d3082.png)
3. 짜잔~. 
