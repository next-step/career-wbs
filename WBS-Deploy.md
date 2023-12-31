
# 🪴 career-WBS
> mermaid로 작성된 과제는 마크다운 파일(WBS.md)로 올려주시면 됩니다. (md 파일 내에 기존 구조를 넣어주세요) <br>
> 별도 아키택쳐나 모델링 도구를 사용한 경우에는 마크다운 파일(WBS.md)과 png, gif, jpg, pdf 파일 형식으로 WBS-{gitID}.png 파일명으로 upload 해주세요
# 요구사항
- [x] 개선하려는 프로젝트의 최종 설계
    - [x] 변경 사항에 대한 Target 시스템 설계를 확정한다. (2주차 미션 활용)
    - [x] 변경 사항에 대한 기대효과를 확정한다. (2주차 미션 활용)
- [x] task list 도출
    - [x] 현 시스템에서 변경되는 부분을 class diagram(DB변경이 발생할 경우 ERD추가)으로 작성
    - [x] 변경, 추가 될 프로그램들의 작업 목록을 작성한다.
- [ ] 일정 계획 문서 (WBS)
  - [ ] 작업목록의 소요일정을 산정 한다.
  - [ ] 작업 목록의 의존성을 정의 한다.
  - [ ] 작업 목록의 전체 일정을 작성한다.
  - [ ] 진행 상태를 check하기위한 마일스톤 설정 한다.


# 🚀미션
## AS-IS
### AS-IS 개선포인트 분석

- __현재 배포 과정__:\
  현재의 배포 과정은 수동으로 진행되며, 이에 따라 서버 간 파일 이동, 서버 접속, 각종 스크립트 실행 등 복잡한 단계를 거쳐야 합니다.
- __통신 및 알림의 부족__:\
  배포가 완료된 후, 지라나 슬랙을 통해 팀원들에게 별도로 알려야 하는 번거로움이 있습니다.
- __배포 코드의 불확실성__:\
  긴급한 요청으로 인해 때때로 feature 브랜치의 코드가 dev 서버에 배포되는 경우가 있어, 어떤 코드가 실제로 서버에 배포되었는지 확인하기 어렵습니다.

### AS-IS 프로세스

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

### Class diagram
- AS-IS 구조에서 개선을 할때 영향을 받게되는 class diagram을 작성한다.

### ERD
-AS-IS 구조에서 개선을 할때 영향을 받게되는 ERD를 작성한다.

## TO-BE 
### TO-BE 기대효과 분석

- __개선 배포 과정__:\
CodeCommit의 stage 브랜치이 업데이트되면 AWS의 코드 비륻와 코드 디플로이를 통해 자동 배포되도록 한다. 
- __알림 기능 추가__:\
  배포가 완료된 후 slack의 어떤 커밋이 올라갔는지 알림이 보내진다

### TO-BE 프로세스

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

### class diagram
- class diagram
```mermaid```
    

### ERD
- TO-BE 구조에서 변경되는 ERD를 작성한다.
```mermaid```

## Task List
1. 코드 커밋 웹훅 추가
2. AWS 파이프라인 설계
    1. 코드 빌드 스크립트 작성
    2. 코드 디플로이 스크립트 작성
    3. EC2 배포 스크립트 작성
    4. 슬랙 메시지 공유 스크립트 작성

## WBS

- 산정 기준 : 4시간/일

1. 요구사항 분석 : 이미수행
2. 설계 : 3d
3. 일정산정: 1d
4. 코드 커밋 웹훅 추가: 1d
5. AWS 파이프라인 설계: 2d
    1. 코드 빌드 스크립트 작성: 2d
    2. 코드 디플로이 스크립트 작성: 2d
    3. EC2 배포 스크립트 작성: 2d
    4. 슬랙 메시지 공유 스크립트 작성: 4d

```mermaid
gantt
    dateFormat  YYYY-MM-DD
    title       결제 재처리 WBS
    excludes    weekends, 2023-12-25, 2024-01-01
    %% (`excludes` accepts specific dates in YYYY-MM-DD format, days of the week ("sunday") or "weekends", but not the word "weekdays".)

    section prepare
    요구사항분석                                     :done,    des1, 2023-12-01, 10d
    설계                                           :active,  des2, 2023-12-11, 3d
    일정산정                                        :active, des3, after des2, 1d
    코드 커밋 웹훅 추가                              :des4, 2024-01-03, 1d

    section AWS 파이프라인 설계
    AWS 파이프라인 스크립트 작성                      :b1, after des4, 2d
    AWS 파이프라인 테스트 스크립트 작성                :b2, after b1, 2d
    코드 빌드 스크립트 작성                          :c1, after b2, 2d
    코드 디플로이 스크립트 작성                       :c2, after c1, 2d
    EC2 배포 스크립트 작성                          :c3, after c2  , 2d
    슬랙 메시지 공유 스크립트 작성                    :c4, after c3  , 4d

    Test & QA                           :after c4, 2d

```

