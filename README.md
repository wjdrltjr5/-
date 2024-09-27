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
