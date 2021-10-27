### 1. 멀티 컨테이너 Pod
![화면 캡처 2021-10-27 211521](https://user-images.githubusercontent.com/62214428/139063580-1455f7bc-3fb1-46cb-87c2-32d08f929dc5.png)
```
 멀티 컨테이너 pod는 명령어로 생성하는 방법 x
 deployment를 yaml로 만들고 그림처럼 yaml을 수정해서 생성
 그런데 이 때 container가 2개라고 container가 2번 들어가는게 아니다.
```

### 2. 멀티 컨테이너 pod를 만들어보자
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


### 3. 멀티 컨테이너에서 두 컨테이너가 하나의 emptydir 볼륨을 바라보게 하기
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

### 4. example을 참고해서 같은 볼륨을 바라보는 멀티컨테이너 pod를 생성하기 위한 emptydir.yaml을 작성해보자
![화면 캡처 2021-10-27 215229](https://user-images.githubusercontent.com/62214428/139069279-e9b3b700-3cce-4bed-b5eb-0ad66c899a23.png)
- emptydir은 그냥 이름일 뿐
- 두 컨테이너 모두 `cache-volume`이란 이름의 볼륨에 마운트를 하는데
- 이 `cache-volume`은 name = cache-volume인 emptyDir이다~
- 일단 앞에서 만든 test라는 pod지우고
- ![화면 캡처 2021-10-27 215559](https://user-images.githubusercontent.com/62214428/139069883-89316cdb-cf4b-4d00-8966-dedc953b9fde.png)


### 5. 생성된 pod를 describe로 자세히 살펴보자
- `kubectl describe pod 이름`
- ![화면 캡처 2021-10-27 220025](https://user-images.githubusercontent.com/62214428/139070615-5b425c3b-58c9-432e-8848-e06f843990d1.png)
- volume을 보면 여기서 이 emptydir는 pod의 상황에 다라 같이 지워지는 영구적인게 아니다

### 6. 컨테이너에 접속해보자
- 원래 단일 컨테이너 pod일 땐 `kubectl exec it pod이름 -- /bin/sh로 가능
- 컨테이너가 하나밖에 없으니 굳이 컨테이너 이름을 명시하지 않아도 됐던것
- 멀티 컨테이너에서는 `kubectl exec it pod이름 -c 컨테이너1 이름 -- /bin/sh` -c는 컨테이너 옵션 - 어떤 컨테이너에 접속하겠다
- pod를 만든 test-pd는 `emptydir.yaml`에서 test-container랑 redis라는 컨테이너를 만들었으니 우선
- `kubectl exec -it test-pd -c test-container -- /bin/sh

### 7. 정말로  volume을 공유하는지 확인해보자
- exec으로 하나의 컨테이너에 접속한 상태 / 여기선 test-container에
- ![화면 캡처 2021-10-27 222119](https://user-images.githubusercontent.com/62214428/139073976-1eec7e25-fccb-42b1-ae52-036a7d6a2fe7.png)
- 현재 아무것도 없음
- 여기에 파일을 하나 만들어보고 / hello.txt를
- 만약 redis컨테이너에 접속해서 redis컨테이너의 cache에도 hello.txt있다면 공유된다는 것을 확인할 수 있다 / touch로 생성
- ![화면 캡처 2021-10-27 222350](https://user-images.githubusercontent.com/62214428/139074445-31d316d6-1a63-42c5-8713-3aff82ea6fdd.png)
- 공유 확인!




