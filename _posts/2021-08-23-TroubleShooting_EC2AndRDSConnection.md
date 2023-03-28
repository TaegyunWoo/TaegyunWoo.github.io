---
category: TS
tags: [TS]
title: "AWS - EC2에서 RDS 연결시, 무반응 현상과 해결방법"
date:   2021-08-23 11:30:00 
lastmod : 2021-08-23 11:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 문제 배경
- AWS의 EC2에서 RDS와 연결을 시도했다. 아래 명령어를 통해 MySql이 설치된 RDS와의 연결을 기대했다.
  ```
  $ mysql -u 계정이름 -p -h RDS_엔드포인트_주소
  ```

- 하지만 `Enter Password:` 가 출력되고 비밀번호를 입력까지 완료했지만, 연결이 되지 않고 반응이 없는 문제가 발생했다.

- 문제 상황  
  ![문제 상황](/assets/img/2021-08-23-TroubleShooting_EC2AndRDSConnection/1.png)

  - **위 상황에서 비밀번호를 입력해도 연결이 진행되지 않고 반응이 없다.**
  

<br><br>

# 문제 해결 탐색과정

- EC2와 RDS를 연결하는 방법에 대해 설명하는 AWS의 문서가 있을 것으로 생각하고 조사하였다.

- 따라서, 해당 해결방법을 참고하여 문제를 해결했다.

> [Amazon EC2 인스턴스를 배스천 호스트로 사용하여 로컬 시스템에서 프라이빗 Amazon RDS DB 인스턴스에 연결하려면 어떻게 해야 합니까?](https://aws.amazon.com/ko/premiumsupport/knowledge-center/rds-connect-ec2-bastion-host/)


<br><br>

# 문제 해결
## 문제 원인

- **EC2와 RDS가 사용하는 VPC에 대한 보안그룹에서, EC2에 대한 인바운드 규칙을 설정하지 않아서 발생한 문제이다.**

- **즉, VPC 보안그룹의 인바운드 규칙에 EC2의 프라이빗IP를 추가해야한다.**

<br><br>

## 해결순서

### 1. VPC id 확인

- RDS와 EC2의 VPC(VPC의 id)가 같은 것인지 확인한다.

- 각각의 세부정보에서 확인할 수 있다.

- 만약, 같은 VPC가 아니라면 똑같이 설정해준다.

<br/>

### 2. VPC 보안그룹 설정

- RDS의 VPC 보안그룹을 클릭한다.  
  ![VPC 보안그룹 탭](/assets/img/2021-08-23-TroubleShooting_EC2AndRDSConnection/2.png)

- 해당 보안그룹의 인바운드 규칙을 수정하기 위해 해당 탭에 들어간다.  
  ![인바운드 규칙 수정](/assets/img/2021-08-23-TroubleShooting_EC2AndRDSConnection/3.png)

- 인바운드 규칙에 다음 규칙을 추가한다.
  - **Type**: 사용자 지정 TCP 규칙
  - **Protocol**: TCP
  - **Port Range**: 본인이 연결할 RDS DB 인스턴스의 포트 (기본값: 3306)
  - **Source**: 본인의 EC2 인스턴스의 프라이빗 IP 주소 (프라이빗IP주소/32)

- 위 규칙을 추가한 뒤, 재연결을 시도하면 정상적으로 RDS에 접속할 수 있다.