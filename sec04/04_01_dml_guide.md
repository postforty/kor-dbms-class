# Oracle DML(Data Manipulation Language) 기초 실습 가이드

DML(데이터 조작어)은 데이터베이스 테이블에 저장된 데이터를 조회(`SELECT`), 추가(`INSERT`), 수정(`UPDATE`), 삭제(`DELETE`)하는 SQL 명령입니다. DDL과 달리 DML 작업은 트랜잭션의 일부로 처리되므로 작업 결과를 데이터베이스에 영구 반영하기 위해 `COMMIT`을 실행하거나 취소하기 위해 `ROLLBACK`을 사용할 수 있습니다.

---

## 0. 사전 준비: 실습 사용자 및 세션 확인

모든 DML 실습은 최고 관리자(`SYSTEM`)가 아닌 **실습용 일반 사용자(`MANTORMALL`)** 세션의 SQL 편집기에서 진행합니다.

```sql
-- 현재 접속 계정 확인 (결과가 MANTORMALL 이어야 함)
SELECT USER FROM DUAL;
```

> 🎯 **핵심 배움 포인트 (실습 환경 격리와 최소 권한 원칙)**:
> - 최고 관리자(`SYSTEM`)는 인프라 관리 및 사용자 권한 관리를 위한 특권 계정입니다.
> - 데이터 조작(DML) 및 일반 실습은 반드시 일반 업무용 사용자(`MANTORMALL`) 세션에서 진행하여 시스템 카탈로그 오염 및 불필요한 장애를 방지합니다.

---

## # 실습 예제 1 : 연습용 테이블 생성하기

DML 실습(데이터 추가, 수정, 삭제)을 진행하기 위해 먼저 실습 대상이 될 고객 테이블(`customers_ex_dml`)을 생성합니다.

### 1.1 테이블 생성 및 조회 쿼리

```sql
-- DML 연습을 위한 고객 테이블 생성
CREATE TABLE customers_ex_dml (
    customer_id      NUMBER,         -- 고객 고유 ID
    name             VARCHAR2(100),  -- 고객 이름
    email            VARCHAR2(100),  -- 이메일 주소
    phone            VARCHAR2(20),   -- 연락처
    registered_date  DATE,           -- 가입일자
    is_active        CHAR(1)         -- 활성 상태(Y/N)
);

SELECT * FROM customers_ex_dml;
```

---

### 1.2 컬럼 명세 설명

| 컬럼명 | 데이터 타입 | 설명 | 비고 |
| :--- | :--- | :--- | :--- |
| **`customer_id`** | `NUMBER` | 고객 고유 식별 번호 (ID) | 숫자형 데이터 |
| **`name`** | `VARCHAR2(100)` | 고객 이름 | 최대 100바이트 가변 길이 문자열 |
| **`email`** | `VARCHAR2(100)` | 이메일 주소 | 최대 100바이트 가변 길이 문자열 |
| **`phone`** | `VARCHAR2(20)` | 고객 연락처 (전화번호) | 최대 20바이트 가변 길이 문자열 |
| **`registered_date`** | `DATE` | 고객 가입 일자 | 날짜 및 시간 정보 저장 |
| **`is_active`** | `CHAR(1)` | 고객 계정 활성 상태 | 고정 길이 문자형 (`'Y'` 또는 `'N'`) |

---

### 1.3 참고: 실습 초기화(테이블 삭제)

실습 도중 테이블을 처음부터 다시 생성하고 싶은 경우 아래 명령을 통해 테이블을 삭제할 수 있습니다.

```sql
-- 기존 연습용 테이블 삭제
DROP TABLE customers_ex_dml;
```

> 🎯 **핵심 배움 포인트 (DDL과 DML의 본질적 차이)**:
> - `CREATE TABLE`, `DROP TABLE`과 같은 데이터 정의어(DDL)는 실행 즉시 **자동 커밋(Auto-commit)**되어 `ROLLBACK`으로 되돌릴 수 없습니다.
> - 반면 이후 다룰 데이터 조작어(DML)는 트랜잭션 내에서 처리되므로, 명시적으로 확정(`COMMIT`)하기 전까지 언제든 취소(`ROLLBACK`)할 수 있습니다.
> - DML 실습을 진행할 때는 원본 데이터를 보호하기 위해 격리된 샌드박스(연습용) 테이블을 마련하여 실습하는 것이 기본 원칙입니다.

---

## # 실습 예제 2 : 단일 행 입력하기

`INSERT INTO` 문을 사용하여 `customers_ex_dml` 테이블에 고객 1명의 정보를 새로 추가(단일 행 입력)합니다.

### 2.1 단일 행 데이터 입력 쿼리

```sql
-- 고객 1명 추가
INSERT INTO customers_ex_dml (customer_id, name, email, phone, registered_date, is_active)
VALUES (1, '김하늘', 'kh@mm.com', '010-1000-0001', SYSDATE, 'Y');

-- 입력된 데이터 조회 및 확인
SELECT * FROM customers_ex_dml;
```

### 2.2 입력 값 및 컬럼 매핑 설명

| 컬럼명 | 입력 값 | 설명 |
| :--- | :--- | :--- |
| **`customer_id`** | `1` | 고객 고유 번호 (숫자형) |
| **`name`** | `'김하늘'` | 고객 이름 (문자열) |
| **`email`** | `'kh@mm.com'` | 이메일 주소 (문자열) |
| **`phone`** | `'010-1000-0001'` | 연락처 (문자열) |
| **`registered_date`** | `SYSDATE` | Oracle 내장 함수로 현재 시스템 날짜/시간 저장 |
| **`is_active`** | `'Y'` | 계정 활성화 여부 (`'Y'`/`'N'`) |

> 🎯 **핵심 배움 포인트 (컬럼 명시와 1:1 매핑의 원칙)**:
> - `INSERT INTO 테이블 (컬럼목록) VALUES (값목록)` 구조에서 괄호 안의 **컬럼 순서와 값의 순서 및 데이터 타입이 1:1로 엄격히 대응**해야 합니다.
> - DML 실행 후 통계 창에 출력되는 **`Updated Rows: 1`**을 확인하여 의도한 건수만큼 데이터가 정상 반영되었는지 반드시 체크해야 합니다.
> - DDL과 달리 DML 작업은 트랜잭션의 일부로 관리되므로, 작업 결과를 데이터베이스에 영구 반영(`COMMIT;`)하거나 즉시 취소(`ROLLBACK;`)할 수 있습니다.

---

## # 실습 예제 3 : 전체 컬럼 입력하기 (컬럼명 생략)

`INSERT INTO` 절에서 컬럼명 목록을 생략하는 방식입니다. 컬럼명을 생략할 경우 테이블 생성 시 정의된 순서와 개수에 맞추어 모든 컬럼 값을 순서대로 입력해야 합니다.

### 3.1 전체 컬럼 데이터 입력 쿼리

```sql
-- 컬럼명 생략의 경우 테이블 정의 순서와 동일하게 모든 컬럼 값 입력
INSERT INTO customers_ex_dml
VALUES (2, '박서준', 'psj@mm.com', '010-1000-0002', SYSDATE, 'Y');

SELECT * FROM customers_ex_dml;
```

### 3.2 입력 값 및 순서 매핑 설명

컬럼명을 기술하지 않았으므로 테이블에 정의된 순서대로 매핑됩니다.

| 순서 | 매핑 컬럼명 | 입력 값 | 설명 |
| :---: | :--- | :--- | :--- |
| **1** | `customer_id` | `2` | 고객 고유 번호 (숫자형) |
| **2** | `name` | `'박서준'` | 고객 이름 (문자열) |
| **3** | `email` | `'psj@mm.com'` | 이메일 주소 (문자열) |
| **4** | `phone` | `'010-1000-0002'` | 연락처 (문자열) |
| **5** | `registered_date` | `SYSDATE` | 현재 시스템 날짜/시간 저장 |
| **6** | `is_active` | `'Y'` | 계정 활성화 여부 (`'Y'`/`'N'`) |

> 🎯 **핵심 배움 포인트 (테이블 스키마 물리 순서 의존성과 위험성)**:
> - 컬럼명을 생략할 경우 테이블 생성 시 정의된 물리적 컬럼 순서대로 모든 값을 빠짐없이 전달해야 합니다. 일부 값이 누락되면 `ORA-00947: not enough values` 오류가 발생합니다.
> - 테이블에 새로운 컬럼이 추가되거나 순서가 변경되면 기존 쿼리가 즉시 파괴되므로, **실무 애플리케이션 개발에서는 반드시 실습 예제 2처럼 컬럼명을 명시하는 것이 표준**입니다.

---

## # 실습 예제 4 : 일부 컬럼만 입력하기

모든 컬럼의 값을 다 알지 못하거나 입력할 필요가 없을 때, 원하는 컬럼만 명시하여 데이터를 입력할 수 있습니다. 입력 목록에서 생략된 컬럼에는 자동으로 `NULL`(기본값이 설정된 경우 DEFAULT 값)이 대입됩니다.

### 4.1 일부 컬럼 데이터 입력 쿼리

```sql
-- 이메일과 전화번호 없이 고객 정보 입력
INSERT INTO customers_ex_dml (customer_id, name, registered_date, is_active)
VALUES (3, '이보영', SYSDATE, 'Y');

SELECT * FROM customers_ex_dml;
```

### 4.2 입력 값 및 컬럼 매핑 설명

| 컬럼명 | 입력 여부 | 입력/처리 값 | 설명 |
| :--- | :---: | :--- | :--- |
| **`customer_id`** | O | `3` | 고객 고유 번호 (숫자형) |
| **`name`** | O | `'이보영'` | 고객 이름 (문자열) |
| **`email`** | **X (생략)** | `NULL` | 명시하지 않아 자동으로 `NULL` 처리 |
| **`phone`** | **X (생략)** | `NULL` | 명시하지 않아 자동으로 `NULL` 처리 |
| **`registered_date`** | O | `SYSDATE` | 현재 시스템 날짜/시간 저장 |
| **`is_active`** | O | `'Y'` | 계정 활성화 여부 (`'Y'`/`'N'`) |

> 🎯 **핵심 배움 포인트 (암시적 NULL 대입과 NOT NULL 제약조건)**:
> - `INSERT` 컬럼 목록에서 특정 컬럼을 생략하면 기본값(`DEFAULT`) 또는 빈 값인 `NULL`이 암시적으로 자동 대입됩니다.
> - 생략된 컬럼에 `NOT NULL` 제약조건이 걸려 있고 기본값(`DEFAULT`)이 없다면 `ORA-01400: cannot insert NULL into ...` 오류가 발생하므로, 생략 전 테이블 제약조건을 항상 파악해야 합니다.

---

## # 실습 예제 5 : NULL의 명시적 입력하기

컬럼을 생략하여 간접적으로 `NULL`이 대입되도록 하는 방식(실습 예제 4)과 달리, `VALUES` 절에 키워드 **`NULL`**을 직접 기술하여 빈 값을 명시적으로 입력하는 방식입니다.

### 5.1 NULL 명시적 입력 쿼리

```sql
-- 이메일과 전화번호에 NULL을 명시적으로 입력
INSERT INTO customers_ex_dml
(customer_id, name, email, phone, registered_date, is_active)
VALUES
(4, '정해인', NULL, NULL, SYSDATE, 'Y');

-- 등록된 고객 데이터 조회
SELECT * FROM customers_ex_dml WHERE customer_id = 4;
```

### 5.2 입력 값 및 컬럼 매핑 설명

| 컬럼명 | 입력 값 | 설명 |
| :--- | :--- | :--- |
| **`customer_id`** | `4` | 고객 고유 번호 (숫자형) |
| **`name`** | `'정해인'` | 고객 이름 (문자열) |
| **`email`** | **`NULL`** | 명시적으로 빈 값(`NULL`) 지정 |
| **`phone`** | **`NULL`** | 명시적으로 빈 값(`NULL`) 지정 |
| **`registered_date`** | `SYSDATE` | 현재 시스템 날짜/시간 저장 |
| **`is_active`** | `'Y'` | 계정 활성화 여부 (`'Y'`/`'N'`) |

> 🎯 **핵심 배움 포인트 (코드 명확성과 명시적 NULL의 가치)**:
> - **암시적 입력 (예제 4)**: 컬럼 목록에서 제외하여 데이터베이스 기본 처리(`NULL`)에 맡김
> - **명시적 입력 (예제 5)**: `VALUES` 절에 `NULL` 키워드를 직접 기입
> - 실수로 컬럼을 누락한 것이 아니라 **"개발자가 의도적으로 값을 비워두었다"**는 비즈니스 의도와 코드 가독성을 명확히 드러내기 위해 실무에서는 명시적 입력을 권장합니다.

---

## # 실습 예제 6-1 : 문자형으로 날짜 데이터 저장하기

Oracle의 `DATE` 타입 컬럼에 `'YYYY-MM-DD'` 형식의 문자열 리터럴을 직접 입력하는 방법입니다. 데이터베이스가 세션의 날짜 형식에 맞춰 문자열을 `DATE` 타입으로 자동(묵시적) 형변환하여 저장합니다.

### 6-1.1 문자형 날짜 데이터 입력 쿼리

```sql
-- 문자형으로 날짜 데이터 저장하기
INSERT INTO customers_ex_dml
(customer_id, name, email, phone, registered_date, is_active)
VALUES
(5, '신민아', 'sma@mm.com', '010-1000-0005', '2025-01-05', 'Y');

-- 결과 확인
SELECT * FROM customers_ex_dml WHERE customer_id = 5;
```

### 6-1.2 입력 값 및 컬럼 매핑 설명

| 컬럼명 | 입력 값 | 설명 |
| :--- | :--- | :--- |
| **`customer_id`** | `5` | 고객 고유 번호 (숫자형) |
| **`name`** | `'신민아'` | 고객 이름 (문자열) |
| **`email`** | `'sma@mm.com'` | 이메일 주소 (문자열) |
| **`phone`** | `'010-1000-0005'` | 연락처 (문자열) |
| **`registered_date`** | `'2025-01-05'` | 문자열 리터럴로 날짜 전달 (DATE 형으로 묵시적 형변환) |
| **`is_active`** | `'Y'` | 계정 활성화 여부 (`'Y'`/`'N'`) |

> 🎯 **핵심 배움 포인트 (묵시적 형변환과 NLS 환경 종속의 위험)**:
> - `'2025-01-05'`처럼 문자열을 날짜 컬럼에 직접 입력하면 DBMS가 내부적으로 `DATE` 타입으로 자동(묵시적) 변환합니다.
> - 하지만 접속 환경이나 서버의 `NLS_DATE_FORMAT` 설정이 다를 경우(예: `DD-MON-RR`) 변환 오류가 발생하므로, 환경에 종속적인 묵시적 변환은 실무 코드에서 지양해야 합니다.

---

## # 실습 예제 6-2 : TO_DATE 함수를 이용한 날짜 입력하기

문자열 리터럴 입력 시 발생할 수 있는 환경별 포맷 오류를 방지하기 위해, Oracle의 내장 함수인 **`TO_DATE()`**를 사용하여 날짜 형식(`'YYYY-MM-DD'`)을 명시적으로 지정하여 저장하는 방식입니다.

### 6-2.1 TO_DATE 함수를 이용한 데이터 입력 쿼리

```sql
-- TO_DATE 함수를 이용한 날짜 입력
INSERT INTO customers_ex_dml
(customer_id, name, email, phone, registered_date, is_active)
VALUES
(6, '이종석', 'ljs@mm.com', '010-1000-0006',
 TO_DATE('2025-01-10', 'YYYY-MM-DD'), 'Y');

-- 결과 확인
SELECT * FROM customers_ex_dml WHERE customer_id = 6;
```

### 6-2.2 입력 값 및 컬럼 매핑 설명

| 컬럼명 | 입력 값 | 설명 |
| :--- | :--- | :--- |
| **`customer_id`** | `6` | 고객 고유 번호 (숫자형) |
| **`name`** | `'이종석'` | 고객 이름 (문자열) |
| **`email`** | `'ljs@mm.com'` | 이메일 주소 (문자열) |
| **`phone`** | `'010-1000-0006'` | 연락처 (문자열) |
| **`registered_date`** | `TO_DATE('2025-01-10', 'YYYY-MM-DD')` | 포맷을 지정하여 문자열을 안전하게 `DATE` 타입으로 명시적 변환 |
| **`is_active`** | `'Y'` | 계정 활성화 여부 (`'Y'`/`'N'`) |

> 🎯 **핵심 배움 포인트 (명시적 형변환 TO_DATE와 이식성 확보)**:
> - 날짜 포맷(`'YYYY-MM-DD'`)을 명시하여 변환하므로 어떤 클라이언트나 오라클 서버 환경에서도 동일하게 동작하는 **높은 이식성과 안정성**을 가집니다.
> - 시간 정보까지 보존해야 할 경우 `TO_DATE('2025-01-10 14:30:00', 'YYYY-MM-DD HH24:MI:SS')`와 같이 시·분·초 포맷을 지정하여 정밀하게 입력하는 패턴을 숙지해야 합니다.

---

## # 실습 예제 7 : 고객 데이터 복사하기

`VALUES` 절 대신 `SELECT` 서브쿼리를 결합하여, 기존 테이블(`customers`)에 저장된 데이터를 조회한 뒤 대상 테이블(`customers_ex_dml`)에 한 번에 일괄 복사(다중 행 입력)하는 방식입니다.

### 7.1 서브쿼리를 이용한 데이터 복사 입력 쿼리

```sql
-- MENTORMALL의 customers 테이블에서 customer_id가 7번 이상인 고객 복사 입력
INSERT INTO customers_ex_dml
(customer_id, name, email, phone, registered_date, is_active)
SELECT customer_id, name, email, phone, registered_date, is_active
FROM customers
WHERE customer_id >= 7;

-- 결과 확인
SELECT * FROM customers_ex_dml
ORDER BY customer_id;
```

### 7.2 쿼리 동작 원리 및 컬럼 매핑 설명

- **`INSERT INTO customers_ex_dml (...)`**: 데이터를 적재할 대상 테이블과 컬럼 목록을 지정합니다.
- **`SELECT ... FROM customers WHERE customer_id >= 7;`**: 원본 테이블(`customers`)에서 `customer_id`가 7 이상인 레코드들을 조회합니다.
- **일괄 매핑 삽입**: `SELECT` 절에서 조회된 컬럼 순서 및 타입이 `INSERT` 대상 컬럼과 1:1로 매핑되어 조건에 만족하는 다중 행이 일괄 삽입됩니다.

> 🎯 **핵심 배움 포인트 (대량 배치 적재와 집합적 사고)**:
> - `VALUES` 절을 사용하지 않고 `SELECT` 서브쿼리를 결합하여 조건에 맞는 복수 행을 원자적으로 일괄 적재(Batch Insert)합니다.
> - `INSERT` 대상 컬럼과 `SELECT` 컬럼의 명칭이 다를 수는 있지만, **나열된 컬럼의 개수와 데이터 타입이 1:1로 엄격히 일치**해야 합니다.
> - 애플리케이션에서 반복문으로 건별 `INSERT`를 수행하는 것보다 디스크 I/O와 네트워크 왕복 횟수를 획기적으로 줄여 대용량 처리 성능을 극대화합니다.

---

## # 실습 예제 8 : 단일 컬럼(phone) 수정하기

`UPDATE` 문을 사용하여 특정 조건(`WHERE`)을 만족하는 행의 단일 컬럼 값을 수정합니다. 앞서 실습 예제 4에서 전화번호를 생략하고 등록했던 3번 고객(이보영)의 `phone` 값을 새로 입력·수정합니다.

### 8.1 단일 컬럼 수정 쿼리

```sql
-- 고객 ID가 3번인 이보영 고객의 전화번호 수정
UPDATE customers_ex_dml
SET phone = '010-1000-0003'
WHERE customer_id = 3;

-- 결과 확인
SELECT customer_id, name, phone
FROM customers_ex_dml
WHERE customer_id = 3;
```

### 8.2 구문 요소 설명

- **`UPDATE customers_ex_dml`**: 수정 작업을 수행할 대상 테이블을 지정합니다.
- **`SET phone = '010-1000-0003'`**: 수정할 컬럼명과 새로 변경할 값을 지정합니다.
- **`WHERE customer_id = 3`**: 수정 대상이 될 특정 행을 선별하는 조건식입니다.

> 🎯 **핵심 배움 포인트 (UPDATE의 기본 구조와 WHERE 조건의 필수성)**:
> - `UPDATE`는 특정 조건에 일치하는 대상 행을 선별하여 컬럼 값을 새로운 데이터로 갱신합니다.
> - **`WHERE` 절을 실수로 생략하면 테이블 내 모든 행의 값이 일괄 덮어쓰기**되므로, 쿼리 작성 시 `WHERE` 절을 먼저 작성하고 변경 대상을 검증하는 습관이 필수입니다.

---

## # 실습 예제 9 : 이메일과 전화번호 동시 수정하기

하나의 `UPDATE` 문에서 `SET` 절에 쉼표(`,`)를 사용하여 여러 컬럼의 값을 동시에 수정할 수 있습니다. 앞서 실습 예제 5에서 이메일과 전화번호를 `NULL`로 입력했던 4번 고객(정해인)의 두 정보를 한 번에 갱신합니다.

### 9.1 다중 컬럼 동시 수정 쿼리

```sql
-- 고객 ID가 4번인 정해인 고객의 이메일과 전화번호 동시 수정
UPDATE customers_ex_dml
SET email = 'jhi@mm.com',
    phone = '010-1000-0004'
WHERE customer_id = 4;

-- 결과 확인
SELECT customer_id, name, email, phone
FROM customers_ex_dml
WHERE customer_id = 4;
```

### 9.2 구문 요소 및 수정 내용 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **대상 테이블** | `customers_ex_dml` | 데이터를 수정할 대상 테이블 |
| **수정 컬럼 1** | `email = 'jhi@mm.com'` | 기존 `NULL` 상태였던 이메일 등록 |
| **수정 컬럼 2** | `phone = '010-1000-0004'` | 기존 `NULL` 상태였던 연락처 등록 (쉼표 `,`로 나열) |
| **조건절** | `WHERE customer_id = 4` | 4번 고객(정해인) 단건 레코드만 지정 |

> 🎯 **핵심 배움 포인트 (다중 컬럼 갱신과 원자적(Atomically) 처리)**:
> - `SET 컬럼1 = 값1, 컬럼2 = 값2`와 같이 쉼표(`,`)로 구분하여 여러 컬럼을 1회의 명령으로 동시에 수정합니다.
> - 컬럼마다 별도의 `UPDATE` 문을 개별 실행하는 것에 비해 데이터베이스 I/O와 트랜잭션 오버헤드를 대폭 줄이고 데이터의 정합성을 보장합니다.

---

## # 실습 예제 10-1 : 등록일이 2025년 2월 이전인 고객 조회하기

데이터를 수정(`UPDATE`)하거나 삭제(`DELETE`)하기 전에, 조건절(`WHERE`)에 해당하는 대상 데이터를 미리 `SELECT` 쿼리로 조회하여 변경 대상 행을 사전에 안전하게 확인하고 검증하는 실무 필수 절차입니다. 등록일이 2025년 2월 1일 이전인 고객 데이터를 조회합니다.

### 10-1.1 대상 데이터 사전 조회 쿼리

```sql
-- (1) 2025년 2월 이전 가입 고객 확인
SELECT customer_id, name, registered_date, is_active
FROM customers_ex_dml
WHERE registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD')
ORDER BY registered_date;
```

### 10-1.2 구문 요소 및 조건절 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **조회 컬럼** | `customer_id, name, registered_date, is_active` | 검증에 필요한 고객 식별 정보 및 등록일, 활성 상태 조회 |
| **대상 테이블** | `customers_ex_dml` | 데이터를 조회할 대상 테이블 |
| **조건절** | `WHERE registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD')` | 등록일이 2025년 2월 1일 이전(미만)인 행 필터링 (`TO_DATE`로 명시적 변환) |
| **정렬 기준** | `ORDER BY registered_date` | 등록일 기준 오름차순(과거 순) 정렬 |

> 🎯 **핵심 배움 포인트 (1단계: 수정 대상 행과 범위의 사전 검증)**:
> - DML을 실행하기 전, 수정 대상 행의 건수와 내용이 정확한지 동일한 `WHERE` 절을 갖는 `SELECT` 문으로 먼저 확인합니다.
> - 의도치 않은 행이 수정되는 사고를 원천 차단하기 위해 모든 실무 엔지니어가 준수하는 안전 수칙 제1원칙입니다.

---

## # 실습 예제 10-2 : 등록일이 2025년 2월 이전인 고객 비활성화하기

앞서 실습 예제 10-1에서 사전 조회하여 확인한 조건(`registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD')`)을 그대로 적용하여, 해당 고객들의 계정 활성 상태(`is_active`)를 일괄적으로 비활성(`'N'`)으로 수정(`UPDATE`)합니다.

### 10-2.1 조건부 일괄 수정 쿼리

```sql
-- (2) 2025년 2월 이전 가입 고객을 비활성 상태로 변경
UPDATE customers_ex_dml
SET is_active = 'N'
WHERE registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD');
```

### 10-2.2 구문 요소 및 수정 내용 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **대상 테이블** | `customers_ex_dml` | 데이터를 수정할 대상 테이블 |
| **수정 컬럼** | `SET is_active = 'N'` | 계정 활성 상태를 기존 `'Y'`에서 비활성인 `'N'`으로 변경 |
| **조건절** | `WHERE registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD')` | 가입일이 2025년 2월 1일 이전(미만)인 대상 행만 일괄 변경 |

> 🎯 **핵심 배움 포인트 (2단계: 실행 결과 영향 건수 대조)**:
> - `UPDATE` 실행 직후 DBMS가 반환하는 `Updated Rows`(영향받은 행 수)가 1단계 사전 조회 건수와 정확히 일치하는지 대조합니다.
> - 건수가 일치하지 않으면 의도하지 않은 레코드가 영향을 받았다는 신호이므로 즉시 `ROLLBACK;`을 실행하여 추가 피해를 방지해야 합니다.

---

## # 실습 예제 10-3 : 수정 결과 확인하기

실습 예제 10-2에서 수행한 `UPDATE` 작업이 정상적으로 반영되었는지 확인하기 위해, 동일한 조건(`WHERE registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD')`)으로 데이터를 다시 조회합니다. 대상 고객들의 `is_active` 컬럼 값이 모두 `'N'`으로 정상 변경되었는지 검증합니다.

### 10-3.1 수정 결과 조회 쿼리

```sql
-- (3) 수정 결과 확인
SELECT customer_id, name, registered_date, is_active
FROM customers_ex_dml
WHERE registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD')
ORDER BY customer_id;
```

### 10-3.2 구문 요소 및 확인 포인트 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **조회 컬럼** | `customer_id, name, registered_date, is_active` | 수정 결과 확인을 위한 고객 식별 정보 및 상태 컬럼 조회 |
| **대상 테이블** | `customers_ex_dml` | 데이터가 수정된 대상 테이블 |
| **조건절** | `WHERE registered_date < TO_DATE('2025-02-01', 'YYYY-MM-DD')` | 앞서 수정을 진행한 동일한 대상 범위 필터링 |
| **정렬 기준** | `ORDER BY customer_id` | 고객 ID 기준 오름차순 정렬하여 결과 가독성 확보 |

> 🎯 **핵심 배움 포인트 (3단계: 실무 DML 표준 3단계 라이프사이클 완성)**:
> 1. **사전 검증 (SELECT)**: 수정 대상 행 및 건수 확인 (예제 10-1)
> 2. **작업 수행 (UPDATE/DELETE)**: 작업 실행 및 영향 건수(`Updated Rows`) 일치 대조 (예제 10-2)
> 3. **사후 검증 (SELECT)**: 실제 컬럼 값이 의도한 상태로 정상 변경되었는지 최종 확인 (예제 10-3)
> - 3단계 검증이 모두 통과했을 때만 `COMMIT;`으로 확정하고, 이상 발생 시 즉시 `ROLLBACK;`으로 원복합니다.

---

## # 실습 예제 11-1 : 전체 고객의 활성화 상태 확인하기

조건절(`WHERE`) 없이 테이블 전체 행을 대상으로 고객 ID, 이름, 계정 활성 상태(`is_active`)를 조회합니다. 이전 실습(예제 10)에서 특정 가입일 조건에 의해 비활성화(`'N'`)된 고객들과 여전히 활성(`'Y'`) 상태를 유지하는 고객들의 상태를 한눈에 비교·확인할 수 있습니다.

### 11-1.1 전체 고객 활성화 상태 조회 쿼리

```sql
SELECT customer_id, name, is_active
FROM customers_ex_dml
ORDER BY customer_id;
```

### 11-1.2 구문 요소 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **조회 컬럼** | `customer_id, name, is_active` | 고객 고유 식별 번호, 이름, 계정 활성화 여부 조회 |
| **대상 테이블** | `customers_ex_dml` | 조회 대상 실습 테이블 |
| **정렬 기준** | `ORDER BY customer_id` | 고객 ID 오름차순으로 정렬하여 전체 목록 순차 확인 |

> 🎯 **핵심 배움 포인트 (사이드 이펙트 교차 검증)**:
> - 특정 조건으로 데이터를 변경한 뒤에는, 변경 대상이 아니었던 다른 행들이 의도치 않게 변경되지 않았는지 전체 데이터를 조회하여 부작용(Side-effect) 유무를 교차 검증해야 합니다.

---

## # 실습 예제 11-2 : 전체 고객 비활성화하기 (실습 후 ROLLBACK)

`UPDATE` 문에서 `WHERE` 절을 실수로 생략했을 때 발생하는 치명적인 현상을 직접 확인해보는 실습입니다. 조건절을 작성하지 않으면 테이블에 존재하는 **모든 행의 `is_active` 값이 일괄적으로 `'N'`으로 변경**됩니다.

### 11-2.1 전체 행 일괄 수정 및 확인 쿼리

```sql
-- WHERE 절이 없는 경우 전체 행이 수정됨
UPDATE customers_ex_dml
SET is_active = 'N';

-- 전체 결과 확인
SELECT customer_id, name, is_active FROM customers_ex_dml;
```

### 11-2.2 구문 요소 및 실행 결과 분석

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **대상 테이블** | `customers_ex_dml` | 수정 작업을 수행할 대상 테이블 |
| **수정 컬럼** | `SET is_active = 'N'` | 모든 고객의 활성 상태를 `'N'`으로 변경 |
| **조건절(WHERE)** | *(생략됨)* | 조건을 기술하지 않아 테이블 내 모든 행이 수정 대상이 됨 |
| **결과 확인** | `SELECT ... FROM customers_ex_dml;` | 전체 고객의 `is_active` 컬럼이 모두 `'N'`으로 일괄 변경되었음을 확인 |

> 🎯 **핵심 배움 포인트 (WHERE 절 누락 재난과 ROLLBACK의 구원)**:
> - `WHERE` 절이 누락된 `UPDATE`는 실무에서 서비스 장애와 대규모 데이터 유실을 일으키는 가장 대표적인 실수입니다.
> - 하지만 DML은 디스크에 즉시 영구 저장되지 않고 트랜잭션 버퍼에 머물기 때문에, **`ROLLBACK;` 명령 하나로 언제든 실행 직전 상태로 완벽히 되돌릴 수 있다는 점**이 DBMS 트랜잭션의 핵심 가치입니다.

---

## # 실습 예제 12 : 서브쿼리를 이용하여 수정하기

고정된 리터럴 값을 직접 대입하는 대신, 다른 테이블(`customers`)의 데이터를 `SELECT` 서브쿼리로 조회하여 그 결과값으로 컬럼을 갱신하는 기법입니다. 실습 예제 4에서 이메일이 `NULL` 상태였던 3번 고객(이보영)의 이메일을 원본 `customers` 테이블의 3번 고객 이메일 값으로 조회하여 동기화·수정합니다.

> 💡 **실습 전 확인 (ROLLBACK)**:
> - 만약 앞선 실습 예제 11-2(전체 비활성화)를 실행한 직후라면, 변경 사항을 취소하고 원래 상태로 되돌리기 위해 먼저 `ROLLBACK;`을 실행하세요.

### 12.1 서브쿼리를 이용한 데이터 수정 쿼리

```sql
-- 3번 고객의 이메일을 customers 테이블의 이메일과 동일하게 수정
UPDATE customers_ex_dml
SET email = (
    SELECT email
    FROM customers
    WHERE customer_id = 3
)
WHERE customer_id = 3;

-- 결과 확인
SELECT customer_id, name, email
FROM customers_ex_dml
WHERE customer_id = 3;
```

### 12.2 구문 요소 및 동작 원리 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **대상 테이블** | `UPDATE customers_ex_dml` | 데이터를 수정할 대상 테이블 지정 |
| **서브쿼리 (SET)** | `(SELECT email FROM customers WHERE customer_id = 3)` | 원본 `customers` 테이블에서 3번 고객의 `email` 값을 단일 값(단일 행·단일 컬럼)으로 조회하여 대입 |
| **조건절 (WHERE)** | `WHERE customer_id = 3;` | `customers_ex_dml` 테이블에서 3번 고객의 레코드만 수정 대상으로 지정 |
| **결과 확인** | `SELECT ... WHERE customer_id = 3;` | 3번 고객의 이메일이 원본 테이블 값으로 정상 갱신되었는지 확인 |

> 🎯 **핵심 배움 포인트 (스칼라 서브쿼리와 단일값 보장의 법칙)**:
> - `SET 컬럼 = (서브쿼리)` 구조에서는 대입할 대상 컬럼이 단일 값이므로, 우측 서브쿼리 역시 반드시 **단 하나의 값(1행 1열, Scalar)**만 리턴해야 합니다.
> - 서브쿼리가 2건 이상의 행을 반환하면 `ORA-01427: single-row subquery returns more than one row` 에러가 발생하므로, 서브쿼리의 `WHERE` 절이 고유키(PK) 등으로 유일성을 확실히 보장하는지 확인해야 합니다.

---

## # 실습 예제 13 : 모든 고객의 이메일을 대문자로 변경하기

Oracle 내장 단일 행 함수인 **`UPPER()`**를 `UPDATE` 문의 `SET` 절과 결합하여, 기존에 저장되어 있던 이메일 데이터를 대문자로 변환하여 일괄 갱신합니다. 또한 `WHERE email IS NOT NULL` 조건을 부여하여 이메일이 입력되어 있는 유효한 행만 안전하게 수정합니다.

### 13.1 함수를 이용한 데이터 일괄 수정 쿼리

```sql
-- 모든 고객의 이메일을 대문자로 변경
UPDATE customers_ex_dml
SET email = UPPER(email)
WHERE email IS NOT NULL;

-- 결과 확인
SELECT customer_id, name, email
FROM customers_ex_dml
ORDER BY customer_id;
```

### 13.2 구문 요소 및 함수 동작 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **대상 테이블** | `UPDATE customers_ex_dml` | 데이터를 수정할 대상 테이블 지정 |
| **수정 컬럼 및 함수** | `SET email = UPPER(email)` | 각 행의 기존 `email` 값을 읽어 `UPPER()` 함수로 영문 대문자로 변환 후 다시 `email` 컬럼에 대입 |
| **조건절** | `WHERE email IS NOT NULL;` | 이메일 값이 존재하는 행만 선별하여 불필요한 `NULL` 데이터 갱신 방지 |
| **결과 확인** | `SELECT ... ORDER BY customer_id;` | 고객 ID 순서대로 대문자 변환 결과 확인 |

> 🎯 **핵심 배움 포인트 (데이터 정제 패턴과 불필요한 I/O 방지)**:
> - `SET 컬럼 = 함수(컬럼)` 패턴은 대소문자 규격화(`UPPER`/`LOWER`), 공백 제거(`TRIM`), 문자열 치환(`REPLACE`) 등 실무 데이터 정제(Data Cleansing) 작업의 표준 기법입니다.
> - `WHERE email IS NOT NULL`처럼 실제 변경이 필요한 유효 행만 한정하여 갱신함으로써 불필요한 디스크 I/O와 Undo/Redo 로그 생성을 방지하는 성능 최적화 관점을 습득합니다.

---

## # 실습 예제 14 : 특정 고객 1명 삭제하기

`DELETE` 문을 사용하여 조건절(`WHERE`)에 지정된 조건에 부합하는 데이터 행을 삭제합니다. 고객 ID가 3번인 고객(이보영)의 레코드를 `customers_ex_dml` 테이블에서 단건 삭제하고 결과를 확인합니다.

### 14.1 특정 고객 단건 삭제 쿼리

```sql
-- 고객 ID가 3번인 고객 삭제
DELETE FROM customers_ex_dml
WHERE customer_id = 3;

-- 결과 확인
SELECT customer_id, name
FROM customers_ex_dml
WHERE customer_id = 3;
```

### 14.2 구문 요소 및 동작 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **명령어 및 대상 테이블** | `DELETE FROM customers_ex_dml` | 데이터를 삭제할 대상 테이블 지정 (`FROM` 키워드는 생략 가능하나 명시 권장) |
| **조건절** | `WHERE customer_id = 3;` | 삭제 대상 행을 식별하는 조건 (`customer_id`가 3인 단일 행) |
| **결과 확인** | `SELECT ... WHERE customer_id = 3;` | 삭제 후 동일 조건으로 조회 시 조회 결과가 0건(행 없음)인지 확인 |

> 🎯 **핵심 배움 포인트 (행 단위 삭제와 물리 삭제 vs 논리 삭제)**:
> - `DELETE`는 컬럼을 비우는 것이 아니라 **행(Row) 자체를 완전히 제거**합니다. (특정 컬럼만 비우려면 `UPDATE ... SET 컬럼 = NULL` 사용)
> - **물리 삭제(Hard Delete) vs 논리 삭제(Soft Delete)**: 현대 실무에서는 데이터 복구 및 법적 이력 보존을 위해 레코드를 완전히 지우기보다, 예제 10-2처럼 `is_active = 'N'`으로 상태 플래그만 변경하는 **논리 삭제(Soft Delete)** 방식을 주로 채택합니다.

---

## # 실습 예제 15 : 비활성 고객(N) 모두 삭제하기

특정 단건 레코드가 아닌, 조건절(`WHERE is_active = 'N'`)에 일치하는 복수 개의 데이터 행을 일괄 삭제하는 다중 행 `DELETE` 실습입니다. 비활성화된 고객 레코드들을 테이블에서 한 번에 제거하고, 조회 결과를 통해 활성(`'Y'`) 상태인 고객만 남아있는지 확인합니다.

### 15.1 조건부 다중 행 삭제 쿼리

```sql
-- 비활성화된 고객 삭제
DELETE FROM customers_ex_dml
WHERE is_active = 'N';

-- 결과 확인
SELECT customer_id, name, is_active
FROM customers_ex_dml
ORDER BY customer_id;
```

### 15.2 구문 요소 및 동작 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **대상 테이블** | `DELETE FROM customers_ex_dml` | 데이터를 삭제할 대상 테이블 지정 |
| **조건절** | `WHERE is_active = 'N';` | 계정 상태가 비활성(`'N'`)인 모든 행을 삭제 대상으로 지정 |
| **결과 확인** | `SELECT ... ORDER BY customer_id;` | 삭제 후 남아있는 데이터를 확인하여 비활성 고객이 모두 정상 제거되었는지 검증 |

> 🎯 **핵심 배움 포인트 (조건부 다중 행 삭제와 대조 검증)**:
> - 조건에 일치하는 다수의 레코드를 한 번에 제거할 때도 반드시 사전에 `SELECT`로 삭제될 건수를 파악한 후 실행해야 합니다.
> - 실행 직후 반환된 `Deleted Rows` 수치가 사전 조회 집계와 정확히 일치하는지 대조하는 것이 다중 행 삭제의 핵심 안전 절차입니다.

---

## # 실습 예제 16 : 서브쿼리를 이용한 특정 고객 삭제하기

`DELETE` 문의 `WHERE` 조건절에 `SELECT` 서브쿼리를 사용하여, 특정 조건(예: 이메일이 `'JHI@MM.COM'`)을 만족하는 대상의 식별 번호(`customer_id`)를 먼저 동적으로 조회한 뒤 해당 행을 삭제하는 방식입니다. 앞서 실습 예제 13에서 대문자로 변환된 4번 고객(정해인)의 레코드를 서브쿼리로 조회하여 삭제합니다.

### 16.1 서브쿼리를 이용한 데이터 삭제 쿼리

```sql
-- 이메일이 'JHI@MM.COM'인 고객의 고객번호를 서브쿼리로 조회해 삭제
DELETE FROM customers_ex_dml
WHERE customer_id = (
    SELECT customer_id
    FROM customers_ex_dml
    WHERE email = 'JHI@MM.COM'
);

-- 결과 확인
SELECT customer_id, name, email
FROM customers_ex_dml
WHERE email = 'JHI@MM.COM';
```

### 16.2 구문 요소 및 동작 원리 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **대상 테이블** | `DELETE FROM customers_ex_dml` | 데이터를 삭제할 대상 테이블 지정 |
| **서브쿼리 (WHERE)** | `(SELECT customer_id FROM customers_ex_dml WHERE email = 'JHI@MM.COM')` | 이메일이 `'JHI@MM.COM'`인 고객의 `customer_id`를 조회 (단일 행 반환) |
| **조건절 연산** | `WHERE customer_id = (서브쿼리);` | 서브쿼리가 반환한 단일 고객 ID와 일치하는 레코드를 삭제 대상으로 지정 |
| **결과 확인** | `SELECT ... WHERE email = 'JHI@MM.COM';` | 삭제 후 해당 이메일로 재조회하여 0건(행 없음)인지 최종 검증 |

> 🎯 **핵심 배움 포인트 (서브쿼리 삭제 시 단일행(=) vs 다중행(IN) 연산자)**:
> - 서브쿼리의 결과가 단 1건임이 100% 보장될 때만 단일행 비교 연산자(`=`)를 사용할 수 있습니다.
> - 조건에 부합하는 결과가 2건 이상 리턴될 가능성이 조금이라도 있다면 반드시 **`IN` 연산자(`WHERE customer_id IN (SELECT ...)`)**를 사용해야 `ORA-01427` 오류를 방지할 수 있습니다. 쿼리 작성 시 결과 집합의 카디널리티(건수)를 항상 예측해야 합니다.

---

## # 실습 예제 17 : 전체 고객 데이터 삭제하기 (실습 후 ROLLBACK)

`DELETE` 문에서 조건절(`WHERE`)을 작성하지 않았을 때 테이블의 모든 데이터 행이 일괄 삭제되는 치명적인 위험을 직접 확인하는 실습입니다. 실습 시작 전 현재 상태를 보존하기 위해 먼저 `COMMIT;`을 실행한 후 전체 삭제를 수행하고 남은 행 수가 `0`이 되는 것을 확인합니다.

### 17.1 전체 데이터 삭제 및 남은 행 수 확인 쿼리

```sql
COMMIT;
-- WHERE 절이 없는 경우 전체 데이터가 삭제됨
DELETE FROM customers_ex_dml;

-- 전체 행 수 확인
SELECT COUNT(*) AS "남은행수" FROM customers_ex_dml;
```

### 17.2 구문 요소 및 실행 결과 분석

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **기준 시점 저장** | `COMMIT;` | 전체 삭제 실행 직전의 데이터를 영구 저장하여 이후 `ROLLBACK` 시 복구할 기준점 생성 |
| **명령어 및 대상 테이블** | `DELETE FROM customers_ex_dml;` | `WHERE` 조건절 없이 대상 테이블의 모든 행(전체 레코드)을 일괄 삭제 |
| **행 수 확인** | `SELECT COUNT(*) AS "남은행수" ...` | 삭제 후 잔여 행 수를 집계하여 결과가 `0`건인지 확인 |

> 🎯 **핵심 배움 포인트 (DELETE와 TRUNCATE의 결정적 차이)**:
> - **`DELETE`(DML)**: 행 단위로 지우며 이전 상태를 **Undo 세그먼트**에 모두 기록하므로 실행 속도는 상대적으로 느리지만 `ROLLBACK;`으로 완벽히 복원할 수 있습니다.
> - **`TRUNCATE`(DDL)**: 저장 공간(High Water Mark)을 통째로 즉시 초기화하며, 실행 즉시 자동 커밋(Auto-commit)되므로 `ROLLBACK`이 불가능합니다.
> - 본 실습 후 다음 단계에서 반드시 `ROLLBACK;`을 실행하여 트랜잭션 복구 원리를 체감해야 합니다.

---

## # 실습 예제 18 : 실습 후 ROLLBACK으로 복원 확인하기

실습 예제 17에서 전체 삭제했던 변경 작업을 `ROLLBACK;` 명령어를 통해 즉시 취소하고, 바로 직전 `COMMIT;` 완료 시점으로 데이터를 복구합니다. 복구 실행 후 `SELECT COUNT(*)`를 통해 삭제되었던 데이터 행들이 온전히 복원되었는지 최종 확인합니다.

### 18.1 작업 취소 및 복원 행 수 확인 쿼리

```sql
-- 변경 내용 취소 (ROLLBACK)
ROLLBACK;

-- 복원 확인
SELECT COUNT(*) AS "복원후행수" FROM customers_ex_dml;
```

### 18.2 구문 요소 및 트랜잭션 복구 원리 설명

| 구분 | 지정 내용 | 설명 |
| :--- | :--- | :--- |
| **작업 취소 명령** | `ROLLBACK;` | 현재 트랜잭션에서 수행된 모든 DML 변경 내역(`DELETE`)을 취소하고 직전 `COMMIT` 시점으로 롤백 |
| **복원 결과 확인** | `SELECT COUNT(*) AS "복원후행수" ...` | 롤백 후 전체 행 수를 조회하여 삭제 전 데이터 건수로 온전히 복원되었는지 확인 |

> 🎯 **핵심 배움 포인트 (트랜잭션 ACID와 Undo 세그먼트의 역할)**:
> - **Undo 세그먼트의 동작**: 오라클은 DML 실행 시 변경 이전 원본 데이터를 Undo 공간에 보관하므로, `ROLLBACK;` 명령 한 줄로 원상 복구가 가능합니다.
> - **트랜잭션 격리성(Read Consistency)**: 내가 DML을 실행하고 `COMMIT`하기 전까지는 **내 세션에서만 변경 사항이 보이고, 다른 세션 사용자에게는 변경 전 원본 데이터가 안전하게 조회**됩니다.
> - 한 번 `COMMIT`되거나 DDL이 실행되면 롤백이 불가능하므로, 트랜잭션 확정 시점을 항상 신중하게 결정해야 합니다.
