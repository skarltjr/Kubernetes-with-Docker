## ConfigMap
- 컨피그맵은 키-값 쌍으로 `기밀이 아닌` 데이터를 저장하는 데 사용하는 API 오브젝트
- 컨테이너에 필요한 환경 설정 내용을 컨테이너 내부가 아닌 외부에 분리하는데 용이
- 클러스터가 구성된 Config 방식을 이해하는 데 사용할 수도 있고, 클러스터를 업그레이드 할 때 활용할 수도 있음

### 1. Config Map을 생성해보자
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config  
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```
- `data`를 보면 `SPECIAL_LEVEL : very`  == `키 : 값`
- `namespace`는 디폴트
- 이름은 `special-config`
- ![화면 캡처 2021-11-03 230610](https://user-images.githubusercontent.com/62214428/140075289-a6c1032e-089d-4429-9030-55e48131c6c4.png)

### 2. ConfigMap의 사용이유를 다시 생각해보자
```
- 컨피그맵은 키-값 쌍으로 기밀이 아닌 데이터를 저장하는 데 사용하는 API 오브젝트
- 컨테이너에 필요한 환경 설정 내용을 컨테이너 `내부가 아닌 외부`에 분리하는데 용이
```
- config를 파드마다 일일이 생성해서 파드 내부에서 관리할 수도있지만
- ![화면 캡처 2021-11-03 231032](https://user-images.githubusercontent.com/62214428/140076166-5ff35412-17ea-4fff-adfc-06449b88d96a.png)
- 그림처럼 configmap을 외부에서 만들어 관리하며 파드를 생성할 때 값만 넣어주며 사용하는 등.. 처럼 사용할 수 있다.
- 예를 들어 이미지는 같은 파든데 파드마다 config는 다르게해야하는 경우.

### 3. configMap의 사용예시 
- 이미지는 같은 파든데 파드마다 config는 다르게해야하는 경우.
- 혹은 pod를 생성할 때 `envFrom`
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/echo", "$(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_LEVEL
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_TYPE
  restartPolicy: Never
```
- 여기서 `env`를 보면 configMap을 참조한다. / 이렇게 pod를 생성할 yaml을 작성할때도 사용할 수 있다.
- 그럼 진짜 `configmap`을 참조했는지 pod를 만들고 `log`를 확인해보자
- 먼저 예상결과는 `command: [ "/bin/echo", "$(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]`에서 변수들을 `configmap`에서 참조하니까 
- 아마도 `very charm`이 나올 것
- ![화면 캡처 2021-11-03 232954](https://user-images.githubusercontent.com/62214428/140079791-12aa1d51-239f-4b60-8c5b-24d4a0d38d77.png)


