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












