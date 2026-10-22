# [트러블슈팅] CREATE TABLE 실행 시 ORA-01031 (권한이 불충분합니다) 해결 가이드

오라클(Oracle 23ai / 21c / 19c) 실습 환경에서 실습용 일반 사용자(`MANTORMALL`)를 생성하고 `CREATE TABLE`을 실행할 때 발생하는 **`ORA-01031: 권한이 불충분합니다`** 오류의 원인과 단계별 해결 방안을 정리한 문서입니다.

---

## 1. 문제 현상 (Symptom)

일반 사용자(`MANTORMALL`) 세션에서 테이블 생성 쿼리를 실행했을 때 다음과 같은 에러가 발생합니다.

```sql
CREATE TABLE customer_ex_ddl (
    customer_id     NUMBER,        -- 고객번호
    name            VARCHAR2(100), -- 이름
    email           VARCHAR2(100), -- 이메일
    phone           VARCHAR2(20),  -- 전화번호
    registered_date DATE,          -- 가입일
    is_active       CHAR(1)        -- 활성 여부
);
```

- **에러 메시지**:
  ```text
  SQL Error [1031] [42000]: ORA-01031: 권한이 불충분합니다
  Error position: line: 1
  ```
- **DBeaver 화면 현상**: `CREATE` 키워드 아래에 빨간색 물결선과 에디터 좌측에 경고/오류 아이콘(`⚠`)이 표시됨.

---

## 2. 근본 원인 분석 (Root Causes)

사용자에게 `RESOURCE` 롤이 부여되어 있거나 `CREATE TABLE` 권한이 있는 상태에서도 이 오류가 발생하는 이유는 크게 **4가지**입니다.

### ① 기본 테이블스페이스가 `SYSTEM`으로 설정된 경우 (가장 흔함)
- `CREATE USER` 시점에 `DEFAULT TABLESPACE`를 명시하지 않으면 PDB 환경 기본값에 따라 **`SYSTEM` 테이블스페이스**가 기본 저장소로 지정될 수 있습니다.
- 오라클은 데이터베이스 무결성과 보안을 위해 **일반 사용자가 `SYSTEM` 테이블스페이스에 테이블을 만드는 것을 원천 차단(ORA-01031 발생)**합니다.

### ② 테이블스페이스 Quota(할당량) 또는 `UNLIMITED TABLESPACE` 누락
- 오라클 12c 이후(19c, 21c, 23ai)부터는 `RESOURCE` 롤에 **`UNLIMITED TABLESPACE` 권한이 자동으로 포함되지 않습니다.**
- 사용자가 저장 공간(테이블스페이스)에 대한 사용 권한(Quota)이 없으면 `CREATE TABLE` 시점에 즉시 `ORA-01031`이 발생합니다.

### ③ DBeaver의 현재 활성 스키마(`CURRENT_SCHEMA`) 불일치
- 실제 로그인 계정은 `MANTORMALL`이지만, DBeaver 상단 툴바의 '활성 스키마'가 `SYSTEM` 등 다른 계정으로 선택되어 있는 경우:
- 오라클은 이 쿼리를 `CREATE TABLE SYSTEM.customer_ex_ddl`로 인식하며, 다른 사용자의 스키마에 테이블을 생성하려면 `CREATE ANY TABLE` 관리자 권한이 필요하므로 `ORA-01031`이 발생합니다.

### ④ 쿼리 실행 방식 (스크립트 전체 실행 `Alt + X` vs 블록 실행 `Ctrl + Enter`)
- 에디터 상단에 관리자용 명령어나 권한 없는 명령어가 함께 적혀 있는 상태에서 `Alt + X`(전체 실행)를 누르면 일반 계정에서 관리자 명령어가 함께 실행되어 오류가 발생할 수 있습니다.

---

## 3. 원인 진단 쿼리 (현재 세션 확인)

오류가 발생한 SQL 편집기에서 아래 쿼리를 실행하여 현재 상태를 정확히 진단합니다.

```sql
-- 1) 현재 로그인 계정 및 접속 PDB 확인
SELECT USER, SYS_CONTEXT('USERENV', 'CON_NAME') FROM DUAL;
-- 기대값: MANTORMALL, FREEPDB1

-- 2) 현재 세션의 활성 스키마 및 기본 테이블스페이스 확인
SELECT SYS_CONTEXT('USERENV', 'CURRENT_SCHEMA') AS CURRENT_SCHEMA,
       (SELECT DEFAULT_TABLESPACE FROM USER_USERS) AS DEFAULT_TS
FROM DUAL;
-- 점검: CURRENT_SCHEMA가 MANTORMALL이어야 하며, DEFAULT_TS가 SYSTEM이 아니어야 함 (USERS 권장)

-- 3) 현재 세션이 테이블 생성 권한을 실제로 들고 있는지 확인
SELECT * FROM SESSION_PRIVS WHERE PRIVILEGE = 'CREATE TABLE';
-- 기대값: CREATE TABLE 행이 조회되어야 함
```

---

## 4. 단계별 완전 해결 방안

### [해결 1] 관리자(`SYSTEM`) 계정에서 계정 설정 및 권한 완벽 재부여 (근본 해결)

최고 관리자(`SYSTEM`) 세션의 SQL 편집기로 이동하여 아래 쿼리를 순서대로 실행합니다.

```sql
-- 1) 기본 저장 공간을 안전한 'USERS' 테이블스페이스로 지정 및 무제한 할당
ALTER USER MANTORMALL DEFAULT TABLESPACE USERS;
ALTER USER MANTORMALL QUOTA UNLIMITED ON USERS;

-- 2) 데이터베이스 접속 및 기본 객체 생성 롤 부여
GRANT CONNECT, RESOURCE TO MANTORMALL;

-- 3) 전역 테이블스페이스 사용 권한 부여
GRANT UNLIMITED TABLESPACE TO MANTORMALL;

-- 4) 롤(Role) 활성화 이슈와 무관하게 CREATE TABLE 시스템 권한 직접 부여
GRANT CREATE TABLE TO MANTORMALL;
```

---

### [해결 2] 테이블 생성 시 스키마 및 테이블스페이스 명시 (즉시 해결)

`MANTORMALL` 편집기에서 스키마명(`MANTORMALL.`)과 저장될 테이블스페이스(`TABLESPACE USERS`)를 명시하여 실행합니다.

> ⚠️ **주의**: 다른 쿼리와 섞이지 않도록 **반드시 해당 쿼리 블록만 마우스로 드래그(선택)한 후 `Ctrl + Enter`**를 누릅니다.

```sql
CREATE TABLE MANTORMALL.customer_ex_ddl (
    customer_id     NUMBER,        -- 고객번호
    name            VARCHAR2(100), -- 이름
    email           VARCHAR2(100), -- 이메일
    phone           VARCHAR2(20),  -- 전화번호
    registered_date DATE,          -- 가입일
    is_active       CHAR(1)        -- 활성 여부
) TABLESPACE USERS;
```

---

### [해결 3] DBeaver 환경 점검 및 세션 새로고침

1. **활성 스키마 확인**:
   - DBeaver 상단 툴바의 활성 데이터베이스/스키마 드롭다운이 **`<FREEPDB1> MANTORMALL`**로 선택되어 있는지 확인합니다.
2. **연결 새로고침**:
   - 좌측 [데이터베이스 내비게이터]에서 `MANTORMALL` 연결 우클릭 → **연결 해제(Disconnect)** 후 다시 **연결(Connect)**하여 새로 부여된 시스템 권한이 세션에 온전히 반영되도록 합니다.
3. **생성 확인**:
   - 테이블이 정상 생성되었는지 확인:
     ```sql
     SELECT * FROM customer_ex_ddl;
     ```

---

## 5. 핵심 요약 체크리스트

| 점검 항목 | 권장 설정값 | 미설정 시 발생하는 문제 |
| :--- | :--- | :--- |
| **접속 사용자 (`USER`)** | `MANTORMALL` | `SYSTEM` 세션 오염 또는 타 계정 충돌 |
| **활성 스키마 (`CURRENT_SCHEMA`)** | `MANTORMALL` | 타 스키마 생성 시도 (`CREATE ANY TABLE` 필요 에러) |
| **기본 테이블스페이스 (`DEFAULT_TS`)** | `USERS` | `SYSTEM` 공간 생성 차단으로 인한 `ORA-01031` |
| **테이블스페이스 사용 권한** | `UNLIMITED TABLESPACE` 또는 `QUOTA ON USERS` | 데이터 저장 불가 (`ORA-01031` 또는 `ORA-01950`) |
| **시스템 권한** | `CREATE TABLE` 직접 부여 | 롤 미반영 시 `ORA-01031` |
