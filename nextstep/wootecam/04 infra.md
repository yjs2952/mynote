# 4주차 그럴듯한 서비스 만들기

## 1단계 - 서비스 구성하기

### 진행 과정

1. VPC 생성
    * Name : yjs2952-vpc
    * CIDR class 생성 (192 ~ 223 : 192.0.0.0 ~ 223.255.255.0) 
    * ex) 222.222.222.0/24
2. Subnet 생성
    * 외부망 64개씩 2개 (AZ를 다르게 구성)
        + Name : yjs2952-public1 / IPv4 CIDR : 222.222.222.0/26 / 가용영역 : ap-northeast-2a
        + Name : yjs2952-public2 / IPv4 CIDR : 222.222.222.64/26 / 가용영역 : ap-northeast-2c
    * 내부망 32개씩 2개
        + Name : yjs2952-private / IPv4 CIDR : 222.222.222.128/27 / 가용영역 : ap-northeast-2c
        + Name : yjs2952-manage / IPv4 CIDR : 222.222.222.160/27 / 가용영역 : ap-northeast-2c
3. Internet Gateway 연결
    * Internet Gateway 생성
        + Name : yjs2952-internet-gateway
    * VPC 연결 (yjs2952-vpc)
4. Route Table 생성
    * 라우팅 -> internet gateway (0.0.0.0/0) public, manage 만
    * 서브넷 연결
5. Security Group 설정
    * 외부망
        + 전체 대역(0.0.0.0/0) : 8080 포트 오픈
        + 관리망 : 22번 포트 오픈
    * 내부망
        + 외부망 : 3306 포트 오픈
        + 관리망 : 22번 포트 오픈
    * 관리망
        + 자신의 공인 IP : 22번 포트 오픈
6. EC2 서버 생성
    * web-service (public)
    * db (private)
    * bastion (manage)