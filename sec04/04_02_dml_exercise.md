# Oracle DML(Data Manipulation Language) 기초 연습문제

본 문서는 [04_01_dml_guide.md](file:///c:/Users/dandycode/Documents/GitHub/kor-dbms-class/sec04/04_01_dml_guide.md)에서 학습한 데이터 조작어(INSERT, UPDATE, DELETE)의 핵심 문법, 데이터 무결성 규칙, NULL 및 날짜 데이터 처리 방법 등을 점검하기 위한 연습문제입니다.

각 문제를 풀고 하단의 정답 및 해설을 펼쳐 정답 여부와 상세 원리를 확인해 보세요.

---

## Q1. INSERT 문 작성 시 필수 일치 항목

**다음 중 `INSERT` 문 작성 시 반드시 일치해야 하는 것은 무엇입니까?**

① 테이블 이름과 데이터 타입  
② 컬럼의 개수와 값의 개수  
③ 제약조건의 개수와 입력 행 수  
④ 입력되는 데이터의 길이  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② 컬럼의 개수와 값의 개수**

### **💡 해설**
- `INSERT INTO 테이블명 (컬럼1, 컬럼2, ...) VALUES (값1, 값2, ...);` 구문에서는 **지정한 컬럼의 개수**와 `VALUES` 절에 나열한 **값의 개수가 1:1로 정확하게 일치**해야 합니다.
- 컬럼명을 생략하는 전체 컬럼 입력 방식(`INSERT INTO 테이블명 VALUES (...)`)에서도 **테이블에 정의된 컬럼의 전체 개수와 입력하는 값의 개수가 반드시 일치**해야 합니다.
- 만약 컬럼 수와 값의 수가 맞지 않으면 Oracle 데이터베이스는 아래와 같은 오류를 발생시킵니다:
  - **입력된 값의 수가 컬럼 수보다 적을 때**: `ORA-00947: not enough values` (값의 수가 충분하지 않습니다)
  - **입력된 값의 수가 컬럼 수보다 많을 때**: `ORA-00913: too many values` (값의 수가 너무 많습니다)

### **❌ 오답 분석**
- **① 테이블 이름과 데이터 타입**: 테이블 이름은 객체 식별자이며 데이터 타입과 일치 여부를 비교하는 대상이 아닙니다.
- **③ 제약조건의 개수와 입력 행 수**: 제약조건은 데이터 유효성을 검사하는 규칙(PK, NOT NULL 등)일 뿐, 입력할 행의 개수와 일치해야 하는 대상이 아닙니다.
- **④ 입력되는 데이터의 길이**: 컬럼에 정의된 최대 길이(예: `VARCHAR2(100)`) 이하의 데이터라면 길이가 달라도 정상 입력됩니다. 반드시 컬럼 크기와 데이터의 길이가 정확히 같아야 하는 것은 아닙니다.

</details>

---

## Q2. NULL 입력 및 처리 특성

**`NULL` 입력에 대한 설명으로 옳은 것은?**

① NULL은 숫자 0과 같다.  
② Oracle에서는 빈 문자열('')도 NULL로 인식된다.  
③ INSERT 시 값을 생략하면 자동으로 공백(' ')이 입력된다.  
④ NULL은 문자형 데이터에서만 사용할 수 있다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② Oracle에서는 빈 문자열('')도 NULL로 인식된다.**

### **💡 해설**
- **Oracle Database의 대표적인 특징**: Oracle에서는 길이가 0인 빈 문자열(`''`)을 입력할 경우 내부적으로 **`NULL`**로 간주하여 처리합니다.
  ```sql
  -- Oracle에서는 아래 두 문장이 동일하게 email에 NULL을 저장합니다.
  INSERT INTO customers_ex_dml (customer_id, name, email) VALUES (10, '테스트', NULL);
  INSERT INTO customers_ex_dml (customer_id, name, email) VALUES (10, '테스트', '');
  ```
  *(참고: 다른 표준 RDBMS인 PostgreSQL, MySQL 등에서는 빈 문자열 `''`과 `NULL`을 서로 다른 데이터로 구분하지만, Oracle은 빈 문자열을 NULL로 취급합니다.)*

### **❌ 오답 분석**
- **① NULL은 숫자 0과 같다**: `NULL`은 "알 수 없는 값(Unknown)" 또는 "값이 존재하지 않는 상태(Missing/Undefined)"를 의미합니다. `0`은 실제 존재하는 유효한 숫자 데이터이므로 `NULL`과 전혀 다릅니다.
- **③ INSERT 시 값을 생략하면 자동으로 공백(' ')이 입력된다**: 컬럼 목록에서 특정 컬럼을 생략하면 `DEFAULT` 값이 없는 한 기본적으로 **`NULL`**이 대입됩니다. 스페이스 1칸짜리 공백(`' '`)은 실제 문자 데이터이므로 자동으로 입력되지 않습니다.
- **④ NULL은 문자형 데이터에서만 사용할 수 있다**: `NULL`은 숫자형(`NUMBER`), 날짜형(`DATE`), 문자형(`VARCHAR2`, `CHAR`) 등 **모든 데이터 타입 컬럼에서 공통으로 사용**될 수 있습니다.

</details>

---

## Q3. 날짜 데이터 입력 함수 (단답형 / 빈칸 완성)

**아래 쿼리문은 고객 테이블에 날짜 데이터를 저장하는 예입니다. 빈칸에 들어갈 알맞은 함수를 작성하세요.**

```sql
01  INSERT INTO customers_ex_dml
02      (customer_id, name, registered_date)
03  VALUES
04      (11, '한소희', ____('2025-01-10', 'YYYY-MM-DD'));
```

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: `TO_DATE`**

### **💡 완성된 쿼리**
```sql
INSERT INTO customers_ex_dml
    (customer_id, name, registered_date)
VALUES
    (11, '한소희', TO_DATE('2025-01-10', 'YYYY-MM-DD'));
```

### **💡 해설**
- Oracle에서 특정 형식의 문자열 리터럴을 표준 날짜 타입(`DATE`)으로 명시적 형변환할 때 사용하는 내장 함수는 **`TO_DATE()`**입니다.
- **문법**: `TO_DATE('날짜문자열', '형식포맷')`
  - `'2025-01-10'`: 변환할 날짜 문자열
  - `'YYYY-MM-DD'`: 연(4자리)-월(2자리)-일(2자리) 포맷 지정
- **실무 권장 이유**:
  - 문자열 `'2025-01-10'`을 그대로 대입해도 Oracle이 묵시적 형변환을 시도하지만, 클라이언트나 DB 서버의 NLS 설정(`NLS_DATE_FORMAT`)에 따라 오류(`ORA-01861: 리터럴이 형식 문자열과 일치하지 않음`)가 발생할 수 있습니다.
  - 따라서 환경 설정에 구애받지 않고 항상 안전하고 정확하게 날짜를 입력하기 위해 **`TO_DATE()` 함수 사용이 강력히 권장**됩니다.

</details>

---

## Q4. UPDATE 문과 WHERE 절 생략 시 동작

**`UPDATE` 문에서 `WHERE` 절을 생략하고 실행하면 어떤 결과가 발생합니까?**

① 조건에 맞는 일부 행만 수정된다.  
② 모든 행이 수정된다.  
③ 아무 행도 수정되지 않는다.  
④ 오류가 발생한다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② 모든 행이 수정된다.**

### **💡 해설**
- `UPDATE` 문에서 `WHERE` 절은 수정할 대상 행을 선별하는 필터 역할을 수행합니다.
- 만약 **`WHERE` 절을 생략하면 테이블에 존재하는 모든 행(전체 레코드)의 지정 컬럼 값이 일괄 변경**됩니다.
  ```sql
  -- 주의: 아래 쿼리는 테이블 내 모든 고객의 전화번호를 '010-0000-0000'으로 일괄 변경합니다!
  UPDATE customers_ex_dml
  SET phone = '010-0000-0000';
  ```
- 이는 실무에서 대형 데이터 사고(데이터 유실 및 서비스 장애)를 유발하는 가장 대표적인 실수입니다.

### **⚠️ 실무 안전 수칙 (Best Practice)**
1. **UPDATE 실행 전 SELECT로 검증**: 작성한 `WHERE` 조건절을 먼저 `SELECT` 문으로 실행하여 의도한 행들만 조회되는지 확인합니다.
2. **트랜잭션 통제**: AUTO COMMIT 설정을 끄고 실행한 후, 변경된 건수(`Updated Rows`)와 결과를 확인한 뒤 `COMMIT` 또는 잘못되었을 경우 즉시 `ROLLBACK`을 실행합니다.

</details>

---

## Q5. 서브쿼리를 활용한 특정 행 삭제 (DELETE)

**다음 중 이메일이 `'JHI@MM.COM'`인 고객을 삭제하기 위한 올바른 SQL 문은 어느 것입니까?**

```sql
① DELETE FROM customers_ex_dml
   WHERE name = (
       SELECT name FROM customers_ex_dml
       WHERE email = 'JHI@MM.COM'
   );

② DELETE FROM customers_ex_dml
   WHERE customer_id = (
       SELECT customer_id FROM customers_ex_dml
       WHERE email = 'JHI@MM.COM'
   );

③ DELETE FROM customers_ex_dml
   WHERE email IN (
       SELECT email FROM customers_ex_dml
   );

④ DELETE FROM customers_ex_dml;
```

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ②**
```sql
DELETE FROM customers_ex_dml
WHERE customer_id = (
    SELECT customer_id FROM customers_ex_dml
    WHERE email = 'JHI@MM.COM'
);
```

### **💡 해설**
- 특정 레코드를 서브쿼리를 이용해 식별하여 삭제할 때, 중복이 발생할 수 있는 일반 컬럼(`name` 등) 대신 **고유 식별자(기본키, PK)인 `customer_id`를 조건으로 지정하는 것이 가장 안전하고 올바른 방법**입니다.
- 서브쿼리 `(SELECT customer_id FROM customers_ex_dml WHERE email = 'JHI@MM.COM')`가 해당 이메일을 가진 고객의 고유 ID를 정확히 조회하고, 메인 쿼리는 그 ID를 가진 단일 고객 행만 안전하게 삭제합니다.

### **❌ 오답 분석**
- **①**: 고객 이름(`name`)은 동명이인이 존재할 수 있습니다. 만약 동일한 이름을 가진 다른 고객이 존재한다면, 의도치 않게 다른 고객의 데이터까지 함께 삭제되는 부작용(Side Effect)이 발생할 수 있습니다.
- **③**: 서브쿼리가 테이블 내의 전체 `email` 목록을 반환하므로, 사실상 이메일이 등록된 모든 고객 데이터가 삭제됩니다.
- **④**: `WHERE` 절이 완전히 누락되어 테이블 내의 **모든 데이터(전체 행)가 삭제**됩니다.

</details>

---

## Q6. DELETE와 TRUNCATE의 차이점

**다음 중 `DELETE`와 `TRUNCATE`의 차이로 올바른 것은 무엇입니까?**

① 둘 다 ROLLBACK이 가능하다.  
② TRUNCATE는 실행 즉시 COMMIT이 수행된다.  
③ DELETE는 WHERE 절을 사용할 수 없다.  
④ TRUNCATE는 DML 명령어이다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② TRUNCATE는 실행 즉시 COMMIT이 수행된다.**

### **💡 해설**
- **`TRUNCATE`는 DDL(데이터 정의어)** 명령어로, 실행 즉시 **암시적 커밋(Auto Commit/Implicit Commit)**이 수행되어 트랜잭션이 종료됩니다. 따라서 `ROLLBACK`으로 데이터를 복구할 수 없습니다.
- 반면 **`DELETE`는 DML(데이터 조작어)** 명령어이므로 트랜잭션 로그(Undo)를 기록하여 실행 후에도 `ROLLBACK`을 통해 이전 상태로 복구할 수 있습니다.

### **❌ 오답 분석**
- **① 둘 다 ROLLBACK이 가능하다**: `DELETE`는 `ROLLBACK`이 가능하지만, `TRUNCATE`는 DDL이므로 즉시 커밋되어 `ROLLBACK`이 불가능합니다.
- **③ DELETE는 WHERE 절을 사용할 수 없다**: `DELETE`는 `WHERE` 절을 사용하여 원하는 특정 조건의 행만 선택적으로 삭제할 수 있습니다. 반면 `TRUNCATE`는 `WHERE` 절을 지원하지 않아 전체 데이터만 초기화합니다.
- **④ TRUNCATE는 DML 명령어이다**: `TRUNCATE`는 테이블의 저장 공간(세그먼트)을 반환하고 초기 상태로 되돌리는 **DDL(Data Definition Language)** 명령어입니다.

### **📊 DELETE vs TRUNCATE vs DROP 완벽 비교표**

| 비교 항목 | DELETE | TRUNCATE | DROP |
| :--- | :--- | :--- | :--- |
| **명령어 분류** | **DML** (데이터 조작어) | **DDL** (데이터 정의어) | **DDL** (데이터 정의어) |
| **삭제 대상** | 데이터 행 (일부 또는 전체) | 데이터 전체 (테이블 초기화) | 테이블 객체 및 데이터 전체 |
| **WHERE 절 지원** | **O (특정 행 선택 삭제 가능)** | **X (전체 데이터만 삭제)** | **X (테이블 자체 삭제)** |
| **ROLLBACK 여부** | **가능 (트랜잭션 롤백 지원)** | **불가능 (자동 COMMIT)** | **불가능 (자동 COMMIT)** |
| **테이블 구조 보존** | 테이블 구조 그대로 유지 | 테이블 구조 그대로 유지 (공간 반환) | 테이블 구조 자체 완전 삭제 |
| **처리 속도** | 행 단위 로깅으로 상대적 느림 | 세그먼트 해제 방식으로 매우 빠름 | 메타데이터 해제로 매우 빠름 |

</details>

---

## 📋 DML 핵심 요약 노트

| 구분 | 주요 문법 및 키워드 | 핵심 주의사항 |
| :--- | :--- | :--- |
| **INSERT** | `INSERT INTO 테이블명 (컬럼들) VALUES (값들);` | 컬럼 목록의 개수와 `VALUES`의 값 개수는 반드시 1:1로 일치해야 함 (`ORA-00947`, `ORA-00913` 방지) |
| **NULL 처리** | `NULL`, 생략, Oracle의 `''` | `NULL`은 0이나 공백과 다름. Oracle에서는 빈 문자열(`''`) 입력 시 `NULL`로 처리됨 |
| **날짜 입력** | `TO_DATE('2025-01-10', 'YYYY-MM-DD')` | NLS 포맷 오류를 방지하기 위해 문자열 직입력 대신 명시적 `TO_DATE()` 함수 사용 권장 |
| **UPDATE** | `UPDATE 테이블명 SET 컬럼=값 WHERE 조건;` | `WHERE` 절 생략 시 **테이블 전체 행이 수정**되므로 실행 전 조건 확인 필수 |
| **DELETE** | `DELETE FROM 테이블명 WHERE 조건;` | `WHERE` 절 생략 시 **테이블 전체 데이터가 삭제**됨. DML이므로 `ROLLBACK` 가능 |
| **TRUNCATE** | `TRUNCATE TABLE 테이블명;` | **DDL** 명령어이므로 실행 즉시 자동 `COMMIT`되어 `ROLLBACK` 불가 |

