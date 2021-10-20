### APIS and Access

#### 1. label과 annotation
- `label` : 오브젝트를 식별하는데 사용되는 문자열 키/쌍 ( 쿼리 가능 )
- `annotation` : 말 그대로 주석이다. 즉 이걸로 쿼리 불가. 단순히 더 자세한 정보를 제공하기 위한 주석

![화면 캡처 2021-10-20 204829](https://user-images.githubusercontent.com/62214428/138087065-76eb6235-be59-45cc-971e-3c25bc17c4b6.png)
- ` -n` 옵션 : namespace를 말한다
   - namespace : 이름공간 또는 네임스페이스(Namespace)는 개체를 구분할 수 있는 범위를 나타내는 말
- 나중에 저 label을 통해 쿼리가능 ex) k8s-app 중 그 값이 calico-kube-controllers인 애들만 보고싶어! 등..

![화면 캡처 2021-10-20 205259](https://user-images.githubusercontent.com/62214428/138087617-ff425bb7-5b4e-41a1-9b97-880f70f5677b.png)
- 하나 더 해보면 여기서도 label을 확인할 수 있고 나중에 pod 중 app = deploy-test인 애들만 가져오도록 쿼리할 수 있을 것
- 물론 이런 label을 직접 표기한 적 없지만 자동으로

#### 2. API접근
- 쿠버네티스와 관련된 접근은 모두 나한테 하세요!! - `API SERVER`
- `API 서버` : 중앙 접근 포인트. Stateless 이며(etcd 활용), 복제 가능
- 주요 기능
   - 1) `API 관리` : 서버에서 API를 노출하고 관리하는 프로세스
   - 2) `요청 처리` : 클라이언트의 개별 API 요청을 처리하는 기능
   - 대부분 `HTTP` 형태로 요청, 콘텐츠는 JSON 기반이 많음
   - 요청의 유형 예 : GET / LIST / POST / DELETE
   - ex)https://kubernetes.io/ko/docs/reference/access-authn-authz/authorization/
   - 3) `내부 제어 루프` : API 작동에 필요한 백그라운드 작업을 담당
       - (대부분은 컨트롤러 매니저에서 수행)
![화면 캡처 2021-10-20 210011](https://user-images.githubusercontent.com/62214428/138088683-641ccf28-ad09-46a7-a4ce-8348852065a9.png)
- 이전 note에서도 봤듯이 모든 클러스터 구성원들은 api server를 바라보고 있으며 api를 통해 접근 및 소통한다. 
- 구성원간 직접적인 소통은 없다.
```
 예를 들어서 
 1.pod를 생성해주세요~ 라는 요청이 kubectl ~~~ 로 전달
 2. api server를 통해 이 요청을 전달 받는다.
 3. api server가 etcd와 소통하며 정보를 얻고
 4. 이 정보를 바탕으로 pod를 생성하기 위해 컨트롤러 매니저, 스케쥴러랑 소통
 5. 생성
```

- ★중요한 것은 모든 요청은 api server를 통해 들어오고 api server가 클러스터 구성원과 소통하며 이를 해결




-----------------

## ACCESS

#### 3. 인증방법 - PKI
- 쿠버네티스에서는 다양한 인증방법이 있으나, 대부분 `PKI(공개 키 인프라)` 기반으로 상호 인증
- 그 전에! 간단하게 대칭키 / 비대칭키
   - 대칭 키 : 암호화와 복호화에 `같은` 암호 키(비밀 키만 있음)를 쓰는 알고리즘
      - (군대 암구호. 서로 동일하게 알고 있는 키) / 혹은 우리가 열쇠를 쓸 때 같은 열쇠르 사용하듯이
   - 비대칭 키 : 공개 키와 비밀 키가 존재하며, 공개 키는 누구나 알 수 있지만 그에 대응하는 `비밀 키는 키의 소유자만`이 가지고 있음
      - 이 공개키는 암호화만 가능. 공개키론 풀 수 없고 개인키로만 푸는 것
      - 예를 들어 내가 친구의 공개키로 나의 인스턴스를 암호화하면? 그 친구만 공개키와 맞는 개인키를 갖고 있을테니 그 친구만 접근 가능
     
   - ★ 이걸 반대로 할 수 있다. `비밀키`로 암호화를 하면 `공개키`로 복호화 할 수 있다.
        - 그럼 비밀키로 A를 암호화하고 인증 기관이 인증된 사람들에게 이 공개키를 전달해주면 인증된 사람들은 공개키로 A에 접근할 수 있다.  / 사실 누구든지 볼 수 있는 정보다 그럼./ 그게 바로
   - PKI : 공개된 키를 개인이나 집단을 대표하는 믿을 수 있는 주체와 엮는 것이며 이는 인증 기관(Certificate authority, 이하 CA)의 등록/인증 발행을 통해 성립
   - 그래서 kubernetes(3)의 note2의 관련 키 확인부분을 보면 crt / key파일이 있다
   - crt : 어떤 기관등이 인증해준 인증서고 key : 나임을 표현, 알려주는 것 정도로 알면된다. 이를 같이 제시해주면 어디에서 인증받은 나야!


#### 3. 인증방법 - RBAC
![화면 캡처 2021-10-20 215659](https://user-images.githubusercontent.com/62214428/138097224-aff1f361-eff8-43dd-87d6-28c72711a210.png)
```
 ex) 
  General Role이라는 역할은 email을 사용할 수 있는 권한이 매핑되어 있는 Role이다
  PayRole은 time cards, Employee records라는 권한이 매핑되어 있는 Role이다
  
  그러면 HR이라는 사람에게 General Role이라는 Role을 할당해주면 이 사람은 email을 사용할 수 있다.
  당연한 것 같은데 이런 방식이 바로 RBAC
  즉 Role과 Control을 매핑시켜놓고 Role을 부여함으로써 어떤 Control( resource..등 )을 가능하게 하는 권한을 부여
  이게 왜 좋냐면 만약에 어떤 큰 회사에서 매일 사람이 발령받아 오고 나가는 경우가 있다고 했을 때
  RBAC없이 매 번 역할이라는 DB에 이 사람 정보를 추가하고 나가면 빼고 하는 과정을 줄일 수 있다.
```
- 이 방식을 쿠버네티스에서도 적용했는데 / 오른쪽 그림을 보자
- Subject는 유저,사람   / Role은 역할이고 / Role에 따라 가능한 권한( resources, verb에 대한)을 미리 매핑
- 그럼 이 subject에게 Role을 부여하는게 RoleBinding


#### 4. Role / Cluster Role
- 어떤 Resource에 어떤 verb 권한을??
- 차이 : kind 에 들어가는 종류명, namespace 존재 여부 (범위만 다름)
- `Cluster Role` 사용 예시 : 클러스터 관리자, 쿠버네티스 컨트롤러
    - 왜냐면.. Cluster Role은 클러스터 전체를 관리하기 위한 Role/ 그래서 클러스터 관리자, 쿠버네티스 컨트롤러한테 부여하고 그냥 Role과 약간의 차이
![화면 캡처 2021-10-20 221305](https://user-images.githubusercontent.com/62214428/138099650-228dcfcb-55e4-4bcf-8355-66a31b2d9baa.png)

#### 5. Role을 만들어보자
![화면 캡처 2021-10-20 221831](https://user-images.githubusercontent.com/62214428/138100596-c46dcf99-8602-4f9f-869d-b926301acb1d.png)
- 명령어 연습도 필요하다
- `1번`에서 뭘 쳐야하는지 몰라서 tap tap
- `2번`에서 보니까 role이름이 필요하네?
- `3번`에서 보니까 role에 부여할 verb가 필요하네?
- `4번` 예시를 봐보자


