# Oracle DDL(Data Definition Language) 기초 실습 가이드

DDL(데이터 정의어)은 데이터베이스의 객체(사용자, 테이블 등)를 생성·변경·삭제하는 SQL 명령입니다. DDL은 실행 즉시 자동 반영(Auto Commit)되므로 신중하게 실행해야 합니다.

---

## 0. 사전 준비: 실습용 사용자(스키마) 생성 및 전환

### 0.1 실습용 사용자 생성 (SYSTEM 계정에서 실행)

최고 관리자(`SYSTEM`) 계정에서 실습용 일반 사용자(`KOREA_MALL`)를 생성하고 권한을 부여합니다.

```sql
-- 1) 현재 접속 사용자 확인 (결과가 SYSTEM 이어야 함)
SELECT USER FROM DUAL;

-- 2) 사용자 생성 (비밀번호: 1234)
-- ※ Oracle에서 숫자로만 구성된 비밀번호는 큰따옴표("")로 감싸야 오류(ORA-00988)가 발생하지 않습니다.
CREATE USER KOREA_MALL IDENTIFIED BY "1234";

-- 3) 데이터베이스 접속 및 기본 객체 생성 권한 부여
GRANT CONNECT, RESOURCE TO KOREA_MALL;

-- 4) 테이블 데이터 저장을 위한 테이블스페이스 사용 권한 부여
GRANT UNLIMITED TABLESPACE TO KOREA_MALL;

-- [참고] 비밀번호 변경이 필요한 경우
-- ALTER USER KOREA_MALL IDENTIFIED BY "새비밀번호";

-- [참고] 계정 잠금 해제가 필요한 경우 (로그인 실패 횟수 초과 등)
-- ALTER USER KOREA_MALL ACCOUNT UNLOCK;

-- [참고] 사용자 초기화(삭제)가 필요한 경우
-- DROP USER KOREA_MALL CASCADE;
```

### 0.2 실습 사용자 전환 및 세션 확인

1. **DBeaver 새 연결(Connection) 생성**:
   - **Database**: `FREEPDB1`
   - **Username**: `KOREA_MALL`
   - **Password**: `1234`
2. 생성한 `KOREA_MALL` 연결에서 새 SQL 편집기(`Ctrl + ]`)를 열고 아래 쿼리를 실행해 **현재 사용자가 `KOREA_MALL`로 전환되었는지 확인**합니다.

```sql
-- 현재 접속 계정 확인 (결과: KOREA_MALL)
SELECT USER FROM DUAL;
```

> ⚠️ **주의**: 1단계(테이블 생성)부터는 반드시 `KOREA_MALL` 세션의 SQL 편집기에서 진행해야 관리자 스키마(`SYSTEM`)를 오염시키지 않습니다.

---

## 1. 테이블 생성 (CREATE)

새로운 테이블 구조를 정의합니다. 컬럼명과 데이터 타입을 지정하여 테이블을 새로 생성하거나, 기존 테이블의 데이터를 기반으로 복제하여 생성(CTAS)할 수 있습니다.

### 1.1 기본 테이블 생성

```sql
-- [실습 예제 1-1] 고객 테이블 생성
CREATE TABLE customer_ex_ddl (
    customer_id     NUMBER,        -- 고객번호
    name            VARCHAR2(100), -- 이름
    email           VARCHAR2(100), -- 이메일
    phone           VARCHAR2(20),  -- 전화번호
    registered_date DATE,          -- 가입일
    is_active       CHAR(1)        -- 활성 여부
);
```

### 1.2 기존 테이블 복제하여 생성 (CTAS: CREATE TABLE AS SELECT)

기존에 존재하는 테이블의 구조와 데이터를 그대로 조회(`SELECT`)하여 새로운 테이블로 복제 생성합니다.

#### 1) 전체 복제 (구조 + 데이터)

```sql
-- [실습 예제 2-1] customers 테이블 복제하기
-- 기존 customers 테이블 복사하여 새 테이블 생성
CREATE TABLE customers_ex_copy
AS
SELECT *
FROM customers;

SELECT *
FROM customers_ex_copy;
```

#### 2) 데이터 없이 테이블 구조(스켈레톤)만 복제

`WHERE 1=0`과 같이 항상 거짓(False)이 되는 조건을 지정하면, 기존 데이터는 가져오지 않고 테이블 스키마 구조만 복제됩니다.

```sql
-- [실습 예제 2-2] customers 테이블 데이터 없이 테이블 구조만 복사하기
-- 데이터 없이 테이블 구조만 복사
CREATE TABLE customers_ex_empty
AS
SELECT *
FROM customers
WHERE 1=0;

SELECT *
FROM customers_ex_empty;
```

> 💡 **참고 (CTAS 특징)**:
> - `WHERE 1=0` 기법: 초기 적재 데이터 없이 동일한 구조의 빈 테이블을 빠르게 만들 때 현업에서 표준적으로 사용하는 패턴입니다.
> - `NOT NULL` 제약조건은 유지되지만, 기본키(`PRIMARY KEY`)나 외래키(`FOREIGN KEY`), 인덱스 등은 자동 복사되지 않습니다.
> - 실습 환경에 `customers` 테이블이 아직 없다면 앞서 생성한 `customer_ex_ddl`을 대상으로 실습할 수 있습니다 (`FROM customer_ex_ddl;`).

---

## 2. 구조 및 설정 변경 (ALTER)

`ALTER` 명령은 기존 테이블의 구조를 수정하거나, 사용자 계정의 비밀번호 및 설정을 변경할 때 사용합니다.

### 2.1 컬럼 추가 (ADD)

기존 테이블에 새로운 컬럼을 추가합니다.

```sql
-- [실습 예제 3] 고객 테이블에 생일 컬럼 추가하기
-- 고객의 생년월일 컬럼 추가
ALTER TABLE customers_ex_copy
ADD birth_date DATE;

SELECT * FROM customers_ex_copy;
```

### 2.2 컬럼명 변경 (RENAME COLUMN)

테이블 내 기존 컬럼의 이름을 변경합니다.

```sql
-- [실습 예제 4] 전화번호 컬럼 이름 변경하기
-- 컬럼 이름 변경
ALTER TABLE customers_ex_copy
RENAME COLUMN phone TO contact_number;

SELECT * FROM customers_ex_copy;
```

### 2.3 컬럼 데이터 타입 및 크기 수정 (MODIFY)

컬럼의 데이터 타입, 허용 크기(길이) 또는 제약조건을 수정합니다.

#### 1) 컬럼 크기 확장

```sql
-- [실습 예제 5] 이름 컬럼의 크기 확장하기
-- 이름 컬럼 크기 수정
ALTER TABLE customers_ex_copy
MODIFY name VARCHAR2(150);
```

#### 2) 데이터 타입 변경 시 발생하는 오류 확인

```sql
-- [실습 예제 6] 데이터 타입 변경 시 발생할 수 있는 오류
ALTER TABLE customers_ex_copy
MODIFY name NUMBER;
```

> ⚠️ **주의 (ORA-01439 오류)**:
> - 기존 데이터가 들어있는 컬럼의 데이터 타입을 호환되지 않는 타입(`VARCHAR2` ➔ `NUMBER` 등)으로 변경하려고 하면 `ORA-01439: 열의 데이터 유형을 변경하려면 열이 비어 있어야 합니다` 오류가 발생합니다.
> - 데이터 타입을 변경하려면 컬럼 데이터를 모두 비우거나(`NULL` 처리), 새로운 컬럼을 생성하여 형변환 후 데이터를 이전해야 합니다.

### 2.4 컬럼 삭제 (DROP COLUMN / DROP)

테이블에서 불필요한 컬럼을 영구적으로 삭제합니다.

#### 1) 단일 컬럼 삭제

```sql
-- [실습 예제 7] 컬럼 삭제하기
-- birth_date 컬럼 삭제
ALTER TABLE customers_ex_copy
DROP COLUMN birth_date;
```

#### 2) 여러 컬럼 동시 삭제

괄호`()`를 사용하여 2개 이상의 컬럼을 한 번에 삭제할 수 있습니다.

```sql
-- [실습 예제 8] 여러 컬럼 삭제하기
ALTER TABLE customers_ex_copy
DROP (email, contact_number);
```

### 2.5 사용자 비밀번호 및 계정 설정 변경 (ALTER USER)

테이블 외에도 데이터베이스 사용자 계정의 비밀번호를 변경하거나 계정 상태를 관리할 때 `ALTER USER`를 사용합니다.

```sql
-- 1) 사용자 비밀번호 변경 (예: 5678)
ALTER USER KOREA_MALL IDENTIFIED BY "5678";

-- 2) 계정 잠금 (보안 이슈 발생 시 접속 차단)
ALTER USER KOREA_MALL ACCOUNT LOCK;

-- 3) 계정 잠금 해제 (로그인 실패 횟수 초과 등으로 잠긴 경우)
ALTER USER KOREA_MALL ACCOUNT UNLOCK;

-- 4) 다음 로그인 시 비밀번호 강제 변경 설정
ALTER USER KOREA_MALL PASSWORD EXPIRE;
```

> 💡 **참고 (권한)**: 
> - 일반 사용자(`KOREA_MALL`)는 **본인 계정의 비밀번호**를 직접 변경할 수 있습니다.
> - 다른 사용자의 비밀번호 변경, 계정 잠금/해제(`LOCK`/`UNLOCK`)는 관리자(`SYSTEM`) 계정에서 수행해야 합니다.

---

## 3. 테이블 이름 변경 (RENAME)

테이블의 명칭을 다른 이름으로 변경합니다.

```sql
-- [실습 예제 9] 테이블 이름 변경하기
-- 테이블 이름 변경
ALTER TABLE customers_ex_copy RENAME TO customers_ex_rename;

-- 새 이름으로 테이블 조회
SELECT * FROM customers_ex_rename;
```

---

## 4. 테이블 초기화 vs 삭제 (TRUNCATE & DROP)

데이터만 초기화할 것인지, 테이블 구조 자체를 삭제할 것인지 목적에 따라 분리하여 사용합니다.

### 4.1 데이터 완전 초기화 (TRUNCATE)

테이블 구조는 남겨두고 모든 레코드(행)를 비웁니다. DML(`DELETE`)과 달리 롤백(ROLLBACK)이 불가능합니다.

```sql
-- [실습 예제 10] 테이블 데이터 전체 삭제하기
-- 테이블의 모든 데이터 삭제
TRUNCATE TABLE customers_ex_rename;

SELECT * FROM customers_ex_rename;
```

### 4.2 테이블 삭제 (DROP)

테이블 구조와 데이터를 데이터베이스에서 완전히 삭제합니다.

```sql
-- [실습 예제 11] 테이블 삭제하기
-- 테이블 삭제
DROP TABLE customers_ex_rename;
```

테이블이 실제로 삭제되었는지 확인하려면 데이터 사전(Data Dictionary) 뷰를 조회할 수 있습니다.

```sql
-- USER_TABLES 뷰에서 삭제 여부 확인
SELECT table_name
FROM user_tables
WHERE table_name = 'CUSTOMERS_EX_RENAME';
```

> 💡 **참고 (휴지통과 PURGE 옵션)**:
> - Oracle에서는 기본적으로 `DROP TABLE` 실행 시 테이블이 오라클 휴지통(`RECYCLEBIN`)으로 이동하여 필요 시 복구(`FLASHBACK TABLE`)할 수 있습니다.
> - 휴지통을 거치지 않고 디스크 공간까지 즉시 영구 삭제하려면 `PURGE` 옵션을 함께 사용합니다 (`DROP TABLE 테이블명 PURGE;`).
> - 휴지통 비우기: `PURGE RECYCLEBIN;`

---

## 5. DDL 핵심 요약

| 분류 | 명령문 | 동작 설명 | 비고 |
| --- | --- | --- | --- |
| **사용자 관리** | `CREATE USER` | 신규 사용자(스키마) 생성 | `GRANT`로 권한 부여 필요 |
| **계정 속성 변경** | `ALTER USER` | 사용자 비밀번호 변경, 계정 잠금/해제 | `IDENTIFIED BY`, `ACCOUNT LOCK` |
| **객체 정의** | `CREATE TABLE` | 새로운 테이블 생성 | 기본키, 제약조건 설정 |
| **테이블 복제** | `CREATE TABLE ... AS` | 기존 테이블 구조 및 데이터 복제 생성 | CTAS (제약조건/인덱스 미복사) |
| **테이블 구조 변경** | `ALTER TABLE` | 테이블 컬럼 추가/수정/삭제 | `ADD`, `MODIFY`, `DROP` |
| **이름 변경** | `RENAME` | 테이블 명칭 변경 | `RENAME old TO new` |
| **데이터 비우기** | `TRUNCATE` | 테이블 구조 유지 + 데이터 전체 삭제 | 롤백 불가, 고속 처리 |
| **객체 삭제** | `DROP TABLE` | 테이블 구조 및 데이터 영구 제거 | `PURGE` 옵션 권장 |
