### Secrets, configmap 마운트

![화면 캡처 2021-12-01 172726](https://user-images.githubusercontent.com/62214428/144198748-a1b2109a-de42-445a-bc0b-5cd64c24beb7.png)


#### 1. configMap 실습해보기
- 먼저 yaml을 파악해보자
    - kind : configMap
    - ★`redis-config`부분을 봐보자
    - redis-config라는 `key`
    - maxmemory 2mb라는 `value`
    - maxmemory-policy allkeys-lru라는 `value`
```
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru    
EOF
```

### 1-1. yaml을 적용해보자
- `kubectl apply -f yaml`
- ![화면 캡처 2021-12-01 174400](https://user-images.githubusercontent.com/62214428/144201129-69b1da81-6a3e-4df1-b301-b0a9a85c96eb.png)
- ![화면 캡처 2021-12-01 174501](https://user-images.githubusercontent.com/62214428/144201263-bcdd28ea-a938-4080-883f-239dce4aa944.png)

### 1-2 configMap을 적용할 pod 생성
```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf
```
- `volumes`부분을 봐보자
- example-redis-config라는 configMap의 `redis-config` key-value를 사용할건데
- `path`를 봐보면 이 configMap정보를 `redis.conf`에 적용하겠다
- ★ 그리고 `volume`들이 어디에 mount될지 `volumeMounts`를 살펴보면
- emptyDir는 파드의 `/redis-master-data`에
- config는 `/redis-master`에 마운트
- ★마지막으로 `command`를 봐보자
- 레디스 서버가 동작할 때 해당 conf를 적용시킬 것

### 1-3. 확인을 해보자
- `kubectl exec -it redis -- sh`
- ![화면 캡처 2021-12-01 175438](https://user-images.githubusercontent.com/62214428/144202769-f034c357-db3a-4e08-ad70-8640f7b9c45f.png)
- 실제 적용이 되었는지도 확인해보자
```
kubectl exec -it redis -- redis-cli

127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"

127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
```
- ![화면 캡처 2021-12-01 175715](https://user-images.githubusercontent.com/62214428/144203093-8bbf4488-c0f2-4a5a-b8ca-73f4785fb59f.png)


### 2. secrets 실습해보기
#### 2-1. secret 생성 후 확인
- username이라는 파일을 만들고 admin을 작성
- password 동일
- 이를 통해 secret 생성
```
echo -n admin > username
echo -n 1q2w3e > password

kubectl create secret generic mysecret --from-file=username --from-file=password
```
- ![화면 캡처 2021-12-01 180159](https://user-images.githubusercontent.com/62214428/144203922-80460d66-a0ed-4af8-b9c3-056dbd1d55bb.png)

#### 2-2 secret를 설정한 pod를 생성해보자
- yaml먼저 확인
- volume => secret으로 방금 만든 myscecret사용할 것
- volumeMounts => `/etc/foo`에
- ★참고로 myscrete이 usernamee , password파일로 구성되어있는데 volume에서 따로 path를 지정해주지 않았으니 그냥 파일이름을 따를 것
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
EOF
```
#### 2-3 확인해보자
![화면 캡처 2021-12-01 180905](https://user-images.githubusercontent.com/62214428/144205091-58f39a1e-7819-4c66-a9ff-5068e6a449f3.png)







