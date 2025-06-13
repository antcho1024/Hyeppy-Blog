Vacuum Monitoring의 핵심 (조사자료 아니고 그냥 의견..)
1. Autovacuum이 잘 수행되고 있는지
2. 수동 Vacuum 을 해줘야할 상황이 있는지
3. autovacuum 설정값 조정이 필요한지
+a: Bloat, long-running 트랜잭션 감지, Transaction ID Wraparound, Vacuum 리소스 사용 현황 등

참고자료: [데이터독](https://www.datadoghq.com/blog/postgresql-monitoring/)

### Auto Vacuum
자동 Vacuum 설정시 사용자는 신경쓰지 않아도 됨.
그러나, 데이터를 지속적으로 업데이트하거나 삭제하는 경우 Vacuum 일정이 이러한 변경의 속도를 따라가지 못하거나 Vacuum 프로세스가 실행되지 않을 수 있음
⬇️⬇️
모니터링으로 Vacuum이 잘 진행되도록 확인!

#### Dead Row
죽은 행 (업데이트 돼서 쓸모 없는 행. 안읽는 행)의 개수를 보여주는 메트릭
Update가 자주 일어나는 테이블 일수록 수치가 높음

![[Vacuum Monitoring-1743558297188.png]]

pg_stat_user_tables의 `n_dead_tup` 지표는 테이블마다 개별적으로 값이 존재함.
- 테이블 단위를 범례로 해서 보여주기
- 합산 (데이터베이스단위)도 그려줄 수 있으면.. 전체적으로도 볼 수 있을듯 (멀티인스턴스로 볼때는 합산으로 봐야하나)

메트릭 조회 쿼리
```SQL
SELECT relname, n_dead_tup FROM pg_stat_user_tables;

  relname  | n_dead_tup
-----------+------------
 blog_joke |    3780
```




#### Table Disk Usage
실제 디스크 사용량을 모니터링
- Dead row가 많아져도, 실제로 테이블 파일 크기가 커지지 않을 수도 있고 이미 어느 정도 크게 잡혀 있을 수도 있음
![[Vacuum Monitoring-1743559798094.png]]
저 실선에 vacuum 실행한건데
그 이후에 많은 데이터를 넣어도
row 많아져도 디스크 사용량은 늘지 않아 (공간 재활용)

#### The last time a Vacuum or AutoVacuum ran
마지막 Vacuum 시간 
```SQL
SELECT relname, last_vacuum, last_autovacuum FROM pg_stat_user_tables;

               relname               |          last_vacuum          |        last_autovacuum
-------------------------------------+-------------------------------+-------------------------------
 blog_joke                           | 2018-01-23 18:03:28.498505-05 | 2018-01-18 14:56:43.060002-05
```

#### Manual/nightly Vacuum events
자동 Vacuum 설정이 되어 있어도 수동 Vacuum 이 가능함
Datadog에서는 수동 Vacuum을 지원하지는 않지만

수동 Vacuum을 원할 경우 명령어 추천(?) > psql 가서 사용자가 직접 수행 > 실행 로그를 데이터독에서 모니터링 해주는 기능 제공

![[Vacuum Monitoring-1743560185222.png]]



### Autovacuum이 제대로 수행 안되는 상황
- Autovacuum 프로세스 비활성화
```SQL
SELECT name, setting FROM pg_settings WHERE name='autovacuum';
    name    | setting
------------+---------
 autovacuum | on
(1 row)
```

특정 테이블 활성화 확인
```fallback
SELECT reloptions FROM pg_class WHERE relname='my_table';
         reloptions
----------------------------
 {autovacuum_enabled=false}
(1 row)
```
테이블 별로 프로세스 잘 활성화 되어 있는지 모니터링하기

- 주기 조정
autovacuum 데몬은 여러 구성 설정에 의존하여 데이터베이스에서 VACUUM 및 ANALYZE 명령을 자동으로 실행해야 하는 시기를 결정하는데
- `autovacuum_vacuum_threshold`(기본값 50)
- `autovacuum_vacuum_scale_factor`: (기본값 0.2)

잠금에 의해
![[Vacuum Monitoring-1743561252539.png]]![[Vacuum Monitoring-1743561269730.png]]Datadog의 라이브 프로세스 뷰는 VACUUM 프로세스가 대기 상태에 있음을 나타냅니다.


Long running open transaction
PostgreSQL의 MVCC 특성상, 어떤 트랜잭션이 오래 열려서 예전 데이터를 참조하고 있으면 VACUUM이 해당 Dead Row를 제거하지 못함.

![[Vacuum Monitoring-1743567234653.png]]**idle in transaction**이란, DB 연결은 열려 있고 트랜잭션이 시작되었지만 실제 쿼리나 작업을 하지 않고 방치되고 있는 상태


------
참고자료: [비트나인](https://bitnine.net/blog-postgresql/vacuum-%EC%9D%B4%ED%95%B4%EC%99%80-%EC%82%AC%EC%9A%A9%EB%B2%95/)

Auto Vacuum 주기 설정시  고려해야할 사항
- Live tuple과 Dead tuple 비율
- Dead tuple 갯수
- 그 외는 아래 코드 블럭 참고..

default:  테이블의 Live tuple과 Dead tuple 비율(ratio) 80% 미만일 때 Vacuum 

Dead tuple 갯수 threshold 설정하는경우: 테이블 사이즈가 클때.
(why? 10GB 테이블은 2GB의 Dead Tuple이 발생했을 때,
1TB 테이블은 200GB의 Dead Tuple이 발생했을 때 AutoVacuum이 수행된다.
작은 테이블에서는 티가 나지 않겠지만 테이블이 점점 커질수록 AutoVacuum이 수행됐을 때 정리해야 하는 Dead Tuple이 너무 많아지고 부하가 커지는 상황이 발생할 수 있기 때문에 조금 더 자주 수행될 수 있도록 조정이 필요)


```text
- AutoVacuum 수행 Term이 긴 경우
    - autovacuum_vacuum_scale_factor & autovacuum_vacuum_scale_threshold
    - autovacuum_vacuum_insert_scale_factor & autovacuum_vacuum_scale_threshold
- AutoVacuum 수행이 느린 경우
    - autovacuum_vacuum_cost_delay
    - autovacuum_vacuum_cost_limit
    - autovacuum_naptime
    - autovacuum_max_workers
    - autovacuum_work_mem
    - max_parallel_maintenance_workers
```
