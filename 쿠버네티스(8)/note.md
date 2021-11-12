## [Operation] Logging & Troubleshooting, Backup
### Logging & Troubleshooting
![화면 캡처 2021-11-12 214951](https://user-images.githubusercontent.com/62214428/141469892-8c6fce81-c3f3-4226-a26a-e21bb592d108.png)


### 쿠버네티스의 기본적인 로깅
- (https://kubernetes.io/ko/docs/concepts/cluster-administration/logging/#쿠버네티스의-기본-로깅)
- ★ logs
- 쿠버네티스에선 컨테이너의 로그를 볼 때 
   - `kubectl logs <container>`를 활용했다 

-----

### 쿠버네티스의 로깅 아키텍쳐
#### 1. 노드 레벨에서의 로깅
- ![화면 캡처 2021-11-12 220450](https://user-images.githubusercontent.com/62214428/141471690-ea786f29-a5f2-418d-975d-733b675a5868.png)
- 이 경우는 바로 노드 레벨에서의 로깅인데 
- 리눅스 기반에서는 /var/log에 모든 로그가 존재한다
- `그래서 kubectl get pod -o wide로 로그를 보고싶은 pod의 노드를 확인하고 ex)워커 2`
- `해당 노드에 접속해서 루트로 /var/log에 접속해보면 모든 로그가 남아있다`
- ![화면 캡처 2021-11-12 220912](https://user-images.githubusercontent.com/62214428/141472214-03db6324-560e-44c6-a90e-13ceec2483e7.png)
- 이런식으로 확인해보면 나오는 결과는 모두 symbolic link로 로그 파일을 확인하려면 끝까지 따라가면 된다
- 이게 노드 레벨에서의 로깅

#### 2. 클러스터 레벨에서의 로깅
- ![화면 캡처 2021-11-12 221637](https://user-images.githubusercontent.com/62214428/141473171-ac3b1bbb-ca14-45b3-9ed9-a718a9820c3f.png)
- 이런 경우는 표시된 pod처럼 로깅을 위한 별도의 pod를 띄우고 
- 이 pod가 log파일에 있는 로그들을 수집해서 가져다가 
- kibana등의 툴을 활용하여 로그를 확인할수도있다.


참고로 당연히 로그를 수집해서 다른 툴을 활용하여 보기 좋게 표현하는등은 다 옛날부터 했던 일. 여기서 쿠버네티스는 로그 수집 pod로 띄워서 하는 등이 조금 다른것일뿐














