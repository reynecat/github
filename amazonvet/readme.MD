# AmazonVet - PetClinic
- 원본 출처: https://github.com/SteveKimbespin/petclinic_btc


## 환경 요구사항
- **OpenJDK 25**
- **Apache Tomcat 11.x**
- **Maven 3.9.11+**

## 주요 변경사항
- Java 25 지원
- Tomcat 11 (Jakarta EE 11) 지원
- Spring Framework 6.2.1
- Hibernate 6.6.4
- H2 인메모리 DB 기본 설정 (단일 인스턴스 실행)
- MariaDB 외부 연결 옵션 추가

## 빌드 및 실행

### 1. 기본 실행 (H2 인메모리 DB)
단일 인스턴스에서 실행하며, 애플리케이션 재시작 시 데이터가 초기화됩니다.

```bash
# WAR 파일 빌드
./mvnw clean package

# 또는 기본 프로필 명시
./mvnw clean package -P H2
```

빌드된 WAR 파일: `target/petclinic.war`

### 2. MariaDB 외부 연결

#### MariaDB 설정
```sql
CREATE DATABASE petclinic CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'petclinic'@'%' IDENTIFIED BY 'petclinic';
GRANT ALL PRIVILEGES ON petclinic.* TO 'petclinic'@'%';
FLUSH PRIVILEGES;
```

#### 연결 정보 수정
`pom.xml`의 MariaDB 프로필에서 연결 정보 수정:
```xml
<profile>
    <id>MariaDB</id>
    <properties>
        <jdbc.url>jdbc:mariadb://YOUR_HOST:3306/petclinic?useUnicode=true&amp;characterEncoding=utf8</jdbc.url>
        <jdbc.username>YOUR_USERNAME</jdbc.username>
        <jdbc.password>YOUR_PASSWORD</jdbc.password>
    </properties>
</profile>
```

#### 빌드
```bash
./mvnw clean package -P MariaDB
```

### 3. MySQL AWS RDS 연결 (기존 설정 유지)
```bash
./mvnw clean package -P MySQL
```

## Tomcat 11 배포

### 수동 배포
1. Tomcat 11 다운로드 및 설치
2. `target/petclinic.war`를 Tomcat의 `webapps/` 디렉토리에 복사
3. Tomcat 시작: `catalina.sh run` (또는 `catalina.bat`)

### Manager App을 통한 배포
1. Tomcat의 `conf/tomcat-users.xml` 설정:
```xml
<tomcat-users>
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <user username="admin" password="admin" roles="manager-gui,manager-script"/>
</tomcat-users>
```

2. Maven을 통한 배포:
```bash
# 최초 배포
./mvnw cargo:deploy

# 재배포
./mvnw cargo:redeploy

# 특정 프로필로 재배포
./mvnw cargo:redeploy -P MariaDB
```

## 접속
애플리케이션 접속: http://localhost:8080/petclinic

## 프로필 선택 가이드

| 프로필 | 용도 | DB 위치 | 데이터 지속성 |
|--------|------|---------|---------------|
| **H2** (기본) | 개발/테스트 | 메모리 | 재시작 시 초기화 |
| **MariaDB** | 프로덕션 | 외부 서버 | 지속 |
| **MySQL** | AWS RDS | 외부 서버 | 지속 |

## 트러블슈팅

### Java 버전 확인
```bash
java -version
# 출력: openjdk version "25" ...
```

### Maven 버전 확인
```bash
./mvnw -v
# Maven 3.9.11+
```

### Tomcat 버전 확인
Tomcat의 `lib/` 디렉토리에서 `catalina.jar` 확인
- 파일명에 버전 표시: `catalina-11.0.x.jar`

### 포트 충돌
Tomcat의 `conf/server.xml`에서 포트 변경:
```xml
<Connector port="8080" protocol="HTTP/1.1" ... />
```

## 참고사항
- H2 콘솔은 기본적으로 비활성화되어 있습니다
- 프로덕션 환경에서는 반드시 외부 DB(MariaDB/MySQL) 사용 권장
- 보안을 위해 DB 비밀번호를 환경 변수로 관리하는 것을 권장합니다