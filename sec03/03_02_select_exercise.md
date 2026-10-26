# Oracle SQL 데이터 조회(SELECT) 기초 연습문제

본 문서는 [03_01_select_guide.md](./03_01_select_guide.md)에서 학습한 SQL 데이터 조회(`SELECT`), 특정 컬럼 프로젝션, 별칭(ALIAS), `WHERE` 조건절, 산술 및 비교/논리 연산자, `NULL` 처리(`IS NULL`, `IS NOT NULL`, `NVL`), 범위(`BETWEEN`) 및 다중 목록(`IN`) 필터링의 핵심 원리를 점검하기 위한 연습문제입니다.

각 문제를 먼저 풀어본 후, 하단의 **정답 및 해설**을 펼쳐 올바른 해설과 오답 분석을 확인해 보세요.

---

## Q1. SELECT 문의 기본 문법 구조

**다음 중 올바른 SELECT 문으로만 구성된 것을 고르시오.**

① `SELECT name, email FROM customers WHERE is_active = 'Y';`  
② `SELECT * customers WHERE name = '홍길동';`  
③ `SELECT name, phone FROM customers IS_ACTIVE = 'Y';`  
④ `FROM customers SELECT name, phone WHERE is_active = 'N';`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ① `SELECT name, email FROM customers WHERE is_active = 'Y';`**

### **💡 해설**
- SQL `SELECT` 문의 표준 구문 순서는 **`SELECT 컬럼목록 FROM 테이블명 WHERE 조건절;`** 입니다.
- 문자열 리터럴 값 `'Y'`는 작은따옴표(`' '`)로 올바르게 감싸져 있으며, 콤마(`,`)를 통한 다중 컬럼 지정도 정확합니다.

### **❌ 오답 분석**
- **② `SELECT * customers WHERE ...`**: 테이블을 지정하는 필수 키워드인 **`FROM`이 누락**되었습니다. (`SELECT * FROM customers ...`)
- **③ `SELECT ... FROM customers IS_ACTIVE = 'Y';`**: 조건식을 시작하는 **`WHERE` 키워드가 누락**되었습니다.
- **④ `FROM customers SELECT ...`**: SQL 문장은 반드시 `SELECT` 절로 시작해야 합니다. `FROM` 절을 맨 앞에 배치할 수 없습니다.

</details>

---

## Q2. 날짜 범위 조건 검색 (BETWEEN과 TO_DATE)

**다음 중 2024년 1월 1일부터 2024년 1월 31일까지 주문된 건을 조회하는 올바른 SQL문은?**

① `SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';`  
② `SELECT * FROM orders WHERE order_date BETWEEN TO_DATE('2024-01-01') AND TO_DATE('2024-01-31');`  
③ `SELECT * FROM orders WHERE order_date BETWEEN TO_DATE('2024-01-01', 'YYYY-MM-DD') AND TO_DATE('2024-01-31', 'YYYY-MM-DD');`  
④ `SELECT * FROM orders WHERE TO_DATE(order_date) BETWEEN '2024-01-01' AND '2024-01-31';`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ③ `SELECT * FROM orders WHERE order_date BETWEEN TO_DATE('2024-01-01', 'YYYY-MM-DD') AND TO_DATE('2024-01-31', 'YYYY-MM-DD');`**

### **💡 해설**
- Oracle Database에서 날짜 데이터를 비교할 때는 데이터베이스 세션 설정(`NLS_DATE_FORMAT`)에 따른 오류를 방지하기 위해 **`TO_DATE('문자열', '날짜포맷')` 함수를 명시적으로 사용**하는 것이 표준 원칙입니다.
- 포맷 문자열 `'YYYY-MM-DD'`를 명시하여 2024년 1월 1일부터 1월 31일까지의 범위를 안전하고 정확하게 지정했습니다.

### **❌ 오답 분석**
- **①**: 문자열(`'2024-01-01'`)을 그대로 전달하는 묵시적 형변환 방식입니다. 세션 환경에 따라 `ORA-01861: literal does not match format string` 오류가 발생할 위험이 있습니다.
- **②**: `TO_DATE` 함수에 날짜 포맷 인자(`'YYYY-MM-DD'`)를 생략하면 기본 NLS 설정에 의존하게 되어 환경에 따라 실패할 수 있습니다.
- **④**: `order_date`가 이미 `DATE` 타입 컬럼임에도 불구하고 컬럼 좌변을 `TO_DATE`로 감싸는 잘못된 가공(좌변 가공)을 수행하여 인덱스를 활용하지 못하게 만듭니다.

</details>

---

## Q3. 특정 목록 필터링 (IN 연산자)

**다음 SQL의 실행 결과로 'Gold', 'Platinum' 등급의 고객만 조회하려 한다. 올바른 SQL문은?**  
*(고객(customers) 테이블에 grade 컬럼이 존재한다고 가정하고 문제를 풀이합니다.)*

① `SELECT * FROM customers WHERE grade = 'Gold' AND 'Platinum';`  
② `SELECT * FROM customers WHERE grade IN ('Gold', 'Platinum');`  
③ `SELECT * FROM customers WHERE grade = IN ('Gold', 'Platinum');`  
④ `SELECT * FROM customers WHERE grade BETWEEN 'Gold' AND 'Platinum';`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `SELECT * FROM customers WHERE grade IN ('Gold', 'Platinum');`**

### **💡 해설**
- 하나의 컬럼에 대해 여러 후보 값 중 하나라도 만족하는 데이터를 조회할 때는 **`IN ('값1', '값2', ...)` 연산자**를 사용합니다.
- `WHERE grade IN ('Gold', 'Platinum')`은 `WHERE grade = 'Gold' OR grade = 'Platinum'`과 논리적으로 완벽히 동일하며 가독성이 훨씬 우수합니다.

### **❌ 오답 분석**
- **①**: SQL에서는 `grade = 'Gold' AND 'Platinum'`과 같은 축약 문법을 지원하지 않습니다. 또한 한 행의 `grade` 컬럼이 'Gold'이면서 동시에 'Platinum'일 수는 없으므로 만약 `grade = 'Gold' AND grade = 'Platinum'`으로 쓴다면 0건이 반환됩니다.
- **③**: 등호(`=`)와 `IN` 연산자를 연속해서 함께 사용할 수 없습니다 (`= IN` 문법 오류).
- **④**: `BETWEEN 'Gold' AND 'Platinum'`은 알파벳 사전순 범위 조회를 의미하므로, 두 등급 외에도 'G'와 'P' 사이로 시작하는 모든 등급 문자열이 포함될 수 있어 의도에 맞지 않습니다.

</details>

---

## Q4. NULL 값 검색 (IS NULL)

**고객 테이블에서 이메일이 입력되지 않은 고객을 조회하는 SQL문은?**

① `SELECT * FROM customers WHERE email = NULL;`  
② `SELECT * FROM customers WHERE email IS NULL;`  
③ `SELECT * FROM customers WHERE email != NULL;`  
④ `SELECT * FROM customers WHERE email NOT NULL;`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `SELECT * FROM customers WHERE email IS NULL;`**

### **💡 해설**
- SQL에서 `NULL`은 값이 존재하지 않거나 알 수 없는(Unknown) 상태를 나타냅니다.
- `NULL`은 데이터 값이 아니므로 일반 등호(`=`)나 부등호(`!=`)로 비교할 수 없으며, **반드시 `IS NULL` 연산자**를 사용해야만 정상적으로 필터링됩니다.

### **❌ 오답 분석**
- **① `WHERE email = NULL`**: 문법 오류는 발생하지 않지만 평가 결과가 항상 `UNKNOWN(거짓)`이 되어 아무런 데이터도 조회되지 않습니다.
- **③ `WHERE email != NULL`**: 마찬가지로 항상 `UNKNOWN`으로 평가되어 원하는 결과를 얻을 수 없습니다.
- **④ `WHERE email NOT NULL`**: `NOT NULL`은 DDL에서 테이블 컬럼을 생성할 때 부여하는 제약조건 키워드이며, WHERE 절에서는 `IS NOT NULL`로 작성해야 합니다.

</details>

---

## Q5. 컬럼 별칭(ALIAS)과 한글 식별자

**고객 테이블에서 name 컬럼을 "고객명"으로 출력하려 한다. 올바른 SELECT 문은?**

① `SELECT name 고객명 FROM customers;`  
② `SELECT name AS "고객명" FROM customers;`  
③ `SELECT name AS 고객명 FROM customers;`  
④ `SELECT "고객명" = name FROM customers;`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `SELECT name AS "고객명" FROM customers;`**

### **💡 해설**
- 컬럼에 별칭을 부여할 때는 **`컬럼명 AS "별칭"`** 형식을 사용합니다.
- 특히 별칭이 **한글**, 공백, 특수문자를 포함하거나 영문 대소문자를 구분해야 할 때는 **반드시 큰따옴표(`""`)**로 감싸주어야 Oracle의 표준 식별자 규칙에 완전히 부합합니다.

### **❌ 오답 분석**
- **①**: `AS` 키워드가 누락되었고 큰따옴표도 감싸지 않아 가독성과 식별자 명확성이 떨어집니다.
- **③**: `AS`는 사용했으나 한글 식별자에 큰따옴표를 씌우지 않았습니다.
- **④**: `SELECT "고객명" = name`은 프로그래밍 언어의 변수 할당 방식이며 SQL 표준 문법이 아닙니다.

</details>

---

## Q6. IN 연산자와 OR 연산자의 동치성

**다음 두 SQL 문장의 차이를 올바르게 설명한 것은?**

```sql
A. SELECT * FROM reviews WHERE rating IN (1, 5);
B. SELECT * FROM reviews WHERE rating = 1 OR rating = 5;
```

① 두 쿼리는 실행 결과가 다르다.  
② A는 NULL도 포함하지만 B는 그렇지 않다.  
③ 두 쿼리는 실행 결과가 동일하다.  
④ A는 rating이 1점만 조회된다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ③ 두 쿼리는 실행 결과가 동일하다.**

### **💡 해설**
- **`IN` 연산자는 다중 `OR` 조건의 문법적 설탕(Syntactic Sugar)**입니다.
- `WHERE rating IN (1, 5)`는 내부적으로 `WHERE rating = 1 OR rating = 5`와 동일하게 해석되므로 **두 쿼리의 결과 셋은 100% 동일**합니다.

### **❌ 오답 분석**
- **①**: 논리식이 동일하므로 결과 집합(조회 건수와 내용)은 완벽히 같습니다.
- **②**: `IN` 연산자와 `OR` 연산자 모두 `NULL` 값과는 일치하지 않으므로 두 쿼리 모두 NULL 데이터를 포함하지 않습니다.
- **④**: A 쿼리는 평점이 1점인 행과 5점인 행이 모두 정상적으로 조회됩니다.

</details>

---

## Q7. 복합 조건(AND, BETWEEN) 쿼리 해석

**다음 쿼리의 실행 결과에 대한 설명으로 옳은 것은?**

```sql
01  SELECT customer_id, name
02  FROM customers
03  WHERE is_active = 'Y'
04  AND registered_date BETWEEN TO_DATE('2025-01-01', 'YYYY-MM-DD')
05                          AND TO_DATE('2025-06-30', 'YYYY-MM-DD');
```

① 비활성 고객도 포함된다.  
② 등록일이 2025년 7월 이후인 고객도 포함된다.  
③ 활성 고객 중 2025년 상반기 등록 고객만 조회된다.  
④ 고객 이름이 NULL인 고객만 조회된다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ③ 활성 고객 중 2025년 상반기 등록 고객만 조회된다.**

### **💡 해설**
- `WHERE is_active = 'Y'`: 현재 활동 중인 고객(`활성 고객`)만 필터링합니다.
- `registered_date BETWEEN TO_DATE('2025-01-01', 'YYYY-MM-DD') AND TO_DATE('2025-06-30', 'YYYY-MM-DD')`: 2025년 1월 1일부터 2025년 6월 30일까지(`2025년 상반기`) 등록된 가입 건만 선별합니다.
- 두 조건이 **`AND` (교집합)**로 연결되어 있으므로, **"활성 고객이면서 동시에 2025년 상반기에 가입한 고객"**만 조회됩니다.

### **❌ 오답 분석**
- **①**: `is_active = 'Y'` 조건에 의해 비활성 고객(`'N'`)은 결과에서 완전히 제외됩니다.
- **②**: `BETWEEN`의 종료일이 `2025-06-30`이므로 7월 이후 등록 고객은 포함되지 않습니다.
- **④**: 고객 이름이 NULL인 고객을 찾는 `WHERE name IS NULL` 조건은 전혀 포함되어 있지 않습니다.

</details>
