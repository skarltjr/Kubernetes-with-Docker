### 쿠버네티스 명령어와 yaml
- 쿠버네티스의 모든 명령어가 다 잘 먹는건 아니다.
- 그래서 종종 파일로 관리할때가 유리할 수 있다

#### 1. 명령어로 파드를 생성하기
ex)
```
kubectl run test --image=nginx

run = 파드를 생성하겠다
test = 파드의 이름
--image = 파드를 생성하려면 이미지가 필수
```

#### 2. yaml로 생성하기
```
kubectl run test --image=nginx --dry-run=client -o yaml > kiseok.yaml

그러면 파드가 생성되는게 아니라 해당 파드를 생성할 수 있는 kiseok.yaml이 생성된것
이를 통해

kubectl apply -f kiseok.yaml
```












