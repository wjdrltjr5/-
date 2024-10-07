# 성능 개선

## ngrinder

https://github.com/naver/ngrinder/releases

-   TPS : Transactions Per Second
-   Vuser : Virtural User

-   Peak TPS : 최고 TPS
-   Mean Test Time : 평균 테스트 시간
-   Executed Tests : 평균 테스트 실행 횟수
-   Successful Tests : 테스트 성공 횟수
-   Errors : 에러횟수
-   Runtime : 테스트 실행 시간

## 모니터링 툴 (스카우트)

https://github.com/scouter-project/scouter/releases

spring 서버 실행시 환경변수 옵션 주기

```
-javaagent:/Users/..생략../study/util/scouter/agent.java/scouter.agent.jar -Dscouter.config/Users/jo-eunho/Documents/eunho/study/util/scouter/server/conf/scouter.conf -Dobj_name=demoTomcat --add-opens java.base/java.lang=ALL-UNNAMED
```

### OS에 CPU , Memory 정보를 확인을 위해 “agent.host” 설정 및 실행

-   /scouter/agent.host/conf 경로에 scouter.conf 설정 변경

-   실행( /scouter/agent.host 경로로 이동)
    -   MAC
        -   ./host.sh 실행
    -   Window
        -   host.bat 클릭

## 캐싱

민감하지 않은 데이터를 위주로 캐시 해야함

### Ehcache

-   Spring local 캐시 라이브러리
-   설정 필요 ehcache.xml
-   캐싱 사용할 메서드 지정

## 인덱스 활용

-   인덱스 기능 비교
    -   mysql profiling
-   개발하면서 작성한 쿼리 분석하기

```sql
select * from notice
where createDate between {startDate} and {endDate}

-- 인덱스 설정
create index idx_notice_createDate on notice (createDate);

-- 인덱스 확인
show index from notice;
```

index 효과

-   검색 성능 향상
-   조인 성능 향상
-   정렬 및 그룹화 성능 향상
-   범위 검색 최적화

where절 join절 컬럼이 여러가지라면? <br>
칼럼의 카디널리티 수치(특정 칼럼이나 관계에서 고유한 값을 가지는 정도)를 기준으로 설정하기

-   ex. 사람 테이블에 성별보다 이메일/주민번호는 다른 칼럼에 비해 카디널리티 수치가 높음(더 유니크한 정도)
-   count(중복제거한 컬럼) / count(_) _ 100

실행계획을 통해 인덱스 확인하기

```sql
explain
SELECT * FROM notice
WHERE createDate BETWEEN '2023-01-15 00:00:00' AND '2023-01-15 23:59:59'
;
```

주의 항목에 속한다면 고쳐야 하는 쿼리

-   id: 각 쿼리 블록 또는 서브쿼리에 대해 부여된 고유한 식별자. 여러 서브쿼리가 있는 경우 계층적으로 표현됩니다.

-   select_type: 쿼리의 유형을 나타냅니다. "SIMPLE"은 단순한 SELECT 쿼리를 나타냅니다.
    -   주의 (DEPENDENT, DERIVED)
-   table: 쿼리가 참조하는 테이블의 이름.

-   type: 테이블에서 레코드를 읽는 방법을 나타냅니다. 여기서 "range"는 인덱스를 사용하는 범위 스캔을 의미합니다.
    -   주의 (index, fulltext, all)
-   possible_keys: 쿼리에서 사용될 수 있는 인덱스 목록.

-   key: 실제로 선택된 인덱스.

-   Extra : 기타 정보.

    -   주의(Full scan ~ , Impossible ~, No matching ~, Using filesort, Using temporary, Using join buffer)

-   인덱스가 있더라도 db엔진이 풀테이블스캔이 더 나은 선택이라고 판단하면 인덱스 안탐

HINT 활용하기.

-   데이터베이스 엔진이 최적에 쿼리 실행을 결정 하겠지만
    -   상황에 따라서 개발자가 확인을 하여 쿼리 개선을 해줘야하는 경우도 간혹 있습니다.
        -   ex 과거 실무에서 테이블에 A , B 라는 인덱스가 있고 쿼리 성능이 좋지 않아 실행계획을 확인 해보니 Cardinality 수치가 좋지 않은 B 라는 인덱스를 타고 있는 상황
        -   A 인덱스가 Cardinality 수치가 더 좋아 힌트 기능을 활용하여 강제로 A 인덱스를 타도록 설정하여 성능결과 확인하기.

```sql
CREATE INDEX idx_notice_who ON notice ( who );

show index from notice;

explain
SELECT * FROM notice
WHERE who = 'incu19'
and
    createDate BETWEEN '2023-01-15 00:00:00' AND '2023-01-15 23:59:59'
;


// 힌트 기능을 사용해서 강제로 idx_notice_who 인덱스를 타도록 하기.
explain
SELECT * FROM notice use index(idx_notice_who)
WHERE who = 'incu19'
and
    createDate BETWEEN '2023-01-15 00:00:00' AND '2023-01-15 23:59:59'
;
```

Mysql Profiling

-   Mysql에서는 쿼리가 처리되는 동안 각 단계별 작업이 얼마나 걸렸는지 확인할 수 있는 기능을 제공
-   프로파일링 기능을 활성화 여부 확인

```sql
show variables like '%profiling%';

set profiling = 1;
set profiling_history_size = 100;
```

-   확인할 Query_id 찾기
    -   profiling 기능을 통해 확인하고자 하는 쿼리 실행하기.
    -   show profiles ; 명령어 실행 후 실행한 쿼리를 찾아서 Query_id 찾기

```sql
SELECT * FROM study_db.notice
WHERE createDate BETWEEN '2023-01-15 00:00:00' AND '2023-02-14 23:59:59'
;

show profiles ;
```

-   찾은 Query_id(번호)에 대해 profiling 기능으로 확인하기.
    -   Profile을 통해 조회할 수 있는 세부 목록
        -   BLOCK IO
        -   MEMORY
        -   CPU
        -   CONTEXT SWITCHES
        -   IPC
        -   PAGE FAULTS
        -   SOURCE
        -   SWAPS

```sql
// 해당 쿼리문의 수행 시간을 더 상세한 단위로 확인
show profile for query 번호;

// 해당 쿼리의 CPU 사용량을 분석
show profile cpu for query 번호;
```

이것을 가지고 인덱스 사용전/후 시간비교
