Datadog > Database 모니터링 기능 기록
참고: [docs](https://docs.datadoghq.com/ko/database_monitoring/)


#### 표준화된 쿼리
동일한 SQL 쿼리를 여러 번 실행할 때, **중복된 정보를 제거하고 정리**된 것. 
표준화된 쿼리(Standardized Query) 혹은 쿼리 다이제스트(Query Digest) 라고 함

숫자, 날짜, 문자열 등의  값(바인드 파라미터)을 `?`로 대체 > 동일한 구조의 SQL 하나의 그룹으로 묶음
```SQL
SELECT * FROM customers WHERE id = 13345;
SELECT * FROM customers WHERE id = 24435;
SELECT * FROM customers WHERE id = 34322;

이와 같은 쿼리를 아래와 같이 표현

SELECT * FROM customers WHERE id = ?
```

효과: 
- 중복 제거(그룹화)
- 성능 트렌드 분석(어떤 유형의 쿼리가 자주 실행되는지 파악)
- 데이터 보호
- 저장공간 줄임(그룹화를 통해)(패턴만 저장하니까)

*Datadog의 데이터베이스 모니터링에서 **표준화된 쿼리(Standardized Query)**는 **바인딩 변수를 사용한 쿼리와 유사한 방식**으로 동작*

*참고*
Bind Variable 란?
SQL에서 특정 값 대신 **변수(Placeholder)를 사용하는 기법**
```SQL
바인딩 변수를 사용하지 않은 SQL (일반 SQL)
SELECT * FROM customers WHERE id = 13345;
SELECT * FROM customers WHERE id = 24435;
SELECT * FROM customers WHERE id = 34322;

바인딩 변수를 사용한 SQL 
SELECT * FROM customers WHERE id = :customer_id;
EXECUTE 'SELECT * FROM customers WHERE id = :customer_id' USING 12345;
```
:customer_id → **바인딩 변수(Bind Variable)**

이와 같이 사용했을 때
데이터베이스는 **SQL을 동일한 쿼리로 인식**하여 실행 계획을 재사용 가능 → **성능 최적화**

(로그에는 변수 포함한 쿼리가 저장될 수도 있음?, 사용자 쿼리 외의 쿼리는 그냥 저장될 수도 있음?, 주석은 난독화 안함)
### 쿼리 샘플
솔직히 잘 안와닿음: docs 링크 > [링크](https://docs.datadoghq.com/ko/database_monitoring/query_samples/)

![[Datadog-1739854242498.png]]

Datadog이 특정 순간에 찍은 SQL 모음
"느리거나 자주 실행된 쿼리" 위주로 랜덤 저장 (샘플링 정책이 있기 때문에, 모든 쿼리를 100% 저장하지 않음)
저장 제한 없음.
메트릭값을 저장하지 않음


### Tracked Queries
[docs 링크](https://docs.datadoghq.com/ko/database_monitoring/query_metrics/)
![[Datadog-1739854479280.png]]

수집 주기마다 총 실행 시간이 긴 상위 200개 표준화된 쿼리에 대해 쿼리당 메트릭을 수집, 저장
클릭시 Normalized Query로 이동 (Query Detail 인듯)
느린 쿼리 위주?
### Database Host 탐색
[docs 링크](https://docs.datadoghq.com/ko/database_monitoring/database_hosts/)

모든 DB 인스턴스 노출
![[Datadog-1739854889893.png]]

호스트 클릭시 Instance Monitoring

### Instance Monitoring
위 화면에서 특정 Instance 클릭시 인스턴스 상세 모니터링 가능함. 
DB 별로 Page에 노출되는 항목이 다름 (SysMaster에서 각 메뉴로 동작)

![[Datadog-1739855034655.png]]

![[Datadog-1739855384812.png]]
상위는 Summary 차트
하위는 각 탭 (SysMaster 메뉴)

#### Top Query
상위 쿼리 노출
쿼리 클릭시 Normalized Query 로 이동 (Query Detail 같은..)

#### Active connections
실행 중인 실시간 쿼리가 표시 (기간 표기 가능)
하위에 스냅샷? 표시되고 클릭시 Query Sample로 이동됨?
#### Blocking Queris 

#### Samples

#### Calling Services

### Query Detail (Normalized Query)
[docs 링크](https://docs.datadoghq.com/ko/database_monitoring/query_metrics/#%EC%BF%BC%EB%A6%AC-%EC%83%81%EC%84%B8-%EC%A0%95%EB%B3%B4-%ED%8E%98%EC%9D%B4%EC%A7%80)

쿼리 상세 정보
이 쿼리가 수행된 인스턴스 목록도 볼 수 있음