<div style="text-align: center;">
  <img src="https://raw.githubusercontent.com/KT-HOO/KTravel/main/img/Ktravel.png" width="330" height="240" />
</div>


# 여행계획 생성 및 공유 서비스(Kt-ravel)

Kt-ravel은 다양한 사용자들이 여행 계획을 생성하고 공유할 수 있는 MSA 기반의 플랫폼입니다.   
사용자는 각자 다른 계획을 만들고, 다른 사용자의 계획을 팔로우하며, 크레딧 시스템을 통해 활동할 수 있습니다.

@@@@@ 서비스 시나리오 더 구체적으로 작성 (연결관계 반영)

## 목차

**I. [서비스 시나리오](#서비스-시나리오)**    
 - [기능적 요구사항](#기능적-요구사항functional-requirements)
 - [비기능적 요구사항](#비기능적-요구사항-non-functional-requirements)

**II. [체크포인트](#체크포인트)**    
 - [아키텍처 설계(IaaS)](#아키텍처-설계iaas)
 - [Data Modeling/서비스 분리/설계 역량(BIZ)](#data-modeling서비스-분리설계-역량biz-15점)
 - [MSA개발 또는 개발관리 역량](#msa개발-또는-개발관리-역량-20점)
 - [클라우드 배포 역량](#클라우드-배포-역량-20점)

**III. [아키텍처 설계(IaaS)](#아키텍처 설계(IaaS))**    
 - [아키텍처](#아키텍처)
 - [데이터](#데이터)
 - [API](#API)
 - [보안](#보안)
 - [확장성 고려](#확장성 고려)

**IIII. [Data Modeling/서비스 분리/설계(BIZ)](#Data Modeling/서비스 분리/설계(BIZ))**    
 - [MSAEZ 모델링(Event Storming 결과)](#MSAEZ 모델링(Event Storming 결과))
 - [주요 마이크로서비스 구조](#주요 마이크로서비스 구조)
 - [헥사고날 아키텍처 다이어그램 도출)](#헥사고날 아키텍처 다이어그램 도출)

**V. [MSA개발 및 개발관리](#MSA개발 및 개발관리)**    
 - [개발 환경 구축 ](#개발 환경 구축)
 - [백엔드 마이크로서비스 개발](#백엔드 마이크로서비스 개발)
 - [프론트엔드](#프론트엔드)

**VI. [클라우드 배포](#클라우드-배포)**    


  
## I. 서비스 시나리오

### 기능적 요구사항(Functional Requirements)

### 1. 회원 가입(Sign up) 및 토큰(Token) 관리   
   **1.1 회원 가입 및 로그인**  
   - 사용자는 구글 또는 네이버 계정을 통해 회원가입 및 로그인이 가능해야 한다.

   **1.2 토큰(크레딧)**  
   - 사용자는 충전/결제 시스템을 통해 토큰을 획득할 수 있어야 한다.
   - 토큰은 여행 계획 추천 기능에 사용될 수 있어야 한다.

### 2. 여행 계획(Plan) 생성, 수정, 삭제    
   **2.1 여행 계획 생성**  
   - 사용자는 목적지(location), 날짜(travelDate), 예산(budget), 인원수(groupSize), 세부 활동(details) 등의 상세 내용을 포함한 여행 계획을 AI를 활용하여 생성할 수 있어야 한다.
   - 목적지(lacation), 인원수(groupSize)는 필수적으로 입력되야 한다.
   
   **2.2 여행 계획 수정**    
   - 사용자는 이미 생성한 여행 계획을 수정할 수 있어야 한다.

   **2.3 여행 계획 삭제**    
   - 사용자는 이미 생성한 여행 계획을 삭제할 수 있어야 한다.

   **2.4 여행 계획 공유**    
   - 계획은 '여행 계획 공유 게시판'에서 공유될 수 있어야 한다. 

### 3. 추천(Recommendation) 시스템    
   **3.1 사용자 맞춤형 추천**  
   - 사용자의 개인적인 계획에 따라 맞춤형 여행 계획을 추천할 수 있어야 한다.
   - 필수 입력 값: 목적지(location), 날짜(travelDate), 예산(budget), 인원수(groupSize)
   - 선택 입력 값: 세부 활동(details)
   
   **3.2 트랜드 기반 추천**  
   - 팔로우 수가 많은 상위 사용자의 여행 계획에 따라, 여행 계획을 추천할 수 있어야 한다.
   - 사용자의 팔로우 여부와 관련 없이 동작해야 한다.
   - 필수 입력 값: 목적지(location), 날짜(travelDate), 인원수(groupSize)

   **3.3 사용자 취향 기반 추천**
   - 저장된 계획 중 내 취향에 맞는 계획을 '하트' 기능이 있어야 한다.
   - '내가 하트를 누른 계획'의 planId
   - 취향: 해당 plan의 details 정보 기반
     -- 이 내용은 내가 짠 여행 계획의 세부사항이야. 참고해서 다음 여행계획을 자유롭게 짜줘.
     -- 필수적으로 포함될 내용은 다음과 같아 [목적지(location), 날짜(travelDate), 예산(budget), 인원수(groupSize), 세부 활동(details)]

   **3.4 추천 여행 계획 저장**
   - 각 추천에는 1개의 토큰이 소모되어야 한다.
   - 추천된 모든 계획은 '계획'에서 확인 가능해야 한다. 

### 4. 사용자 팔로우/언팔로우(Follow/Unfollow)    
   **4.1 팔로우 기능**  
   - 사용자는 다른 사용자를 팔로우할 수 있다.
   
   **4.2 언팔로우 기능**  
   - 더 이상 관심이 없는 사용자를 언팔로우할 수 있다.
   
   **4.3 팔로우 대시보드** 
   - 팔로우한 사용자 목록은 별도 대시보드에서 확인할 수 있어야 한다.

### 5. 알림(Notification) 기능    
   **5.1 계획 생성 알림**  
   - 팔로우한 사용자가 계획을 생성할 경우, 팔로워에게 시스템에서 알림을 발송한다.
   
   **5.2 팔로우 알림**  
   - 사용자가 다른 사용자를 팔로우할 경우, 다른 사용자에게 시스템에서 알림을 발송한다.



### 비기능적 요구사항 (Non-functional Requirements)

### 1. 트랜잭션 일관성 유지    

   **1.1 데이터 일관성**  
   - 사용자 토큰 사용, 여행 계획 생성/수정/삭제 등의 트랜잭션이 중단되지 않고 일관성을 유지해야 한다.
   - 분산 트랜잭션 처리 메커니즘을 통해 이를 보장한다.
   
   **1.2 ACID 속성 보장**  
   - 중요한 데이터(회원 정보, 여행 계획, 토큰 내역 등)에 대해 ACID 속성을 보장하는 데이터베이스 구조를 구현한다.

### 2. 서비스 간 느슨한 결합    

   **2.1 MSA**  
   - 각 기능이 독립된 마이크로서비스로 운영되며, 서로 간의 결합도를 최소화하여 유지보수 및 확장성을 높여야 한다.
   
   **2.2 API 기반 통신**  
   - 마이크로서비스 간의 통신은 RESTful API 기반으로 이루어져야 하며, 서비스의 장애나 업데이트가 다른 서비스에 영향을 미치지 않도록 설계해야 한다.

### 3. 장애 격리    

   **3.1 장애 감지 및 복구**  
   - 특정 서비스가 실패하더라도 전체 시스템이 영향을 받지 않도록 장애 격리 메커니즘을 구현해야 한다.
   
   **3.2 로그 관리**  
   - 장애를 감지하고, 이를 분석하여 빠르게 대응할 수 있는 로그 관리 시스템을 구현해야 한다.

### 4. 확장성 및 유연성    

   **4.1 자동 확장**  
   - 사용자 수 증가에 따라 자동으로 서버 리소스를 확장할 수 있는 기능을 갖추어야 한다.
   - 이를 위해 컨테이너화된 서비스나 클라우드 인프라를 사용할 수 있다.
   
   **4.2 유연한 데이터베이스 확장**  
   - 데이터베이스는 수평적 확장을 지원하며, 많은 양의 사용자 데이터를 효율적으로 처리할 수 있어야 한다.
   
   **4.3 플랫폼 확장**  
   - 새로운 기능이 추가되더라도 기존 시스템에 영향을 주지 않고 손쉽게 확장할 수 있는 유연한 아키텍처를 적용해야 한다.

### 5. 보안 및 개인정보 보호    

   **5.1 데이터 암호화**  
   - 사용자 정보 및 민감한 데이터는 전송 시 암호화해야 한다.
   
   **5.2 접근 제어**  
   - 각 서비스에 대한 접근 권한을 엄격히 관리하며, 사용자 인증 및 권한 부여를 통해 보호해야 한다.
   
   **5.3 개인정보 보호**  
   - 사용자의 여행 계획과 같은 개인정보는 외부에 노출되지 않도록 정책을 설정하고 준수해야 한다.


## II. 체크포인트

### 1. 아키텍처 설계(IaaS) (15점)
  - 클라우드 아키텍처 구성
  - MSA 아키텍처 구성
    - 각 마이크로서비스가 독립적으로 배포 및 운영될 수 있도록 설계되었는가?
    - 각 서비스의 기능 및 책임이 명확히 분리되어 있는가?

### 2. Data Modeling/서비스 분리/설계 역량(BIZ) (15점) 
  - 도메인분석-이벤트스토밍
    - MSA분석/설계 DDD기반의 이벤트스토밍 및 분석 도구를 통하여 구체적인 내용으로 도메인이 분석되고 구현 가능한 방법으로 설계 되었는가?
    - 각 도메인 이벤트가 의미있는 수준으로 정의 되었는가? 

### 3. MSA개발 또는 개발관리 역량 (20점)
  - 분산트랜잭션-Saga
  - 보상처리-Compensation
  - 단일 진입점-Gateway
  - 분산 데이터 프로젝션
    - 각 로컬 트랜잭션이 순차적으로 실행되는지, 트랜잭션 간의 흐름이 적절하게 관리되는가?
    - 트랜잭션 실패 시 실행되는 보상 트랜잭션이 명확하게 정의 되어 있는가?
    - Gateway가 정상적으로 동작하여 트래픽 제어를 할 수 있는가?
    - 분산된 시스템 간 데이터 일관성을 유지하기 위한 동기화 전략이 적절한가(CQRS)?

### 4. 클라우드 배포 역량 (20점)
  - Docker Build 이미지 생성
  - Docker 이미지 배포(Docker Hub 사용)
  - K8S 배포(Azure)
    - Docker 이미지가 최적화되었는가?
    - Docker Hub로 이미지 배포를 하였는가?
    - Kubernetes 배포가 정상적으로 되었는가?

## III. 아키텍처 설계(IaaS)

### 1. 아키텍처     
   - MSA 기반 설계:
     - Spring Boot 마이크로서비스
     - Vue.js 프론트엔드
   - Azure 클라우드 서비스 활용:
     - Azure Kubernetes Service (AKS)를 기반으로 한 컨테이너 오케스트레이션
     - Azure API Management를 활용한 API Gateway 구현
     - @@@@@ Elastic Load Balancer
     - @@@@@ Auto Scaling
     - @@@@@ VPC
    
### 2. 데이터     
   - 서비스별 독립 데이터베이스 설계: Spring Boot H2 메모리 DB
   - JPA와 Hibernate를 활용한 ORM 구현

### 3. API
   - RESTful API 설계 원칙 준수
   - Spring Boot 기반 API 엔드포인트 정의

### 4. 보안
   - OAuth 2.0 기반 소셜 로그인 (Google, Naver) 구현

### 5. 확장성 고려
   - @@@@@ Azure Cache for Redis를 활용한 분산 캐싱 전략
   - @@@@@ Azure Service Bus를 이용한 비동기 메시징 시스템 설계

## IIII. Data Modeling/서비스 분리/설계(BIZ)

### 1. MSAEZ 모델링(Event Storming 결과)
http://www.msaez.io/#/storming/QtpQtDiH1Je3wad2QxZUJVvnLzO2/share/6f36e16efdf8c872da3855fedf7f3ea9

   **1.1 이벤트 도출**
<img src="https://github.com/KT-HOO/KTravel/blob/main/img/0910_1.png" width="500" height="400" />
    - 여행계획 생성 및 공유 서비스에 적합한 이벤트 도출

   **1.2 액터, 커맨드 부착**


   **1.3 어그리게잇으로 묶기**

    - Room, Reservation, Payment, Review 은 그와 연결된 command 와 event 들에 의하여 트랜잭션이 유지되어야 하는 단위로 그들 끼리 묶어줌

   2.5 바운디드 컨텍스트로 묶기

    - 도메인 서열 분리 
        - Core Domain:  reservation, room : 없어서는 안될 핵심 서비스이며, 연간 Up-time SLA 수준을 99.999% 목표, 배포주기는 reservation 의 경우 1주일 1회 미만, room 의 경우 1개월 1회 미만
        - Supporting Domain:   message, viewpage : 경쟁력을 내기위한 서비스이며, SLA 수준은 연간 60% 이상 uptime 목표, 배포주기는 각 팀의 자율이나 표준 스프린트 주기가 1주일 이므로 1주일 1회 이상을 기준으로 함.
        - General Domain:   payment : 결제서비스로 3rd Party 외부 서비스를 사용하는 것이 경쟁력이 높음 

   2.6 폴리시 부착 (괄호는 수행주체, 폴리시 부착을 둘째단계에서 해놔도 상관 없음. 전체 연계가 초기에 드러남)

   2.7 폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)

   2.8 완성된 1차 모형

![image](https://user-images.githubusercontent.com/15603058/119305002-0edd3000-bca3-11eb-9cc0-1ba8b17f2432.png)

    - View Model 추가

   2.9 1차 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증

![image](https://user-images.githubusercontent.com/15603058/119306321-f110ca80-bca4-11eb-804c-a965220bad61.png)

    - 호스트가 임대할 숙소를 등록/수정/삭제한다.(ok)
    - 고객이 숙소를 선택하여 예약한다.(ok)
    - 예약과 동시에 결제가 진행된다.(ok)
    - 예약이 되면 예약 내역(Message)이 전달된다.(?)
    - 고객이 예약을 취소할 수 있다.(ok)
    - 예약 사항이 취소될 경우 취소 내역(Message)이 전달된다.(?)
    - 숙소에 후기(review)를 남길 수 있다.(ok)
    - 전체적인 숙소에 대한 정보 및 예약 상태 등을 한 화면에서 확인 할 수 있다.(View-green Sticker 추가로 ok)
    
   2.10 모델 수정

    
    - 수정된 모델은 모든 요구사항을 커버함.

   2.11 비기능 요구사항에 대한 검증


- 마이크로 서비스를 넘나드는 시나리오에 대한 트랜잭션 처리
- 고객 예약시 결제처리:  결제가 완료되지 않은 예약은 절대 받지 않는다고 결정하여, ACID 트랜잭션 적용. 예약 완료시 사전에 방 상태를 확인하는 것과 결제처리에 대해서는 Request-Response 방식 처리
- 결제 완료시 Host 연결 및 예약처리:  reservation 에서 room 마이크로서비스로 예약요청이 전달되는 과정에 있어서 room 마이크로 서비스가 별도의 배포주기를 가지기 때문에 Eventual Consistency 방식으로 트랜잭션 처리함.
- 나머지 모든 inter-microservice 트랜잭션: 예약상태, 후기처리 등 모든 이벤트에 대해 데이터 일관성의 시점이 크리티컬하지 않은 모든 경우가 대부분이라 판단, Eventual Consistency 를 기본으로 채택함.

### 2. 주요 마이크로서비스 구조
     - 사용자(member) 서비스: 토큰(크레딧) 포함
     - 여행 계획(plan) 서비스: AI 추천 포함
     - 팔로우(follow) 서비스
     - 알림(notification) 서비스

### 3. 헥사고날 아키텍처 다이어그램 도출
<img src="https://github.com/KT-HOO/KTravel/blob/main/img/kafka%20%ED%99%94%EB%A9%B4.png" width="650" height="380" />

    - 각 서비스는 Kafka Publisher와 Kafka Listener를 통해 이벤트를 송신하고 수신 (느슨한 결합 및 확장성 확보)
    - 'Recommendation' 서비스는 Kafka 외에 REST 기반 통신을 사용하여 다른 시스템과 데이터를 주고 받음
    - 호출 관계에서 Pub/Sub과 Req/Resp 구분
    - JPA를 통해 각 서비스는 각각의 데이터베이스(H2)를 관리

## V. MSA개발 및 개발관리

### 1. 개발 환경 구축    
   - Spring Boot 개발 환경 설정
   - Vue.js 프로젝트 구조 설정 (Vue CLI 활용)
   - Docker를 이용한 컨테이너화 환경 구성
   - @@@@@ Azure DevOps를 활용한 CI/CD 파이프라인 구축

### 2. 백엔드 마이크로서비스 개발
이전 단계에서 도출된 OOOOO 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 Spring Boot로 구현하였다.    
구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
   mvn spring-boot:run
```  
   **2.1 CQRS**  
   - 예시 숙소(Room) 의 사용가능 여부, 리뷰 및 예약/결재 등 총 Status 에 대하여 고객(Customer)이 조회 할 수 있도록 CQRS 로 구현하였다.
     - 예시 room, review, reservation, payment 개별 Aggregate Status 를 통합 조회하여 성능 Issue 를 사전에 예방할 수 있다.
     - 예시 비동기식으로 처리되어 발행된 이벤트 기반 Kafka 를 통해 수신/처리 되어 별도 Table 에 관리한다
     - 예시 Table 모델링 (ROOMVIEW)

  ![image](https://user-images.githubusercontent.com/77129832/119319352-4b198c00-bcb5-11eb-93bc-ff0657feeb9f.png)
     - viewpage MSA ViewHandler 를 통해 구현 ("RoomRegistered" 이벤트 발생 시, Pub/Sub 기반으로 별도 Roomview 테이블에 저장)

   **2.2 API 게이트웨이**
      1. gateway 스프링부트 App을 추가 후 application.yaml내에 각 마이크로 서비스의 routes 를 추가하고 gateway 서버의 포트를 8080 으로 설정
       
          - application.yaml 예시
            ```
            spring:
              profiles: docker
              cloud:
                gateway:
                  routes:
                    - id: payment
                      uri: http://payment:8080
                      predicates:
                        - Path=/payments/** 
                    - id: room
                      uri: http://room:8080
                      predicates:
                        - Path=/rooms/**, /reviews/**, /check/**
                    - id: reservation

            ```

      2. Kubernetes용 Deployment.yaml 을 작성하고 Kubernetes에 Deploy를 생성함
          - Deployment.yaml 예시
          

            ```
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: gateway
              namespace: airbnb
              labels:
                app: gateway
            spec:
              replicas: 1
              selector:
                matchLabels:
                  app: gateway
            
![image](https://user-images.githubusercontent.com/80744273/119321943-1d821200-bcb8-11eb-98d7-bf8def9ebf80.png)
	    

### 3. 프론트엔드


## VI. 클라우드 배포

