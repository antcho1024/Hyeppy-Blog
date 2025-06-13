설치해서  Oracle, PostgreSQL 설치해둔 상태!
설치 방법은 [[Datadog 모니터링 환경 구축하기]]참고!


데이터베이스 모니터링 화면 
- Database Monitoring 메인 화면
모든 데이터베이스 모아보기 (Oracle, Postegres 등 모든 db)



(사진 첨부 예정)

지표 설명
##### Average Active Sessions
AAS (Average Active Sessions) = Active Time / Elapsed Time

- Active Time: 특정 구간 동안 실행중이거나 대기중인 세션들의 누적 시간
- Elapsed Time: 경과 시간 (?)

| 세션 ID | 실행 시작    | 실행 종료    | 실행 시간(초) |
| ----- | -------- | -------- | -------- |
| A     | 12:00:00 | 12:00:10 | 10       |
| B     | 12:00:05 | 12:00:15 | 10       |
| C     | 12:00:08 | 12:00:12 | 4        |
-> Active Time: 10 + 10 + 4 = 24
-> Elapsed Time: 00 ~ 15 = 15

AAS = 24/15= 1.6

해당 구간 (0초-15초) 에서 평균적으로 1.6개의 활성세션이 존재했다~ 라는 의미

보통 표현은 아래처럼 하는듯?

| AAS 값     | 의미                      |
| --------- | ----------------------- |
| **< 1**   | 부하가 적음 (대부분의 세션이 유휴 상태) |
| **1 ~ 2** | 적정 수준의 트랜잭션 처리 중        |
| **> 2**   | 부하 증가 (대기 이벤트 분석 필요)    |
| **> 5**   | 성능 병목 발생 가능성 높음 (튜닝 필요) |

##### Connection(Load) - By wait event
breakdown of average active sessions per wait event
대기 이벤트 별 AAS(평균 활성 세션) 분포

대기 이벤트: I/O 대기, 락 대기, CPU 대기 등

| Wait          | AAS | 비율(%) |
| ------------- | --- | ----- |
| CPU Waits     | 2.5 | 50%   |
| I/O Waits     | 1.5 | 30%   |
| Lock Waits    | 0.8 | 16%   |
| Network Waits | 0.2 | 4%    |
![[Datadog 메뉴분석-1738908041617.png]]


--------------
https://mail.google.com/mail/u/0/#inbox/FMfcgzQZTMPJkwcbzjlpTZjPXkSgLvrT
알람 발생해서 메일 옴

https://us5.datadoghq.com/event/explorer?cols=&event=AwAAAZUh5kFII_LzUgAAABhBWlVoNWtYVkFBRGJNUTBWa3g2VXJ4ZU4AAAAkMDE5NTIxZTYtNDE0OC00MmFkLWI3YzEtYWE1NmI2NjE4YmVlAAAAAA&link_source=monitor_notif&messageDisplay=expanded-lg&options=&refresh_mode=paused&sort=DESC&view=all&from_ts=1740029983000&to_ts=1740030883000&live=false
view event 하니까 일로 옴