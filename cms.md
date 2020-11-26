# H1 App Store API 및 관리자 페이지 개발 환경 구성 가이드

## 목차

1. [소개](#1-소개)
2. [개발 환경 구성](#2-개발-환경-구성)
3. [관리자 페이지 빌드하기](#3-관리자-페이지-빌드하기)
4. [API 서비스 빌드하기](#4-API-서비스-빌드하기)

### 1. 소개

본 서비스는 H1 구독 경제 서비스의 API 서비스와 관리자 페이지를 포함하고 있습니다.  
API 서비스는 Java 1.8 기반의 Spring boot 로 구성되어 있고, 관리자 페이지는 리액트를 이용한 웹 애플리케이션 형태로 구성되어 있습니다.  
웹 애플리케이션은 모노로직하게 구성되어 있지 않으므로, 단독 서비스로도 구축이 가능합니다.  
다만 본 서비스에는 운영상의 편의를 위해 정적 웹 애플리케이션으로 빌드해 API 서비스 내의 정적 리소스로 탑재하도록 구성했습니다.

#### 필요 요소

* Java 1.8
* MySQL 5.7

### 2. 개발 환경 구성

스프링 부트는 [Spring Initializr][https://start.spring.io/]를 이용해 생성되었습니다.  
스프링 이니셜라이저는 특정 IDE에 대한 디펜던시가 없으므로 이클립스, 인텔리제이, VS Code 등 어느 개발 환경을 이용해서든 개발 환경을 구성할 수 있습니다.  
Maven CLI를 이용할 수 있으므로 IDE의 메이븐 관리 도구를 꼭 이용하지 않아도 됩니다. 다만 본 가이드에서는 인텔리제이를 기준으로 설명합니다.

인텔리제이의 프로젝트 오픈 기능으로 프로젝트를 엽니다.  
`pom.xml`의 내용에 따라 프로젝트를 인식하므로 별도로 프로젝트에 대한 설정을 하지 않아도 Run, Debug, Profile 등의 기능이 활성화 됩니다.  
프로젝트가 인식되면 메이븐의 디펜던시 설치가 진행됩니다.

#### 데이터베이스 준비

h1appstore 라는 이름으로 데이터베이스를 생성합니다.  
권장되는 charset은 utf8 입니다.

```sql
create database h1appstore default character set utf8 collate utf8_general_ci;
```

#### application.properties 준비

application.properties 파일에 데이터베이스에 관련된 내용을 입력합니다.  
변경하지 않았다면 기본값으로 이용해도 무관한 부분이지만, 실행 전 체크해보는 것이 좋습니다.

* 데이터베이스 관련
  * spring.datasource.url
  * spring.datasource.username
  * spring.datasource.password
* 인증 관련
  * security.jwt.token.secret-key
* SSL 관련
  * server.ssl.enabled - 로컬 개발시 인증서가 없다면 비활성 해야 할 수 있습니다
  * server.ssl.key-store
  * server.ssl.key-store
  * server.ssl.key-store

#### 데이터베이스 마이그레이션

본 프로젝트는 JPA를 이용해 구성되었으므로,  
서버 구동시 JPA가 도메인 스트럭쳐를 분석해 데이터베이스를 마이그레이션 합니다.  
따라서 별도의 마이그레이션을 수동으로 수행할 필요는 없습니다.

#### 최초 관리자 계정 생성

**주의: 관리자 계정 생성은 데이터 마이그레이션 이후에 진행되어야 하니, 최소 1회 이상 서버가 구동되어 데이터 마이그레이션이 진행된 후 계정 생성을 시도해야 합니다.**

최소 1개의 관리자 계정이 있어야 관리자 페이지 로그인이 가능하므로, 계정을 생성합니다.

```sql
INSERT INTO `managers` ( `created_at`, `email`, `mobile`, `name`, `password`, `role`, `updated_at`, `user_id`)
VALUES
	( CURRENT_TIMESTAMP, 'admin@admin.com', '01011112222', 'admin', '$2a$10$G.7QZr9SgdUXq07KztoMFuNq6qrE4njdz9iZrUGkLX6Xr8Yet/lTq', 'ADMIN', CURRENT_TIMESTAMP, 'admin');
```

암호화에는 bcrypt가 사용됩니다. 기본 비밀번호가 아닌 직접 지정한 비밀번호로 계정을 생성하려면 bcrypt로 해시된 값을 password 필드에 할당하시기 바랍니다.

### 3. 관리자 페이지 빌드하기

관리자 페이지의 원본 소스는 `back-office` 디렉토리 하위에 있습니다.  
최종적으로 빌드된 웹 애플리케이션은 `src/main/resources/static` 하위에 생성됩니다.

```shell script
cd back-office
npm run build
```

빌드 스크립트 내에 자바 정적 리소스 내부로 이동하는 명령까지 포함되어 있으므로 빌드 외에 다른 처리를 더 하실 필요는 없습니다.

### 4. API 서비스 빌드하기

인텔리제이 내의 Build Project 기능을 통해 프로젝트를 빌드할 수 있습니다.  
하지만 실제적으로 서비스 빌드는 리눅스 서버상에 하는 경우가 많습니다.  
그럴 경우에는 Maven CLI를 이용하는 것이 좋습니다.

메이븐 CLI에서 `package` 명령을 통해 서비스를 빌드할 수 있습니다.

```shell script
mvn package
```

빌드된 jar 파일은 `target/Toki-H1-0.0.1-SNAPSHOT.jar` 경로상에 생성됩니다. 

끝.
