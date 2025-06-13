## Connection 상태 구분
- Idle 상태: 세션은 생성되었지만 **아무 작업도 하지 않고 있는 상태**.
- Active 상태: 세션이 실제로 **작업 중**인 상태.**쿼리 실행 중**, **I/O 작업**, **대기(waiting)** 등의 활동을 수행 중인 세션.


**Active** = Running + Waiting
- **Active 세션**:
    - **작업 중이거나, 대기(waiting)** 중인 세션.
    - 예: 쿼리를 실행하거나 I/O, Lock 등 특정 리소스를 기다리는 상태.
- **Running 세션**:
    - **실제로 CPU에서 실행 중인 세션**.
    - 대기(waiting) 상태가 아니라, DB 작업이 진행되고 있는 상태.

#### (1) Oracle의 Connection 종류
Oracle에서는 **Session(세션)** 또는 **Connection**이라는 용어로 상태를 구분하며, 주요 상태는 다음과 같습니다:

1. **Inactive (Idle) Session**
    - **의미**: DB와 연결은 되어 있지만 작업이 없는 상태.
    - 예시:
        - 사용자가 SQL*Plus 또는 애플리케이션에서 세션을 열었지만 쿼리를 실행하지 않은 상태.
    - 확인: `SELECT sid, status, username, program FROM v$session WHERE status = 'INACTIVE';`
        
2. **Active Session**
    - **의미**: SQL 쿼리를 실행 중이거나 자원을 대기(waiting) 중인 세션.
    - 확인: `SELECT sid, status, username, program FROM v$session WHERE status = 'ACTIVE';`
        
3. **Background Session**
    - **의미**: Oracle 데이터베이스가 사용하는 내부 백그라운드 프로세스.
    - 주요 프로세스:
        - `DBWn`: 데이터베이스 쓰기 작업
        - `LGWR`: REDO 로그 기록
        - `SMON`, `PMON`: 시스템 및 프로세스 관리
    - 확인: `SELECT sid, username, program FROM v$session WHERE type = 'BACKGROUND';`
        
4. **Blocked Session**
    - **의미**: 다른 세션이 자원을 점유하고 있어 작업이 지연된 상태.
    - 확인: `SELECT * FROM v$session WHERE blocking_session IS NOT NULL;`
        
5. **Killed Session**
    - **의미**: 관리자가 세션을 종료 요청했지만, 아직 정리되지 않은 상태.
    - 확인: `SELECT sid, status FROM v$session WHERE status = 'KILLED';`
        
---

#### (2) PostgreSQL의 Connection 종류
PostgreSQL에서는 `pg_stat_activity` 뷰를 통해 세션(Connection) 상태를 확인할 수 있습니다. 주요 상태는 다음과 같습니다:

1. **Idle**
    - **의미**: 클라이언트가 DB에 연결되었지만 쿼리를 실행하지 않는 상태.
    - 표시: `state = 'idle'`
2. **Active**
    - **의미**: 클라이언트가 SQL 쿼리를 실행 중인 상태.
    - 표시: `state = 'active'`
3. **Idle in Transaction**
    - **의미**: 트랜잭션을 시작했으나 커밋/롤백하지 않고 대기 중인 상태.
    - 주의: 장기간 `idle in transaction` 상태가 지속되면 **Lock** 문제를 유발할 수 있음.
    - 표시: `state = 'idle in transaction'`
4. **Idle in Transaction (Aborted)**
    - **의미**: 트랜잭션이 실패한 후, 종료되지 않고 대기 중인 상태.
    - 표시: `state = 'idle in transaction (aborted)'`
5. **Waiting**
    - **의미**: 쿼리 실행 중 특정 자원(I/O, Lock 등)을 대기하는 상태.
    - 확인: `SELECT * FROM pg_stat_activity WHERE wait_event IS NOT NULL;`
        
6. **Background Worker**
    - **의미**: PostgreSQL에서 내부적으로 사용하는 백그라운드 프로세스.
    - 예: `autovacuum worker`, `logical replication worker`.
    - 표시: `state = 'background'`
7. **Replication**
    - **의미**: PostgreSQL의 복제(slave) 서버에서 연결된 세션.
    - 표시: `state = 'replication'`