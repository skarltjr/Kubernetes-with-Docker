### 멀티 컨테이너 Pod
![화면 캡처 2021-10-27 211521](https://user-images.githubusercontent.com/62214428/139063580-1455f7bc-3fb1-46cb-87c2-32d08f929dc5.png)
```
 멀티 컨테이너 pod는 명령어로 생성하는 방법 x
 deployment를 yaml로 만들고 그림처럼 yaml을 수정해서 생성
 그런데 이 때 container가 2개라고 container가 2번 들어가는게 아니다.
```

### 멀티 컨테이너 pod를 만들어보자
1. `kubectl run test --image=nginx --dry-run=client -o yaml > multicontainer.yaml`로 pod를 만들 yaml 생성
2. 멀티 컨테이너 pod를 만들기 위해 yaml을 수정하자
  - 이 때 `container`가 2번 들어가는게 아니다! 주의
  - ![화면 캡처 2021-10-27 212442](https://user-images.githubusercontent.com/62214428/139064858-24cb54c5-1cf5-4b16-ab70-e5bd5bad8518.png)
  - 그런데 에러가 발생
  - `kubectl logs test` -> test파드 로그 찍어보니
  - `error: a container name must be specified for pod test, choose one of: [test test2]`
3. 다시 yaml을 작성
  - ![화면 캡처 2021-10-27 212945](https://user-images.githubusercontent.com/62214428/139065674-89485acf-6b7c-467f-9dba-1040bce8aa7d.png)
  - ![화면 캡처 2021-10-27 213050](https://user-images.githubusercontent.com/62214428/139065843-7a2de581-e054-493a-9bbc-2bdaae4cab3a.png)
4. 2개의 컨테이너를 가진 multicontainer pod를 생성완료


### 멀티 컨테이너에서 두 컨테이너가 하나의 emptydir 볼륨을 바라보게 하기
- 시험에 나오는 문제로 쉽게 말해 도커에서 볼륨을 생성해주는 것 처럼 마찬가지로 컨테이너의 정보를 따로 저장해둘 볼륨을 잡는데 이 볼륨을 두 컨테이너가 공유
- https://kubernetes.io/docs/concepts/storage/volumes/#emptydir-configuration-example
```
example
...

apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```









