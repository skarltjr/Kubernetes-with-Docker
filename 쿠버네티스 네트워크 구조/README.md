## pod-pod, pod-service 통신
- ![image](https://user-images.githubusercontent.com/62214428/222945798-86a9716f-6a66-48f1-b061-016f463dcc34.png)

## 외부 - service 통신
- <img width="728" alt="스크린샷 2023-03-05 오전 3 04 45" src="https://user-images.githubusercontent.com/62214428/222921784-ab06a89b-0799-46a9-b02e-26dbe2164e65.png">

1. 외부에서 service로 접근할 수 있도록 하는 nodeport의 경우 각 노드의 nic마다 port를 부여한다.
2. 해당 포트로 들어오는 요청은 모두 해당 포트를 가진 노드로 진입한다.
3. 그리고 이 주소를 내부간 통신에서는 cluster ip(서비스)로 소통하기 때문에 쿠베프록시를 통해 여기서는(10.3.241.~)로 변환한다.
4. 그리고 이 cluster ip(서비스)에 연결된 pod로 보낸다.
