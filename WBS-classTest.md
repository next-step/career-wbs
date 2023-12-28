
# 🪴 career-WBS
> mermaid로 작성된 과제는 마크다운 파일(WBS.md)로 올려주시면 됩니다. (md 파일 내에 기존 구조를 넣어주세요) <br>
> 별도 아키택쳐나 모델링 도구를 사용한 경우에는 마크다운 파일(WBS.md)과 png, gif, jpg, pdf 파일 형식으로 WBS-{gitID}.png 파일명으로 upload 해주세요
# 요구사항
- [ ] 개선하려는 프로젝트의 최종 설계
    - [ ] 변경 사항에 대한 Target 시스템 설계를 확정한다. (2주차 미션 활용)
    - [ ] 변경 사항에 대한 기대효과를 확정한다. (2주차 미션 활용)
- [ ] task list 도출
    - [ ] 현 시스템에서 변경되는 부분을 class diagram(DB변경이 발생할 경우 ERD추가)으로 작성
    - [ ] 변경, 추가 될 프로그램들의 작업 목록을 작성한다.
- [ ] 일정 계획 문서 (WBS)
  - [ ] 작업목록의 소요일정을 산정 한다.
  - [ ] 작업 목록의 의존성을 정의 한다.
  - [ ] 작업 목록의 전체 일정을 작성한다.
  - [ ] 진행 상태를 check하기위한 마일스톤 설정 한다.


# 🚀미션
## AS-IS
### AS-IS 개선포인트 분석
#### 1.배포 개선
- __현재 배포 과정__:\
  현재의 배포 과정은 수동으로 진행되며, 이에 따라 서버 간 파일 이동, 서버 접속, 각종 스크립트 실행 등 복잡한 단계를 거쳐야 합니다.
- __통신 및 알림의 부족__:\
  배포가 완료된 후, 지라나 슬랙을 통해 팀원들에게 별도로 알려야 하는 번거로움이 있습니다.
- __배포 코드의 불확실성__:\
  긴급한 요청으로 인해 때때로 feature 브랜치의 코드가 dev 서버에 배포되는 경우가 있어, 어떤 코드가 실제로 서버에 배포되었는지 확인하기 어렵습니다.
#### 2. 배치 개선
- __과중한 단일 Job__:\
  현재 배치 Job이 너무 많은 기능을 수행하고 있으며, 다양한 타입의 알림을 한 번에 처리하고 있습니다. 이는 작성한 사람만 아는 코드가 되버립니다.
- __Job의 작동시간__:\
  하나의 Job이 과도하게 크고 복잡할 경우, 실행 시간이 길어지며 이는 고객에게 알림을 전달하는 시간의 지연을 초래할 수 있습니다.\
  중간에 실패한 타입 때문에 전체 Job이 재실행되는 구조는 효율성을 크게 저하시키며, 이는 시스템 자원의 낭비와 서비스 지연을 초래할 수 있습니다.
- __테스트 환경의 불안정성__:\
  배치 테스트는 개발(dev) 환경의 Elasticsearch 서버에 직접 요청하여 진행되고 있습니다. 이로 인해 테스트 데이터의 유무에 따라 테스트 결과가 변동되는 문제가 있습니다.


### AS-IS 프로세스

#### 1. 배포 개선
```mermaid
flowchart TD
  A[기능 개발 완료] -->B(JAR 빌드)
  B --> C(SCP로 dev 서버로 전달)
  C --> D(SSH로 접속해서 배포 스크립트 실행)
  D --> E(정상 배포되면 프런트 팀에 내용 전달)
  D --> |잘못 배포한 경우 재배포| A
  E --> |미스 커뮤니케이션 | A
  E --> F(마스터 브랜치에 풀리퀘스트)
```
#### 2. 배치 개선

```mermaid
flowchart TD
    A[잰킨스에서 30분 마다 실행] -->B{타입이 사용량인지}
    B --> C(타입 SP/RI 데이터 처리)
    B --> |yes| F{알림에 설정된 기간?}
    C --> F
    F --> |월별| Q(일별 알림 로직)
    F --> |일별| W(월별 알림 로직)
    Q --> G(알림 DB 저장)
    W --> G
    G --> T(이메일 전송 Job 실행)
    
```

### Class diagram
- AS-IS 구조에서 개선을 할때 영향을 받게되는 class diagram을 작성한다.
#### 2. 배치 개선
```mermaid
classDiagram 
    class expiration {
        +_int4 notifiedtype
        +timestamp endtime
        +String linkedaccount
        +String reservedinstanceid
    }

    class send_standard {
        +String sendType
        +boolean useyn
    }
    
    class usage_notification {
        +int sendamount
        +String sumtype
        +Long send_standard_id
    }
    usage_notification -- send_standard
    
    class email_address {
        +int useridx
        +String email
        +int languageid
        +boolean isSubscripion
    }
    
    class email_group {
        +String name
        +_text emailidlist
        +int useridx
    }
    email_address -- email_group
    
    class tag_standard {
        +long id
        +String tagname
    }
    
    class email_mapping {
        +long emailid
        +int useridx
        +int tagid 
        +int tableid
    }
    email_mapping -- email_address
    email_mapping -- tag_standard
    email_mapping <|-- usage_notification
    email_mapping <|-- expiration
    
    class expiration_target {
      +_String emails
      +_long tableids
      +_String expirationids
      +_String linkedaccount
      +_String notifiedtypes
    }
    expiration_target "*"--"1" email_mapping

  class hyper_batch_execution {
    +int executiontagid
    +text sendcontent
    +timestamp billingdt
    +timestamp senddt
  }
  hyper_batch_execution "*"--"1" expiration_target
  hyper_batch_execution "1"--"1" email_mapping : 사용량 알림
  
```


### ERD
-AS-IS 구조에서 개선을 할때 영향을 받게되는 ERD를 작성한다.

```mermaid
erDiagram
  hyper_batch_execution {
    Long id 
    Integer executiontagid
    Long tableid
    Integer useridx
    boolean active
    timestamp billingdt
    String sendtype
    text sendContent
  }

   expiration_target {
    _String emails
    _Long tableids
    _String expirationids
    _String linkedaccount
    _String notifiedtypes
}

```

## TO-BE 
### TO-BE 기대효과 분석
- timeout 건을 처리하는데 매일 소요되는 2시간의 업무 시간을 30분 내외로 줄일 수 있다.
- 사람이 직접 하는 부분을 자동화 하여 실수를 줄일 수 있다.
    - 가끔 결제가 되었는데 timeout건으로 나왔으나 수기처리시 누락된 경우 고객의 CS 클래임이 인입되고 좋지않은 고객경험을 준다.
    - timeout 갤제 CS인입건 1건/week 을 0건으로 줄일 수 있다.
- 익일 처리되던 프로세스를 5분단위의 batch로 처리하여서 고객만족을 줄 수 있다.
    - 주문은 실패 했지만 결제가 되었다는 CS 건 3건/week를 0건으로 줄일 수 있다.

#### 1.배포 개선
- __개선 배포 과정__:\
CodeCommit의 stage 브랜치이 업데이트되면 AWS의 코드 비륻와 코드 디플로이를 통해 자동 배포되도록 한다. 
- __알림 기능 추가__:\
  배포가 완료된 후 slack의 어떤 커밋이 올라갔는지 알림이 보내진다

#### 2. 배치 개선
- __개별 Job__:\
  타입별 Job을 나눠준다. 그리고 각기 Job들이 한가지 알림을 처리하도록 한다.
- __테스트 개선__:\
  배치 테스트에서 외부 서버를 Mocking해서 테스트가 다른 서버에 의존하지 않도록 한다.
- __Job 상태 저장__:\
  특정 잡이 실패했을 때, 현재는 batch_job_execution에서 실패 이유를 추적해야하였는데,\
  이를 개선하기 위해서 특정 잡의 실행 상태와 이유를 저장한다.  


### TO-BE 프로세스

#### 1. 배포 개선
```mermaid
flowchart TD
  A[기능 개발 완료] -->B(코드커밋 특정 브런치 merged)
  B --> AWS_파이프라인
  subgraph AWS_파이프라인
    C(코드 빌드) --> D(코드 디플로이)
    D -->|배포1| E(dev EC2)
    D -->|배포2| F(dev EC2)
    E --> G(배포 완료)
    F --> G
    G --> H(Slack에 배포 내용 공유)
  end
```
#### 2. 배치 개선

```mermaid
flowchart TD
    A[잰킨스에서 30분 마다 실행] -->B{타입이 사용량인지}
    B --> C(타입 SP/RI 데이터 처리)
    B --> |yes| F{알림에 설정된 기간?}
    C --> F
    F --> |월별| Q(일별 알림 로직)
    F --> |일별| W(월별 알림 로직)
    Q --> G(알림 DB 저장)
    W --> G
    G --> T(이메일 전송 Job 실행)
    
```

### class diagram
- class diagram
```mermaid
classDiagram

    class PaymentMethod {
        +String paymentMethodID
        pay()
        cancel()
    }
    PaymentMethod <|-- Card
    PaymentMethod <|-- Bank

    class PG {
        +String pgID
        pay()
        cancel()
    }
    PG <|-- Card
    PG <|-- Bank


    class Payment {
        +String paymentID
        +String transactionID
        void pay()
    }

    class Cancel {
        +String cancelID
        +PaymentID paymentID
        +String transactionID
        void cancel()
    }

    class CancelDetail {
        +String cancelDetailID
        +String cancelID
    }

    class PaymentDetail {
        +PaymentID paymentID
    }


    class Card {
        CardID
        pay()
        cancel()
        checkTransaction()
    }
    note for Card "checkTransaction() : 결제내역확인"

    class Bank {
        BankID
        pay()
        cancel()
        checkTransaction()
    }
    note for Bank "checkTransaction() : 결제내역확인"

   Payment "1" -- "*" PaymentDetail : 결제수단, 금액, 상품 정보
   Cancel "1" -- "*" CancelDetail : 결제수단, 금액, 상품 정보
   Cancel "0..1" --> "1" Payment : 원결제 정보
   Payment --> PaymentMethod : 결제요청
   Cancel --> PaymentMethod : 취소요청


   class PaymentTiemoutListner {
        +beforeCancelForTimeout()
        -checkLimitRetryCount()
        -isPay()
        +cancelForTimeout()
        +postCancelForTimeout()
   }

   class timeoutResultNotification {
        sendNotification()
   }


    PaymentTiemoutListner "1" -- "1" Payment : 원결제확인
    Payment --> PaymentTiemoutListner : Timeout Event
    PaymentTiemoutListner --> PaymentMethod : cancel 처리

```
    

### ERD
- TO-BE 구조에서 변경되는 ERD를 작성한다.
```mermaid
erDiagram
  Payment {
    Integer id 
    String name
  }
  Payment ||--|{ PaymentDetail : has

  PaymentMethod {
    Integer id
    String name 
  }

  PaymentDetail {
    Integer id
    Integer paymentId
    Integer paymentMethodId
    Long productId
    Integer amount
    Integer quantity
    Integer unitPrice
    String productInfo
  }
  PaymentDetail ||--|{ PaymentMethod : fundingsource

  Cancel {
    Integer id
    Integer paymentId
    String transactionId
  }
  Cancel ||--|{ CancelDetail : has
  CancelDetail ||--|{ PaymentMethod : fundingsource

  CancelDetail {
    Integer id
    Integer cancelId
    Integer amount
    String productInfo
  }

  PaymentDetail {
    String paymentId
    String paymentMethodId
  }

  CancelDetail {
    String cancelId
  }

  payTimeoutRetry {
    String id
    Integer retryCnt
    String status
  }

  payTimeoutRetryHistories {
    String id
    String status
  }

payTimeoutRetry ||--o{ Retry-Process : do
payTimeoutRetryHistories ||--o{ Retry-Process : dohistories

```

## Task List
1. Timeout 발생 시 Event발생 수정- SQS, SNS <br>
2. Timeout event subscription module 작성<br>
3. Timeout log table 설계, 생성<br>
4. Timeout 재처리 service 설개, 구현<br>
&nbsp; &nbsp; 1. transaction 성공여부 확인 <br>
&nbsp; &nbsp; 2. transaction 취소 처리 하기 (결제시)<br>
&nbsp; &nbsp; 3. 재처리 logging(DB) : 처리 횟수(3회), 처리 내역<br>
5. Timeout 재처리 현황 조회 어드민 page.<br>
6. Timeout 재처리 실패시 메일 발송 모듈.<br>


## WBS

- 산정 기준 : 4시간/일

1. 요구사항 분석 : 이미수행
2. 설계 : 3d
3. 일정산정: 1d
4. Timeout 발생 시 Event발생 수정- SQS, SNS : 이미 사용하는 SQS가 있고 큐생성 및 기존코드 수정 : 2d
5. Timeout event subscription module 작성 : SQS, SNS : 이미 사용하는 SQS가 있고 신규 class 생성 : 2d
6. Timeout log table 설계, 생성 : 1d
7. Timeout 재처리 service 설개, 구현 : 2d
    1. transaction 성공여부 확인 : 0.5d
    2. transaction 취소 처리 하기 (결제시) : 0.5d
    3. 재처리 logging(DB) : 처리 횟수(3회), 처리 내역 : 1d
8. Timeout 재처리 현황 조회 어드민 page.: 기존 admin에 메뉴 추가 : 5d
9. Timeout 재처리 실패시 메일 발송 모듈: 기존 notification에 method 추가 : 1d

```mermaid
gantt
    dateFormat  YYYY-MM-DD
    title       결제 재처리 WBS
    excludes    weekends, 2023-12-25, 2024-01-01
    %% (`excludes` accepts specific dates in YYYY-MM-DD format, days of the week ("sunday") or "weekends", but not the word "weekdays".)

    section prepare
    요구사항분석                    :done,    des1, 2023-12-01, 10d
    설계                            :active,  des2, 2023-12-11, 3d
    일정산정                        :         des3, after des2, 1d
    Timeout log table 설계, 생성    :       des4, 2023-12-27, 1d

    section 기존 모듈 수정
    Payment timeout event 발생          :crit, b1, 2024-01-03,2d
    Cancel timeout용 cancel 추가        :crit, b2, 2024-01-10, 2d

    section 신규 모듈 구현
    Timeout event consumer 모듈작성    :c1, after b1, 2d
    Queue 동작확인                      :milestone, after c1, 0d
    Timeout service 구현                  :c2, after b2  , 2d
    Timeout 재처리 현황 조회 어드민 개발    :c3, after c2  , 5d
    Timeout 재처리 실패시 notification     : c4, after c3, 1d

    section 테스트
    Test & QA                           :after c4, 2d

```

