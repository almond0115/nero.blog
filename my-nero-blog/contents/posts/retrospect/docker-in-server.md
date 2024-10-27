---
title: "프로덕션 환경의 Docker MySQL DB 도입기"
description: "Docker MySQL에서 마주친 이슈들과 해결 방법"
date: 2024-10-27
update: 2024-10-27
tags:
  - docker
  - mysql
  - db
  - devOps
series: "글또 10기"
---

## 들어가며

운영 서버에 Docker로 MySQL을 안전하게 구축하는 법을 공유해보려합니다 👀

이 글에서는 제가 실제 프로덕션 환경에서 겪은 경험을 바탕으로, 안전하고 효율적인 Docker MySQL 구축 방법을 단계별로 살펴보겠습니다.

## 🧑🏻‍💻 Docker MySQL 구축 기본 원칙

프로덕션 환경에서 DB를 가장 안전하게 사용하는 방법은 AWS를 사용하신다면 `RDS`를 사용하는 게 가장 안전하고 빠를 것입니다. <br>
하지만 RDS는 비용이 발생하고 대신 Docker 를 통해 DB를 띄워서 사용하신다면 다음의 원칙들을 준수하는 것이 좋습니다.

1. 최소 권한 원칙 (Principle of Least Privilege)
2. 설정의 단계적 적용
3. 볼륨 관리를 통한 데이터 영속성 보장
4. 보안 설정의 명확한 문서화

## 🤔 처음부터 꼬인 설정의 늪

서버에서 Docker로 MySQL을 세팅하면서 가장 많이 본 화면이 있습니다.<br>

![그림 0. Access denied for user](./images/image1.png)

구글링을 해보아도 위 접근 권한을 해결하기 위한 수많은 포스팅 글이 존재합니다. 오늘은 제가 이 에러 화면과 이별하기까지의 여정을 공유하려고 합니다. 더 이상 이 에러로 고통받지 않도록, 제가 쉽게 세팅하고 안전하게 사용할 수 있도록 그 해결 과정을 정리해보았습니다.

## 🔌 기본 환경 구성

<div style="color: black; background-color: #e6f3ff; border: 1px solid #b3d9ff; padding: 15px; margin: 20px 0; border-radius: 5px;">
  <strong>💡 최소한의 설정으로 시작하기 </strong> <br>
</div>

```yaml
version: '3.8'

services:
  mysql:
    container_name: production-db
    image: mysql:latest
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: db-name
      TZ: Asia/Seoul
      character-set-server: 'utf8mb4'
      collation-server: 'utf8mb4_unicode_ci'
    ports:
      - '3306:3306'
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/conf.d:/etc/mysql/conf.d
    command:
        - "mysqld"
        - "--character-set-server=utf8mb4"
        - "--collation-server=utf8mb4_unicode_ci"

```

  가장 기본적인 세팅으로 다음은 제가 최종적으로 도달한 기본 docker-compose.yml 설정입니다. <br> 이렇게 최소한의 접속이 보장되도록 먼저 설정하는게 좋습니다.


## 🔨 단계별로 하나씩 해결하기

### 1. 도커 컨테이너 실행하기

```yaml
# Docker 컨테이너 올리기
docker-compose up --build -d

# Docker 컨테이너 확인
docker ps
```

여기서 팁 하나! -d 옵션을 꼭 붙이세요.
이 옵션이 있어야 Docker가 백그라운드에서 실행됩니다.

### 2. MySQL 접속 확인하기

![그림 1. docker status](./images/image2.png)

위 이미지처럼 도커가 잘 작동한다면 해당 MYSQL가 잘 동작한다는 의미입니다. 이제 접속만 잘 되기를 기도하면 됩니다.

```yaml
# 서버 내부에서 Docker로 띄운 DB 접속하기
docker exec -it sbpb-db mysql -u root -p
```

<div style="color: black; background-color: #d4edda; border: 1px solid #c3e6cb; padding: 15px; margin: 20px 0; border-radius: 5px;">
    <strong>⚠️ 여기서 잠깐! </strong> <br> 이렇게 기본 설정만 하고 방치하면 프로덕션 환경에서는 DB 해킹을 당합니다. <br>
</div>

### 3. DB 보안 강화하기

먼저 root 계정이 어떤 호스트에서 접근 가능한지 확인해봅시다!

아래 명령어로 '%'가 아닌 다른 호스트에 대한 root 계정 사용 유무를 확인해주어야 합니다.


```sql
SELECT user, host FROM mysql.user WHERE user='root';
```

![그림 1. root](./images/image3.png)

위 명령어에서 확인한 모든 Host의 root 계정 비밀번호를 다음 명령어 순서로 변경해주고 Docker를 재시작 해줍니다.

```sql
# 모든 root 비밀번호 변경
ALTER USER 'root'@'localhost' IDENTIFIED BY '새로운_복잡한_비밀번호';
ALTER USER 'root'@'%' IDENTIFIED BY '새로운_복잡한_비밀번호';
FLUSH PRIVILEGES;
```

```yaml
# Docker 재시작
docker-compose down -v
docker-compose up -d
```

## 🌟 외부 접속 설정: 편리함과 보안 사이의 균형

이제 매번 서버에 접속하여 DB에 접속하기 싫다면 외부에서 접속할 수 있도록 다음 세팅을 해주어야 합니다.

매번 서버에 접속해서 DB를 들여다보는 건 너무 불편하죠.
외부에서도 안전하게 접속할 수 있도록 새로운 사용자를 만들어봅시다.

저는 'sbpb' 이라는 유저를 등록해주었습니다. 제가 사용할 유저이기 때문에 모든 호스트 접속 설정까지 해두었습니다.

```sql
# 새로운 사용자에게 localhost에서의 접속 권한 부여
CREATE USER 'sbpb'@'localhost' IDENTIFIED BY '새로운 복잡한 비밀번호';
GRANT ALL PRIVILEGES ON geultto.* TO 'sbpb'@'localhost';
FLUSH PRIVILEGES;

# 새로운 사용자에게 필요한 권한만 부여
CREATE USER 'sbpb'@'%' IDENTIFIED BY '새로운_복잡한_비밀번호';
GRANT ALL PRIVILEGES ON geultto.* TO 'sbpb'@'%';
FLUSH PRIVILEGES;
```

이제 로컬 환경에서 새로운 터미널을 열고 다음의 접속 명령어를 작성해줍니다.

```
mysql -h 서버_IP주소 -P MYSQL_포트번호 -u MYSQL_유저 -p MYSQL_비밀번호 MYSQLDB_이름
```

## 🔒 마지막으로, 파일 권한도 신경 씁시다

해킹 이후 배운 교훈 중 하나는 파일 권한의 중요성입니다.

다음과 같이 중요 파일들의 권한을 제한해두세요!

```bash
# docker-compose.yml 파일과 .env 파일은 소유자만 읽고 쓸 수 있도록 설정
chmod 600 docker-compose.yml
chmod 600 .env

# mysql 디렉토리 및 내부 파일 권한 설정
chmod -R 600 mysql
chmod -R root:root mysql
```

## 💪 귀찮은 건 자동화하자

매번 로컬에서 서버 DB에 긴 접속 명령어를 입력하기 너무 귀찮습니다.

저는 다음과 같은 쉘 스크립트를 만들어서 사용하고 있습니다 👾

```bash
#!/bin/bash

# 변수 정의하기
SERVER_IP="IP 주소"
MYSQL_PORT="포트번호"
MYSQL_USER="유저이름"
MYSQL_PASSWORD="복잡한 비밀번호"
MYSQL_DATABASE="DB 이름"

# MySQL database 연결하기
mysql -h $SERVER_IP -P $MYSQL_PORT -u $MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE
```

위 코드로 쉘 파일을 만들었다면 다음 권한 부여도 잊지 말아야 합니다.

```bash
chmod +x {쉘파일명}.sh
```

위 설정을 해두면 이제 아래 명령어로 쉽게 접속이 가능합니다!
```bash
./{쉘파일이름}.sh
```

## 🪜 Docker DB 초기화

그럼에도 설정 과정에서 문제가 발생한다면.. 

리셋이 필요한 경우 다음 명령어를 사용하면 좋습니다.

```bash
# Docker 컨테이너 볼륨 제거
docker-compose down -v
 
# Docker 컨테이너 기존 볼륨 확인
docker volume ls
 
# 모든 볼륨 확인 및 제거
docker volume rm <volume_name>
 
# 사용하지 않는 모든 볼륨 제거
docker volume prune -f
```

그리고 docker-compose.yml 에서 만들어진 파일을 제거하고 다음 명령어로 새롭게 생성합니다!
```bash
# 기본 mysql 데이터 디렉토리 제거
rm -rf /mysql
 
# Docker 컨테이너 새롭게 생성
docker-compose up --build --force-recreate -d
```


## 마치며

이렇게 해서 제 Docker MySQL 도입기를 마무리합니다. 처음에는 단순해 보였던 작업이 이것저것 신경 쓸 게 많았네요.

여러분도 제가 겪은 시행착오를 피해서, 안전하고 편리한 Docker MySQL 환경을 구축하시길 바랍니다!
혹시 비슷한 경험이 있으시다면 댓글로 공유해주세요.
다른 개발자분들의 경험담도 궁금합니다. 😊