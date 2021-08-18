# 4. 도커 이미지

도커 이미지는 애플리케이션 자신 뿐만아니라 실행에 필요한 모든 것들을 담고 있다. OS, DBMS, 웹서버 등 시스템 상에서 이용되는 대부분의 요소를 이미지로 생성할 수 있다.

## 4.1 도커 이미지 개요
```bash
sudo docker container run -it --name ubuntu ubuntu:latest
```
- 기본적으로 로컬에는 필요한 도커 이미지가 존재하지 않는다.
- 여기서 마지막 ubuntu를 보자
- 
