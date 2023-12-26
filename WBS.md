
# 🪴 career-WBS
> mermaid로 작성된 과제는 마크다운 파일(WBS.md)로 올려주시면 됩니다. (md 파일 내에 기존 구조를 넣어주세요) <br>
> 별도 아키택쳐나 모델링 도구를 사용한 경우에는 마크다운 파일(WBS.md)과 png, gif, jpg, pdf 파일 형식으로 WBS-{gitID}.png 파일명으로 upload 해주세요
# 요구사항
- [x] 개선하려는 프로젝트의 최종 설계
    - [x] 변경 사항에 대한 Target 시스템 설계를 확정한다. (2주차 미션 활용)
    - [x] 변경 사항에 대한 기대효과를 확정한다. (2주차 미션 활용)
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
- 새로 생성하는 프로젝트들이 늘어나고 있다.
- 팀에서 [linter](https://ko.wikipedia.org/wiki/%EB%A6%B0%ED%8A%B8_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4))를 사용하고 있는데 프로젝트들이 생성될 때마다 같은 내용을 복사 + 붙여넣고 있다
- 확장이 필요하다. 프론트, 백엔드 고유의 linter 설정이 존재하고 어떤 경우는 project 개별적으로 적용할 수 있다
 
### AS-IS 프로세스
```mermaid
flowchart TB
    프로젝트 생성 -> 기존 프로젝트에 있는 설정 복사 붙여넣기 -> linter 사용
```

## TO-BE 
### TO-BE 기대효과 분석
- 복사 + 붙여넣기가 사라지고 모듈을 불러오는 방식으로 사용할 수 있다
- 사람이 직접 하는 부분을 자동화 하여 실수를 줄일 수 있다.
    - 잘못된 내용을 복사 붙여넣기 하는 상황을 방지할 수 있다
- 확장성 있게 사용할 수 있다
    - 팀마다 혹은 프로젝트 전용 linter를 설정할 수 있다 
 
### TO-BE 프로세스
```mermaid
flowchart TB
    프로젝트 생성 -> linter 모듈 load
```

## Task List
1. 팀 회의(base에 어떤 linter 설정을 넣을 것인지)
2. base 모듈 구현
3. github package로 배포
4. 새로운 프로젝트로 테스트
5. 기존 프로젝트들에 linter 설정 적용


## WBS

- 산정 기준 : 4시간/일

1. 요구사항 분석 : 이미 수행
2. 설계 : 이미 수행
3. 일정산정: 이미 수행
4. linter 설정 회의: 1d
5. base 모듈 구현: 1d
6. github packages로 배포:0.5d
7. 새로운 프로젝트로 테스트: 0.5d
8. 기존 프로젝트들에 linter 적용하고 테스트: 1d
```mermaid
gantt
    dateFormat  YYYY-MM-DD
    title       linter 모듈 WBS
    excludes    weekends, 2023-12-25, 2024-01-01
    %% (`excludes` accepts specific dates in YYYY-MM-DD format, days of the week ("sunday") or "weekends", but not the word "weekdays".)

    section prepare
    요구사항분석                    :done,    des1, 2023-12-20, 1d
    설계                            :done,  des2, 2023-12-21, 1d
    일정산정                        :done    des3, 2023-12-22, 1d

    section linter 설정 회의
    eslint rule 결정             :crit, b1, 2023-12-26, 1d

    section base 모듈 구현
    eslint-base-config 작성    :c1, after b1, 2023-12-27, 1d
    
    section github packages로 배포
    publish package    :c2, after c1, 2023-12-28, 0.5d
    
    section 새로운 프로젝트로 테스트
    github 프로젝트 생성 후 테스트    :c3, after c2, 2023-12-28, 0.5d

    section 기존 프로젝트들에 linter 적용하고 테스트
    Test & QA                           :after c3, 1d

```

