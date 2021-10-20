### APIS and Access

#### 1. label과 annotation
- `label` : 오브젝트를 식별하는데 사용되는 문자열 키/쌍 ( 쿼리 가능 )
- `annotation` : 말 그대로 주석이다. 즉 이걸로 쿼리 불가. 단순히 더 자세한 정보를 제공하기 위한 주석

![화면 캡처 2021-10-20 204829](https://user-images.githubusercontent.com/62214428/138087065-76eb6235-be59-45cc-971e-3c25bc17c4b6.png)
- ` -n` 옵션 : namespace를 말한다
   - namespace : 이름공간 또는 네임스페이스(Namespace)는 개체를 구분할 수 있는 범위를 나타내는 말
- 나중에 저 label을 통해 쿼리가능 ex) k8s-app 중 그 값이 calico-kube-controllers인 애들만 보고싶어! 등..
