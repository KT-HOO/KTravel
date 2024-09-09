<img src="https://github.com/KT-HOO/KTravel/blob/main/img/Ktravel.png" width="330" height="240"/>

# 여행계획 생성 및 공유 서비스(Kt-ravel)

Kt-ravel은 다양한 사용자들이 여행 계획을 생성하고 공유할 수 있는 MSA 기반의 플랫폼입니다.   
사용자는 각자 다른 계획을 만들고, 다른 사용자의 계획을 팔로우하며, 크레딧 시스템을 통해 활동할 수 있습니다.

@@@@@ 서비스 시나리오 더 구체적으로 작성 (연결관계 반영)

## 목차

**I. [서비스 시나리오](#서비스-시나리오)**    
 - 기능적 요구사항
 - 비기능적 요구사항

**II. [체크포인트](#체크포인트)**    
 - 분석/설계
 - 구현
 - 운영

**III. [분석/설계](#분석설계)**    
 - 아키텍처 설계
 - MSAEZ 모델링(Event Storming 결과)
 - OOOOO 아키텍처 다이어그램 도출

**IIII. [구현:](#구현-)**    
 - [DDD 의 적용](#ddd-의-적용)
 - [폴리글랏 퍼시스턴스](#폴리글랏-퍼시스턴스)
 - [폴리글랏 프로그래밍](#폴리글랏-프로그래밍)
 - [동기식 호출 과 Fallback 처리](#동기식-호출-과-Fallback-처리)
 - [비동기식 호출 과 Eventual Consistency](#비동기식-호출-과-Eventual-Consistency)

**V. [운영](#운영)**    
 - [CI/CD 설정](#cicd설정)
 - [동기식 호출 / 서킷 브레이킹 / 장애격리](#동기식-호출-서킷-브레이킹-장애격리)
 - [오토스케일 아웃](#오토스케일-아웃)
 - [무정지 재배포](#무정지-재배포)

**VI. [신규 개발 조직의 추가](#신규-개발-조직의-추가)**    

## I. 서비스 시나리오

### 기능적 요구사항(Functional Requirements)

**1. 회원 가입(Sign up) 및 토큰(Token) 관리**   
   **1.1 회원 가입 및 로그인**  
   - 사용자는 구글 또는 네이버 계정을 통해 회원가입 및 로그인이 가능해야 한다.

   **1.2 @@@@@사용자 프로필 관리** 
   - 사용자는 자신의 프로필 정보를 수정할 수 있어야 한다.

   **1.3 토큰(크레딧)**  
   - 사용자는 여행 계획 생성, 팔로우, 충전 등을 통해 토큰(크레딧)을 획득할 수 있어야 한다.
   - 토큰 획득 및 사용 기록이 @@@@@사용자 대시보드/프로필에서 조회 가능해야 한다.
   - 토큰은 여행 계획 추천 기능에 사용될 수 있어야 한다.

**2. 여행 계획(Plan) 생성, 수정, 삭제**    
   **2.1 여행 계획 생성**  
   - 사용자는 목적지(location), 날짜(travelDate), 예산(budget), 인원수(groupSize), 세부 활동(details) 등의 상세 내용을 포함한 여행 계획을 AI를 활용하여 생성할 수 있어야 한다.
   
   **2.2 여행 계획 수정**    
   - 사용자는 이미 생성한 여행 계획을 수정할 수 있어야 한다.

   **2.3 여행 계획 삭제**    
   - 사용자는 이미 생성한 여행 계획을 삭제할 수 있어야 한다.

   **2.4 @@@@@공개/비공개 설정 또는 여행 계획 공개**    
   - 사용자는 각 여행 계획에 대해 공개 여부를 설정할 수 있어야 하며, 공개된 계획은 다른 사용자와 공유된다.

**3. @@@@@추천(Recommendation) 시스템**    
   **3.1 사용자 맞춤형 추천**  
   - 사용자의 여행 취향(예: 방문한 지역, 좋아하는 활동 등)을 분석하여 맞춤형 여행 계획을 추천한다.
   
   **3.2 트렌드 기반 추천**  
   - 현재 플랫폼에서 인기 있는 여행 계획이나 활동을 추천하는 기능을 포함한다.
   
   **3.3 사용자 활동 분석**  
   - 사용자들이 팔로우한 계획이나 조회 기록을 분석하여 관련 여행 계획을 추천한다.

**4. @@@@@ 계획을 팔로우, 사용자를 팔로우인지?? 다른 사용자의 계획 팔로우/언팔로우(Follow/Unfollow)**    
   **4.1 팔로우 기능**  
   - 사용자는 다른 사용자가 만든 여행 계획을 팔로우할 수 있다.
   
   **4.2 언팔로우 기능**  
   - 더 이상 관심이 없는 사용자의 계획을 언팔로우할 수 있다.
   
   **4.3 @@@@@ 따로 대시보드가 있어야 할듯 팔로우 계획 모니터링**  
   - 팔로우한 계획은 별도 대시보드에서 확인할 수 있어야 한다.

**5. 알림(Notification) 기능**    
   **5.1 계획 생성 알림**  
   - 사용자가 계획을 생성할 경우, 사용자에게 시스템에서 알림을 발송한다.
   
   **5.2 팔로우 알림**  
   - 사용자가 다른 사용자를 팔로우할 경우, 다른 사용자에게 시스템에서 알림을 발송한다.



### 비기능적 요구사항 (Non-functional Requirements)

**1. 트랜잭션 일관성 유지**    

   **1.1 데이터 일관성**  
   - 사용자 토큰 사용, 여행 계획 생성/수정/삭제 등의 트랜잭션이 중단되지 않고 일관성을 유지해야 한다.
   - 분산 트랜잭션 처리 메커니즘을 통해 이를 보장한다.
   
   **1.2 ACID 속성 보장**  
   - 중요한 데이터(회원 정보, 여행 계획, 토큰 내역 등)에 대해 ACID 속성을 보장하는 데이터베이스 구조를 구현한다.

**2. 서비스 간 느슨한 결합**    

   **2.1 MSA**  
   - 각 기능이 독립된 마이크로서비스로 운영되며, 서로 간의 결합도를 최소화하여 유지보수 및 확장성을 높여야 한다.
   
   **2.2 API 기반 통신**  
   - 마이크로서비스 간의 통신은 RESTful API 또는 메시지 큐 기반으로 이루어져야 하며, 서비스의 장애나 업데이트가 다른 서비스에 영향을 미치지 않도록 설계해야 한다.

**3. 장애 격리**    

   **3.1 장애 감지 및 복구**  
   - 특정 서비스가 실패하더라도 전체 시스템이 영향을 받지 않도록 장애 격리 메커니즘을 구현해야 한다.
   - 개별 서비스 장애 시 자동 복구 및 롤백이 가능해야 한다.
   
   **3.2 모니터링 및 로그 관리**  
   - 실시간 모니터링을 통해 장애를 감지하고, 이를 분석하여 빠르게 대응할 수 있는 로그 관리 시스템을 구현해야 한다.

**4. 확장성 및 유연성**    

   **4.1 자동 확장**  
   - 사용자 수 증가에 따라 자동으로 서버 리소스를 확장할 수 있는 기능을 갖추어야 한다.
   - 이를 위해 컨테이너화된 서비스나 클라우드 인프라를 사용할 수 있다.
   
   **4.2 유연한 데이터베이스 확장**  
   - 데이터베이스는 수평적 확장을 지원하며, 많은 양의 사용자 데이터를 효율적으로 처리할 수 있어야 한다.
   
   **4.3 플랫폼 확장**  
   - 새로운 기능이 추가되더라도 기존 시스템에 영향을 주지 않고 손쉽게 확장할 수 있는 유연한 아키텍처를 적용해야 한다.

**5. 보안 및 개인정보 보호**    

   **5.1 데이터 암호화**  
   - 사용자 정보 및 민감한 데이터는 저장 및 전송 시 모두 암호화해야 한다.
   
   **5.2 접근 제어**  
   - 각 서비스에 대한 접근 권한을 엄격히 관리하며, 사용자 인증 및 권한 부여를 통해 보호해야 한다.
   
   **5.3 개인정보 보호**  
   - 사용자의 여행 계획과 같은 개인정보는 외부에 노출되지 않도록 정책을 설정하고 준수해야 한다.


## II. 체크포인트

###  분석 설계

  - 이벤트스토밍: 
    - 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
    - 각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
    - 어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
    - 기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?    

  - 서브 도메인, 바운디드 컨텍스트 분리
    - 팀별 KPI 와 관심사, 상이한 배포주기 등에 따른  Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
      - 적어도 3개 이상 서비스 분리
    - 폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
    - 서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?
  - 컨텍스트 매핑 / 이벤트 드리븐 아키텍처 
    - 업무 중요성과  도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
    - Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
    - 장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
    - 신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
    - 이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?

  - 헥사고날 아키텍처
    - 설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?
    
###  구현
  - [DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?
    - Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
    - [헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를 적응시킬 수 있는가?
    - 분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?
  - Request-Response 방식의 서비스 중심 아키텍처 구현
    - 마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
    - 서킷브레이커를 통하여  장애를 격리시킬 수 있는가?
  - 이벤트 드리븐 아키텍처의 구현
    - 카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
    - Correlation-key:  각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
    - Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
    - Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
    - CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가 가능한가?

  - 폴리글랏 플로그래밍
    - 각 마이크로 서비스들이 하나이상의 각자의 기술 Stack 으로 구성되었는가?
    - 각 마이크로 서비스들이 각자의 저장소 구조를 자율적으로 채택하고 각자의 저장소 유형 (RDB, NoSQL, File System 등)을 선택하여 구현하였는가?
  - API 게이트웨이
    - API GW를 통하여 마이크로 서비스들의 집입점을 통일할 수 있는가?
    - 게이트웨이와 인증서버(OAuth), JWT 토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?
###  운영
  - SLA 준수
    - 셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
    - 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
    - 오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
    - 모니터링, 앨럿팅: 
  - 무정지 운영 CI/CD (10)
    - Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명 
    - Contract Test :  자동화된 경계 테스트를 통하여 구현 오류나 API 계약위반를 미리 차단 가능한가?


## III. 분석/설계

### 1. 아키텍처 설계     

   **1.1 아키텍처**  
   - MSA 기반 설계:
     - Spring Boot 마이크로서비스
     - @@@@@ Vue.js 프론트엔드
   - Azure 클라우드 서비스 활용:
     - Azure Kubernetes Service (AKS)를 기반으로 한 컨테이너 오케스트레이션
     - Azure API Management를 활용한 API Gateway 구현
     - @@@@@ Azure Service Fabric을 이용한 서비스 디스커버리
   - 주요 마이크로서비스 구조:
     - 사용자(member) 서비스: 토큰(크레딧) 포함
     - 여행 계획(plan) 서비스: AI 추천 포함
     - 팔로우(follow) 서비스
     - 알림(notification) 서비스
  
   **1.2 데이터**  
   - 서비스별 독립 데이터베이스 설계:
     - @@@@@ Azure Database for PostgreSQL: 관계형 데이터 저장
     - @@@@@ Azure Cosmos DB: 비관계형 데이터 저장
   - JPA와 Hibernate를 활용한 ORM 구현
   - 분산 트랜잭션을 위한 Saga 패턴 설계

   **1.3 API**
   - RESTful API 설계 원칙 준수
   - Spring Boot 기반 API 엔드포인트 정의
   - @@@@@ Swagger를 활용한 API 문서화
   - @@@@@ API 버전 관리 전략 수립

   **1.4 보안**
   - Spring Security와 JWT를 이용한 인증/인가 처리
   - @@@@@ Azure Key Vault를 활용한 비밀 정보 관리
   - OAuth 2.0 기반 소셜 로그인 (Google, Naver) 구현
   - HTTPS 적용을 통한 전송 계층 보안

   **1.5 확장성 고려**
   - 수평적 확장이 가능한 Stateless 아키텍처 설계
   - @@@@@ Azure Cache for Redis를 활용한 분산 캐싱 전략
   - @@@@@ Azure Service Bus를 이용한 비동기 메시징 시스템 설계


### 2. MSAEZ 모델링(Event Storming 결과)
http://www.msaez.io/#/storming/QtpQtDiH1Je3wad2QxZUJVvnLzO2/share/6f36e16efdf8c872da3855fedf7f3ea9


   **2.1 이벤트 도출**

   **2.2 부적격 이벤트 탈락**

    - 과정중 도출된 잘못된 도메인 이벤트들을 걸러내는 작업을 수행함
        - 등록시>RoomSearched, 예약시>RoomSelected :  UI 의 이벤트이지, 업무적인 의미의 이벤트가 아니라서 제외

   2.3 액터, 커맨드 부착하여 읽기 좋게

   2.4 어그리게잇으로 묶기

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


### 3. OOOOO 아키텍처 다이어그램 도출

![image](https://user-images.githubusercontent.com/80744273/119319091-fc6bf200-bcb4-11eb-9dac-0995c84a82e0.png)


    - Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
    - 호출관계에서 PubSub 과 Req/Resp 를 구분함
    - 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐


## IIII. 구현

**1. 개발 환경 구축**    
   - Spring Boot 개발 환경 설정
   - Vue.js 프로젝트 구조 설정 (Vue CLI 활용)
   - Docker를 이용한 컨테이너화 환경 구성
   - @@@@@ Azure DevOps를 활용한 CI/CD 파이프라인 구축

**2. 백엔드 마이크로서비스 개발**
분석/설계 단계에서 도출된 OOOOO 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 Spring Boot로 구현하였다.    
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
                      uri: http://reservation:8080
                      predicates:
                        - Path=/reservations/**
                    - id: message
                      uri: http://message:8080
                      predicates:
                        - Path=/messages/** 
                    - id: viewpage
                      uri: http://viewpage:8080
                      predicates:
                        - Path= /roomviews/**
                  globalcors:
                    corsConfigurations:
                      '[/**]':
                        allowedOrigins:
                          - "*"
                        allowedMethods:
                          - "*"
                        allowedHeaders:
                          - "*"
                        allowCredentials: true

            server:
              port: 8080            
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
              template:
                metadata:
                  labels:
                    app: gateway
                spec:
                  containers:
                    - name: gateway
                      image: 247785678011.dkr.ecr.us-east-2.amazonaws.com/gateway:1.0
                      ports:
                        - containerPort: 8080
            ```               
            

            ```
            Deploy 생성
            kubectl apply -f deployment.yaml
            ```     
          - Kubernetes에 생성된 Deploy. 확인
            
![image](https://user-images.githubusercontent.com/80744273/119321943-1d821200-bcb8-11eb-98d7-bf8def9ebf80.png)
	    
   **2.3 API 동기식 호출(Sync) 과 Fallback 처리**
      1. 분석 단계에서의 조건 중 하나로 예약 시 숙소(room) 간의 예약 가능 상태 확인 호출은 동기식 일관성을 유지하는 트랜잭션으로 처리하기로 하였다. 호출 프로토콜은 이미 앞서 Rest Repository 에 의해 노출되어있는 REST 서비스를 FeignClient 를 이용하여 호출하도록 한다. 또한 예약(reservation) -> 결제(payment) 서비스도 동기식으로 처리하기로 하였다.
      - 룸, 결제 서비스를 호출하기 위하여 Stub과 (FeignClient) 를 이용하여 Service 대행 인터페이스 (Proxy) 를 구현 

   **2.3 API 동기식 호출(Sync) 과 Fallback 처리**
   **2.4 비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트**

## V. 운영


### 1. CI/CD 설정

각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD는 buildspec.yml을 이용한 AWS codebuild를 사용하였습니다.

- CodeBuild 프로젝트를 생성하고 AWS_ACCOUNT_ID, KUBE_URL, KUBE_TOKEN 환경 변수 세팅을 한다
```
SA 생성
kubectl apply -f eks-admin-service-account.yml
```
![codebuild(sa)](https://user-images.githubusercontent.com/38099203/119293259-ff52ec80-bc8c-11eb-8671-b9a226811762.PNG)
```

### 2. 동기식 호출 / 서킷 브레이킹 / 장애격리

* 서킷 브레이킹 프레임워크의 선택: istio 사용하여 구현함

시나리오는 예약(reservation)--> 룸(room) 시의 연결을 RESTful Request/Response 로 연동하여 구현이 되어있고, 예약 요청이 과도할 경우 CB 를 통하여 장애격리.

- DestinationRule 를 생성하여 circuit break 가 발생할 수 있도록 설정
최소 connection pool 설정
```
#### destination-rule.yml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: dr-room
  namespace: airbnb
spec:
  host: room
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
#    outlierDetection:
#      interval: 1s
#      consecutiveErrors: 1
#      baseEjectionTime: 10s
#      maxEjectionPercent: 100
```

* istio-injection 활성화 및 room pod container 확인

```
kubectl get ns -L istio-injection
kubectl label namespace airbnb istio-injection=enabled 
```

![Circuit Breaker(istio-enjection)](https://user-images.githubusercontent.com/38099203/119295450-d6812600-bc91-11eb-8aad-46eeac968a41.PNG)

![Circuit Breaker(pod)](https://user-images.githubusercontent.com/38099203/119295568-0cbea580-bc92-11eb-9d2b-8580f3576b47.PNG)


* 부하테스터 siege 툴을 통한 서킷 브레이커 동작 확인:

siege 실행

```
kubectl run siege --image=apexacme/siege-nginx -n airbnb
kubectl exec -it siege -c siege -n airbnb -- /bin/bash
```


- 동시사용자 1로 부하 생성 시 모두 정상
```
siege -c1 -t10S -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'

** SIEGE 4.0.4
** Preparing 1 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.49 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.05 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     256 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     256 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     256 bytes ==> POST http://room:8080/rooms
```

- 동시사용자 2로 부하 생성 시 503 에러 168개 발생
```
siege -c2 -t10S -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'

** SIEGE 4.0.4
** Preparing 2 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.10 secs:      81 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.04 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.05 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.22 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.08 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.07 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.01 secs:      81 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.01 secs:      81 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.00 secs:      81 bytes ==> POST http://room:8080/rooms

Lifting the server siege...
Transactions:                   1904 hits
Availability:                  91.89 %
Elapsed time:                   9.89 secs
Data transferred:               0.48 MB
Response time:                  0.01 secs
Transaction rate:             192.52 trans/sec
Throughput:                     0.05 MB/sec
Concurrency:                    1.98
Successful transactions:        1904
Failed transactions:             168
Longest transaction:            0.03
Shortest transaction:           0.00
```

- kiali 화면에 서킷 브레이크 확인

![Circuit Breaker(kiali)](https://user-images.githubusercontent.com/38099203/119298194-7f7e4f80-bc97-11eb-8447-678eece29e5c.PNG)


- 다시 최소 Connection pool로 부하 다시 정상 확인

```
** SIEGE 4.0.4
** Preparing 1 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms

:
:

Lifting the server siege...
Transactions:                   1139 hits
Availability:                 100.00 %
Elapsed time:                   9.19 secs
Data transferred:               0.28 MB
Response time:                  0.01 secs
Transaction rate:             123.94 trans/sec
Throughput:                     0.03 MB/sec
Concurrency:                    0.98
Successful transactions:        1139
Failed transactions:               0
Longest transaction:            0.04
Shortest transaction:           0.00

```

- 운영시스템은 죽지 않고 지속적으로 CB 에 의하여 적절히 회로가 열림과 닫힘이 벌어지면서 자원을 보호하고 있음을 보여줌.
  virtualhost 설정과 동적 Scale out (replica의 자동적 추가,HPA) 을 통하여 시스템을 확장 해주는 후속처리가 필요.


### 오토스케일 아웃
앞서 CB 는 시스템을 안정되게 운영할 수 있게 해줬지만 사용자의 요청을 100% 받아들여주지 못했기 때문에 이에 대한 보완책으로 자동화된 확장 기능을 적용하고자 한다. 

- room deployment.yml 파일에 resources 설정을 추가한다
![Autoscale (HPA)](https://user-images.githubusercontent.com/38099203/119283787-0a038680-bc79-11eb-8d9b-d8aed8847fef.PNG)

- room 서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 50프로를 넘어서면 replica 를 10개까지 늘려준다:
```
kubectl autoscale deployment room -n airbnb --cpu-percent=50 --min=1 --max=10
```
![Autoscale (HPA)(kubectl autoscale 명령어)](https://user-images.githubusercontent.com/38099203/119299474-ec92e480-bc99-11eb-9bc3-8c5246b02783.PNG)

- 부하를 동시사용자 100명, 1분 동안 걸어준다.
```
siege -c100 -t60S -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'
```
- 오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다
```
kubectl get deploy room -w -n airbnb 
```
- 어느정도 시간이 흐른 후 (약 30초) 스케일 아웃이 벌어지는 것을 확인할 수 있다:
![Autoscale (HPA)(모니터링)](https://user-images.githubusercontent.com/38099203/119299704-6a56f000-bc9a-11eb-9ba8-55e5978f3739.PNG)

- siege 의 로그를 보아도 전체적인 성공률이 높아진 것을 확인 할 수 있다. 
```
Lifting the server siege...
Transactions:                  15615 hits
Availability:                 100.00 %
Elapsed time:                  59.44 secs
Data transferred:               3.90 MB
Response time:                  0.32 secs
Transaction rate:             262.70 trans/sec
Throughput:                     0.07 MB/sec
Concurrency:                   85.04
Successful transactions:       15675
Failed transactions:               0
Longest transaction:            2.55
Shortest transaction:           0.01
```

## 무정지 재배포

* 먼저 무정지 재배포가 100% 되는 것인지 확인하기 위해서 Autoscaler 이나 CB 설정을 제거함

```
kubectl delete destinationrules dr-room -n airbnb
kubectl label namespace airbnb istio-injection-
kubectl delete hpa room -n airbnb
```

- seige 로 배포작업 직전에 워크로드를 모니터링 함.
```
siege -c100 -t60S -r10 -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'

** SIEGE 4.0.4
** Preparing 1 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms

```

- 새버전으로의 배포 시작
```
kubectl set image ...
```

- seige 의 화면으로 넘어가서 Availability 가 100% 미만으로 떨어졌는지 확인

```
siege -c100 -t60S -r10 -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'


Transactions:                   7732 hits
Availability:                  87.32 %
Elapsed time:                  17.12 secs
Data transferred:               1.93 MB
Response time:                  0.18 secs
Transaction rate:             451.64 trans/sec
Throughput:                     0.11 MB/sec
Concurrency:                   81.21
Successful transactions:        7732
Failed transactions:            1123
Longest transaction:            0.94
Shortest transaction:           0.00

```
- 배포기간중 Availability 가 평소 100%에서 87% 대로 떨어지는 것을 확인. 원인은 쿠버네티스가 성급하게 새로 올려진 서비스를 READY 상태로 인식하여 서비스 유입을 진행한 것이기 때문. 이를 막기위해 Readiness Probe 를 설정함

```
# deployment.yaml 의 readiness probe 의 설정:
```

![probe설정](https://user-images.githubusercontent.com/38099203/119301424-71333200-bc9d-11eb-9f75-f8c98fce70a3.PNG)

```
kubectl apply -f kubernetes/deployment.yml
```

- 동일한 시나리오로 재배포 한 후 Availability 확인:
```
Lifting the server siege...
Transactions:                  27657 hits
Availability:                 100.00 %
Elapsed time:                  59.41 secs
Data transferred:               6.91 MB
Response time:                  0.21 secs
Transaction rate:             465.53 trans/sec
Throughput:                     0.12 MB/sec
Concurrency:                   99.60
Successful transactions:       27657
Failed transactions:               0
Longest transaction:            1.20
Shortest transaction:           0.00

```

배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.


# Self-healing (Liveness Probe)
- room deployment.yml 파일 수정 
```
콘테이너 실행 후 /tmp/healthy 파일을 만들고 
90초 후 삭제
livenessProbe에 'cat /tmp/healthy'으로 검증하도록 함
```
![deployment yml tmp healthy](https://user-images.githubusercontent.com/38099203/119318677-8ff0f300-bcb4-11eb-950a-e3c15feed325.PNG)

- kubectl describe pod room -n airbnb 실행으로 확인
```
컨테이너 실행 후 90초 동인은 정상이나 이후 /tmp/healthy 파일이 삭제되어 livenessProbe에서 실패를 리턴하게 됨
pod 정상 상태 일때 pod 진입하여 /tmp/healthy 파일 생성해주면 정상 상태 유지됨
```

![get pod tmp healthy](https://user-images.githubusercontent.com/38099203/119318781-a9923a80-bcb4-11eb-9783-65051ec0d6e8.PNG)
![touch tmp healthy](https://user-images.githubusercontent.com/38099203/119319050-f118c680-bcb4-11eb-8bca-aa135c1e067e.PNG)

