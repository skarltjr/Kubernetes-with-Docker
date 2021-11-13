## Troubleshooting
#### 과제 
- pod생성이 막힌 상황에서 쿠버네티스 구조를 떠올리며 log를 통해 왜 생성이 안되는지 원인을 찾고 해결해보자
- ![화면 캡처 2021-11-13 151840](https://user-images.githubusercontent.com/62214428/141608361-ff6b3aa3-9122-4382-b02e-185a4617870b.png)
- 원인을 찾기 위해 describe 찍어보자
- ![화면 캡처 2021-11-13 152114](https://user-images.githubusercontent.com/62214428/141608475-59b5279e-9204-4907-a91a-d3ff7c68f932.png)
- 노드에 taint가 묻어있는 것 같다. 확인을 해보자
- 동작할 pod가 생성될 곳은 worker / worker describe해보자
-  ![화면 캡처 2021-11-13 154803](https://user-images.githubusercontent.com/62214428/141609152-95c0b8ab-e394-4361-8ad5-6cfd2100c1a2.png)

#### 해결법 - 
- 
#### 확인 
- 참고로 원래 마스터노드엔 noSchedule이다. 
- 

----------------


## 백업 및 복구
#### 과제




















