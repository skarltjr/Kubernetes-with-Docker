### 1. 쿠버네티스의 설계 사상
- `선언적 구성`  : "내 웹서비스의 레플리카를 항상 5개씩 실행하고 싶습니다 < - > 명령어 구성 : 레플리카를 5개 만들어
   - 제어 루프 : 현재 상태를 관찰하여 사용자가 원하는 상태로 유지
   - 즉, 명령어로 구성하면 `실패할 땐` 또 명령어로 실행해야한다
   - 그러나 여기서 말하고자 하는 것은 매 번 명령어로 수행할 필요없이 현재 상태를 계속 관찰하며 요구 내용에 맞추도록 조정
   - ex) 난 a노드에 파드 5개 유지하고싶어. -> 만약 5개 중 2개가 죽으면 알아서 2개를 만들어 채운다

- `컨트롤러 구성` : 각 기능별로 독립적으로 분산되게 설계 => 유연하고 안정적이지만 복잡
- `동적 그룹화` : 우리 팀원은 오렌지 색 옷을 입은 사람들입니다 or 애플리케이션이 `백엔드`인 것들만 < - > 정적 그룹화 : 우리 팀원은 남기석 남기석2
   - 구체화보다 추상적으로. 역할과 담당자를 구분 / 언제든지 대체 가능
   - 레이블(쿼리 가능) 및 어노테이션(메타데이터용)으로 구성

- `API 기반 상호작용` : 쿠버네티스 요소들이 서로 직접 접근불가. 구성 요소를 대체하기 용이
  - 구성 요소원에게 의존x  / API에 기반하여 이 API를 담당하는 구성 요소는 언제든지 바뀌어도 대처가능한 유연함 
![화면 캡처 2021-10-13 204005](https://user-images.githubusercontent.com/62214428/137125859-17f2b672-d51c-48fa-9118-37ab8f1a7467.png)


### 2. 쿠버네티스의 구조 : 클러스터
- `클러스터`란? : `노드`라고 하는 `워커 머신`의 집합
- `노드`란? : 컨테이너화 된 애플리케이션을 실행하는 서버  / 하나의 노드에 여러 컨테이너화 된 애플리케이션 ( `파드` ) 존재가능
- `워커 노드`란? : 애플리케이션의 구성요소인 파드( > 컨테이너 )를 호스트
- `컨트롤 플레인`이란? : == `마스터 노드`로 워커 노드와 클러스터의 파드들을 관리

![화면 캡처 2021-10-13 204415](https://user-images.githubusercontent.com/62214428/137126654-e427652f-cc29-4d5a-99b7-9ee5e144d745.png)


### 3. 쿠버네티스의 구조 : 클러스터 구성요소 ( 마스터노드 )
![화면 캡처 2021-10-13 205001](https://user-images.githubusercontent.com/62214428/137127162-187b997a-dd8f-48fc-930f-bca1d6f27440.png)
- `API서버` : 즉 외부에서 접근할 수 있도록 api노출 / 외부에선 이 api들을 통해서 접근
- `etcd` : 쿠버네티스에 관련된 모든 내용이 etcd에 저장 / 쿠버네티스에 지금 파드가 몇개인지~ , a파드가 어떤 노드에 존재하는지~ ..
- `스케쥴러` : 스케쥴러는 파드를 신규 생성 할때 할당. 컨트롤러 매니저는 기 생성된 파드를 유지하는 역할. 이때도 기 생성된 파드가 없어질경우 유지를 위해 스케쥴러를 호출
     - ex) 난 a노드에 파드 5개 유지하고싶어. -> 만약 5개 중 2개가 죽으면 알아서 2개를 만들어 채운다
- `컨트롤러 매니저` : 컨트롤러 매니저는 기 생성된 파드를 유지하는 역할


### 4. 쿠버네티스의 구조 : 클러스터 구성요소 ( 워커 노드 )
![화면 캡처 2021-10-13 210241](https://user-images.githubusercontent.com/62214428/137128783-91b3253e-ee29-4004-9523-50de37afac91.png)
- `Kubelet`: 컨트롤 플레인의 구성요소인 api-server로부터 전달된 요청이 자신에게 온 요청인지 판단하여 받아들이고 요청에 맞게 pod 실행,중지,삭제를 수행한다.
  - 즉 마스터 노드와 워커 노드의 연결이자 파드 관리요소
- `kube-proxy` : pod는 유동적이기에 직접 연결을 하는경우 연결 안정성이 굉장히 떨어진다. 그래서 service를 통해 유동적인 pod에 고정적인 연결을 제공한다. 그런데 단순히 service만 있다고 알아서 pod에 연결되는것은 아니다. 어떤 pod에 요청을 전달할지 로드밸런싱 및 pod 연결을 담당하는게 바로 쿠베 프록시. 즉 어떤 pod로 전달하지 라우팅 rule을 정하며 그 라우팅체계를 관리하는것이 쿠베프록시다.
- `컨테이너 런타임` : ex) 컨테이너를 실행하는 도커


### 5. 타 솔루션과의 비교
![화면 캡처 2021-10-13 210917](https://user-images.githubusercontent.com/62214428/137129626-511096f4-8e49-401f-8bc3-df4dbaae5dbb.png)
- 중요한것은 `쿠버네티스가 모든 걸 새로이 만든 것이 아니다. 이미 존재하던 기능들이다`
- `다만, !! 컨테이너에 맞게 이 기능들을 활용할 수 있도록 한 것이 바로 쿠버네티스`


-------------
## 쿠버네티스 설치

### 그 전에 구성 방식에 대해 살펴보기
- 중첩된 etcd vs 외부 etcd : etcd 배포 위치의 차이
- etcd에 모든 내용이 기록된다고 했다. 그래서 외부에 또 서버를 설치해서 etcd만 따로 둔다고하면 안전할 수 있지만 실질적으로 etcd하나 당 서버 하나( = 소나타 한 대)는 어렵다
![화면 캡처 2021-10-13 212827](https://user-images.githubusercontent.com/62214428/137132233-cdf5a1f6-6531-4e68-ba1b-428a3c9a49fa.png)


### 구성도
![화면 캡처 2021-10-13 213728](https://user-images.githubusercontent.com/62214428/137133699-d0d8f95f-6ccc-44fa-8d73-2c93580a7a00.png)
- 물리적인 L4스위치를 사용할 수 있다면 좋겠지만 실습에선 그럴 수 없다
- 그래서 ha proxy를 사용

### 결과적으로 아래처럼 구성한다
![화면 캡처 2021-10-13 214154](https://user-images.githubusercontent.com/62214428/137134724-36b80d04-7859-416b-a925-c6b80865038a.png)
- 중요한 것은 `haproxy`는 하나의 마스터노드에만 설치하고 이 노드는 `로드밸런싱`을 담당한다
- 따라서 `워커노드`들은 이 `마스터노드`에만 요청을 보내고 이 `마스터노드`가 요청을 뿌린다.
- 그럼 이 대장 마스터노드가 여러 마스터 노드 중 만약 누가 죽었으면 다른 애한테 요청을 뿌린다.
- 즉 마스터1로 전달받은 데이터를 마스터node1 ~ node3의 6443 포트로 포워드 시켜줍니다.
