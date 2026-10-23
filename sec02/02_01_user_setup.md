# Oracle 실습용 사용자(스키마) 생성 및 권한 설정 가이드

Oracle 데이터베이스 실습을 진행하기 위해 최고 관리자(`SYSTEM`) 계정에서 실습용 일반 사용자(`KOREA_MALL`)를 생성하고, 필요한 권한을 부여한 뒤 DBeaver에서 접속을 전환하는 절차입니다.

---

## 1. 실습용 사용자 생성 및 권한 부여 (SYSTEM 계정)

> ⚠️ **주의**: 아래의 계정 생성 및 권한 부여 쿼리는 반드시 **`SYSTEM` (최고 관리자)** 세션의 SQL 편집기에서 실행해야 합니다.

```sql
-- 1) 현재 접속 사용자 확인 (결과가 SYSTEM 이어야 함)
SELECT USER FROM DUAL;

-- 2) 사용자 생성 (계정명: KOREA_MALL, 비밀번호: 1234)
-- ※ Oracle에서 숫자로만 구성된 비밀번호는 큰따옴표("")로 감싸야 오류(ORA-00988)가 발생하지 않습니다.
CREATE USER KOREA_MALL IDENTIFIED BY "1234";

-- 3) 데이터베이스 접속 및 기본 객체 생성 권한 부여
GRANT CONNECT, RESOURCE TO KOREA_MALL;

-- 4) 테이블 데이터 저장을 위한 테이블스페이스 사용 권한 부여
GRANT UNLIMITED TABLESPACE TO KOREA_MALL;
```

---

## 2. 권한 설정 및 SQL 분류 상세 해설

### 2.1 `GRANT`는 어떤 SQL 분류에 해당할까? (DCL: 데이터 제어어)

**네, 맞습니다! `GRANT`는 DCL(Data Control Language, 데이터 제어어)입니다.**

SQL 문법은 목적에 따라 크게 4가지로 분류됩니다:

| 분류 | 영문 명칭 | 주요 명령어 | 설명 |
| :--- | :--- | :--- | :--- |
| **DDL** | Data Definition Language (데이터 정의어) | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | 테이블, 사용자 등 데이터베이스 객체의 구조를 생성·수정·삭제 |
| **DML** | Data Manipulation Language (데이터 조작어) | `SELECT`, `INSERT`, `UPDATE`, `DELETE` | 테이블 내부의 데이터를 조회하고 추가·수정·삭제 |
| **DCL** | **Data Control Language (데이터 제어어)** | **`GRANT`**, **`REVOKE`** | 데이터베이스 접근 및 객체 사용에 관한 **권한을 부여하거나 회수** |
| **TCL** | Transaction Control Language (트랜잭션 제어어) | `COMMIT`, `ROLLBACK`, `SAVEPOINT` | DML 작업 결과를 실제 DB에 영구 반영하거나 취소 |

- **`GRANT`**: 사용자에게 특정 권한이나 롤(Role)을 **부여**합니다.
- **`REVOKE`**: 이미 부여된 권한이나 롤을 다시 **회수(박탈)**합니다.
  ```sql
  -- 예시: 부여했던 테이블스페이스 권한 회수
  REVOKE UNLIMITED TABLESPACE FROM KOREA_MALL;
  ```

---

### 2.2 왜 `CONNECT, RESOURCE`만 지정하는가?

실습 및 애플리케이션 개발용 계정에 **`CONNECT`**와 **`RESOURCE`** 롤(Role)만 부여하는 이유는 **"개발 실습에 필요한 최소한의 권한만 안전하게 부여(최소 권한의 원칙)"**하기 위함입니다.

| 롤(Role) | 포함된 주요 시스템 권한 | 설명 및 목적 |
| :--- | :--- | :--- |
| **`CONNECT`** | `CREATE SESSION` | 데이터베이스에 **로그인(접속 및 세션 생성)**할 수 있는 필수 권한 |
| **`RESOURCE`** | `CREATE TABLE`<br>`CREATE SEQUENCE`<br>`CREATE PROCEDURE`<br>`CREATE TRIGGER`<br>`CREATE TYPE` 등 | 사용자가 **자신의 스키마 내부에서 테이블, 시퀀스, 프로시저 등 주요 객체를 생성하고 관리**할 수 있는 기본 권한 모음 |

#### 핵심 이유
1. **보안 및 스키마 격리 (최소 권한 원칙)**:
   - 만약 편리하다는 이유로 `DBA` 롤을 주면, 다른 사용자의 데이터를 조회/수정/삭제하거나 시스템 설정을 변경할 수 있는 치명적인 위험이 발생합니다.
   - `CONNECT, RESOURCE`만 부여하면 자신의 스키마(`KOREA_MALL`) 안에서만 자유롭게 테이블을 만들고 실습할 수 있어 시스템이 안전합니다.
2. **개발/학습 환경의 표준 템플릿**:
   - 오라클 교재, 자격증(SQLD 등), 실무 개발 환경에서 일반 사용자 계정 생성 시 가장 널리 쓰이는 표준 관례입니다.

> 💡 **참고 (포함되지 않는 권한)**:
> `CREATE VIEW`(뷰 생성)나 `CREATE SYNONYM`(동의어 생성) 등은 `RESOURCE`에 포함되어 있지 않습니다. 추후 뷰 실습 시에는 관리자(`SYSTEM`) 계정에서 `GRANT CREATE VIEW TO KOREA_MALL;`을 추가로 실행해야 합니다.

---

### 2.3 `UNLIMITED TABLESPACE`를 주지 않으면 데이터를 저장하지 못할까?

**네, 맞습니다.** 이 권한(또는 테이블스페이스에 대한 Quota 할당)을 주지 않으면 **테이블에 데이터를 `INSERT`하여 저장할 때 오류가 발생하면서 저장이 되지 않습니다.**

#### 1) 발생하는 오류
```sql
-- 데이터를 삽입하려고 할 때 오류 발생
INSERT INTO customer_ex_ddl (customer_id, name) VALUES (1, '홍길동');
-- ORA-01950: 테이블스페이스 'USERS'에 대한 권한이 없습니다. (no privileges on tablespace 'USERS')
```

#### 2) 왜 `CREATE TABLE` 할 때는 에러가 안 나고 `INSERT` 할 때 날까?
오라클(11g R2 이후)은 **지연 세그먼트 생성(Deferred Segment Creation)** 기능이 기본으로 켜져 있습니다:
- **`CREATE TABLE` 시점**: 실제 디스크 공간을 할당하지 않고 테이블의 정의(메타데이터)만 등록하므로 에러 없이 성공합니다.
- **첫 `INSERT` 시점**: 실제 데이터를 디스크에 쓰기 위해 테이블스페이스(기본: USERS)에 저장 공간(세그먼트) 할당을 요청하며, 이때 사용자에게 할당된 용량(Quota)이 없음을 감지하고 `ORA-01950` 오류를 냅니다.

#### 3) 테이블스페이스 권한 부여 방식 비교 (실습 vs 실무)

| 구분 | 1) `GRANT UNLIMITED TABLESPACE` (실습 권장) | 2) `ALTER USER ... QUOTA` (실무 권장) |
| :--- | :--- | :--- |
| **명령어** | `GRANT UNLIMITED TABLESPACE TO KOREA_MALL;` | `ALTER USER KOREA_MALL QUOTA UNLIMITED ON USERS;`<br>또는 `ALTER USER KOREA_MALL QUOTA 500M ON USERS;` |
| **설명** | DB 내 **모든 테이블스페이스**에 무제한 용량 권한 부여 | **지정한 특정 테이블스페이스**(예: USERS)에만 허용 용량 지정 |
| **용도** | **교육/실습용** (테이블스페이스 구조를 몰라도 에러 없이 간편함) | **실무 운영용** (특정 계정이 디스크 공간을 독점 소진하는 사태 방지) |


---

## 3. 계정 관리 참고 쿼리

계정 운영 중 발생할 수 있는 주요 상황별 관리 명령어입니다.

```sql
-- 1) 비밀번호 변경이 필요한 경우
ALTER USER KOREA_MALL IDENTIFIED BY "새비밀번호";

-- 2) 계정 잠금 해제가 필요한 경우 (로그인 실패 횟수 초과 등으로 잠겼을 때)
ALTER USER KOREA_MALL ACCOUNT UNLOCK;

-- 3) 사용자 및 해당 사용자의 모든 객체(테이블 등) 완전 삭제/초기화
DROP USER KOREA_MALL CASCADE;
```

---

## 4. 실습 사용자 전환 및 세션 확인 (DBeaver)

사용자 생성이 완료되면 DBeaver에서 새 연결을 생성하여 실습용 세션으로 전환합니다.

### 4.1 DBeaver 새 연결(Connection) 생성
1. DBeaver 상단 메뉴: **데이터베이스(Database) → 새 데이터베이스 연결(New Database Connection)**
2. **Oracle** 선택 후 **다음(Next)**
3. 연결 정보 입력:
   - **Host**: `localhost`
   - **Port**: `1521`
   - **Database (Service Name)**: `FREEPDB1`
   - **Username**: `KOREA_MALL`
   - **Password**: `1234`
4. **Test Connection(연결 테스트)** 클릭하여 정상 연결(`Connected`) 확인 후 **완료(Finish)**

### 4.2 실습 세션 확인 및 주의사항
1. 생성된 `KOREA_MALL` 연결을 선택하고 새 SQL 편집기(`Ctrl + ]`)를 엽니다.
2. 아래 쿼리를 실행하여 현재 접속 사용자가 정상적으로 전환되었는지 확인합니다.

```sql
-- 현재 접속 계정 확인 (결과: KOREA_MALL)
SELECT USER FROM DUAL;
```

> ⚠️ **중요**:
> 실습 중 테이블 생성(`CREATE TABLE`), 데이터 삽입(`INSERT`) 등 모든 DDL/DML 실습은 반드시 **`KOREA_MALL` 세션의 SQL 편집기**에서 수행해야 최고 관리자(`SYSTEM`) 스키마를 오염시키지 않습니다.

---

### 4.3 기본 활성 스키마(CURRENT_SCHEMA) 설정 및 고정 (접두사 없이 사용하기)

Oracle에서는 세션의 활성 스키마가 `KOREA_MALL`로 고정되어 있어야 테이블명 앞에 `KOREA_MALL.` 접두사를 일일이 붙이지 않고 간결하게 `SELECT * FROM customers;` 형태로 실습할 수 있습니다. DBeaver에서 기본 스키마를 설정하는 방법은 크게 두 가지가 있습니다:

#### 방법 1: SQL 쿼리로 세션 스키마 변경 (가장 빠르고 확실함)
SQL 편집기에서 아래 명령어를 실행(`Ctrl + Enter`)하면 현재 편집기 세션의 기본 스키마가 `KOREA_MALL`로 즉시 변경됩니다.

```sql
-- 현재 세션의 활성 스키마를 KOREA_MALL로 변경
ALTER SESSION SET CURRENT_SCHEMA = KOREA_MALL;

-- 이후 접두사(KOREA_MALL.) 없이 간결하게 테이블 쿼리 가능
SELECT * FROM customers;
```

#### 방법 2: DBeaver UI에서 기본 스키마 활성화하기
1. **상단 툴바 드롭다운 선택**:
   - SQL 편집기 바로 위(또는 DBeaver 상단 툴바)의 데이터베이스 연결(`FREEPDB1`) 옆 스키마 선택 드롭다운을 클릭하여 **`KOREA_MALL`**을 선택합니다.
2. **데이터베이스 내비게이터에서 설정**:
   - 좌측 트리 메뉴의 [데이터베이스 내비게이터]에서 `FREEPDB1` → `Schemas` → **`KOREA_MALL`** 스키마를 우클릭합니다.
   - **`Set as default` (단축키: `Ctrl + Shift + A`)**를 클릭하여 활성화합니다.

