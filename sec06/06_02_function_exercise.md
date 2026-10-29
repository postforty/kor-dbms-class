# Oracle SQL 단일행 함수(Single-Row Functions) 및 제어문 연습문제

본 문서는 [06_01_function_guide.md](./06_01_function_guide.md)에서 학습한 문자열 함수, 날짜/숫자 함수, 형 변환 함수, 그리고 `NULL` 처리 및 조건 분기 함수의 기본 문법과 활용법을 점검하기 위한 연습문제입니다.

각 문제를 먼저 풀어본 후, 하단의 **정답 및 해설**을 펼쳐 올바른 해설과 오답 분석을 확인해 보세요.

---

## Q1. 고객 테이블 NULL 처리 (NVL)

**고객 테이블에서 이메일이 없는 고객에게 '미등록'을 표시하려고 한다. 다음 SQL의 빈칸을 채우시오.**

```sql
SELECT name, ________(email, '미등록') AS "이메일"
FROM customers;
```

① `NVL`  
② `TO_CHAR`  
③ `DECODE`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ① `NVL`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT name, NVL(email, '미등록') AS "이메일"
  FROM customers;
  ```
- **`NVL(컬럼, 대체값)`** 함수는 첫 번째 인자의 컬럼 값이 `NULL`일 경우 지정한 대체값(두 번째 인자)을 반환하고, `NULL`이 아니면 원래의 컬럼 값을 그대로 반환합니다.
- 문제에서 이메일이 없는(`NULL`) 고객에게 `'미등록'`이라는 기본 문자열을 출력하고자 하므로 `NVL`이 가장 적절합니다.

### **❌ 오답 분석**
- **② `TO_CHAR`**: 숫자나 날짜 데이터를 문자열 형태로 변환하는 함수로, `NULL`을 특정 문자열로 직접 치환하는 기능은 없습니다.
- **③ `DECODE`**: `DECODE(컬럼, 기준값1, 결과1, ... , 기본값)` 형태로 동등 비교(`=`) 조건 분기를 수행하는 함수입니다. 인자의 구성 형태(`(email, '미등록')`)가 맞지 않습니다.

</details>

---

## Q2. 전화번호 등록 여부 분기 처리 (NVL2)

**고객 테이블에서 전화번호가 있으면 '등록됨', 없으면 '미등록'으로 출력하려고 한다. 다음 SQL의 빈칸을 채우시오.**

```sql
SELECT name, ____(phone, '등록됨', '미등록') AS "전화상태"
FROM customers;
```

① `NVL`  
② `NVL2`  
③ `COALESCE`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `NVL2`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT name, NVL2(phone, '등록됨', '미등록') AS "전화상태"
  FROM customers;
  ```
- **`NVL2(컬럼, 값1, 값2)`** 함수는 삼항 연산자와 유사하게 동작합니다:
  - 첫 번째 인자(`phone`)가 **`NULL`이 아니면** 두 번째 인자(`'등록됨'`) 반환
  - 첫 번째 인자(`phone`)가 **`NULL`이면** 세 번째 인자(`'미등록'`) 반환
- 따라서 세 개의 인자를 받아 `NULL` 여부에 따라 서로 다른 두 가지 값을 반환하는 함수는 `NVL2`입니다.

### **❌ 오답 분석**
- **① `NVL`**: 인자를 2개만 받으며(`NVL(expr1, expr2)`), `NULL`일 때만 두 번째 인자를 반환하므로 3개의 인자를 사용하는 위 구문에는 적합하지 않습니다.
- **③ `COALESCE`**: 나열된 여러 인자 중 **첫 번째로 `NULL`이 아닌 값**을 순차적으로 찾아 반환하는 함수로, 위와 같이 조건 분기('등록됨' / '미등록')를 처리하는 용도와는 문법적 의도가 다릅니다.

</details>

---

## Q3. 문자열 부분 추출 (SUBSTR)

**상품 테이블에서 상품명 중 앞의 5글자만 표시하려고 한다. 다음 SQL의 빈칸을 채우시오.**

```sql
SELECT ________(name, 1, 5) AS "상품명_요약"
FROM products;
```

① `LENGTH`  
② `SUBSTR`  
③ `INSTR`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `SUBSTR`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT SUBSTR(name, 1, 5) AS "상품명_요약"
  FROM products;
  ```
- **`SUBSTR(문자열, 시작위치, 길이)`** 함수는 대상 문자열의 특정 시작 위치에서 지정한 길이만큼 부분 문자열을 잘라내어 반환합니다.
- `SUBSTR(name, 1, 5)`는 `name` 컬럼의 1번째 글자부터 시작하여 총 5글자를 추출하므로 요구사항에 정확히 부합합니다.

### **❌ 오답 분석**
- **① `LENGTH`**: 문자열의 전체 글자 수를 반환하는 함수로, 인자를 1개(`LENGTH(문자열)`)만 받습니다.
- **③ `INSTR`**: 특정 문자나 문자열이 시작되는 위치(인덱스 번호)를 찾아 정수로 반환하는 함수입니다.

</details>

---

## Q4. 날짜 연산 (날짜 더하기)

**주문 테이블에서 주문일(order_date)로부터 7일 후의 날짜를 구하려고 한다. 빈칸에 들어갈 내용으로 올바른 것은?**

```sql
SELECT order_id, order_date, ____________(order_date, 7) AS "배송예정일"
FROM orders;
```

① `ADD_MONTHS`  
② `ADD_MONTHS(order_date, 0.25)`  
③ `order_date + 7`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ③ `order_date + 7`**

### **💡 해설**
- **날짜 연산 완성 구문**:
  ```sql
  SELECT order_id, order_date, order_date + 7 AS "배송예정일"
  FROM orders;
  ```
- Oracle에서 날짜(`DATE`) 데이터 타입에 정수를 더하거나 빼면 **'일수(Day)' 단위**로 직접 가감 계산이 수행됩니다. 따라서 7일 후의 날짜는 함수 호출 없이 간단히 `order_date + 7`로 작성합니다.
- *(참고: 문제 지문 템플릿에 `____(order_date, 7)` 형태로 괄호 표기가 있더라도, 날짜에 일수를 더할 때 가장 표준적이고 올바른 오라클 표현식은 산술 연산자 `+`를 사용한 `order_date + 7`입니다.)*

### **❌ 오답 분석**
- **① `ADD_MONTHS`**: 날짜에 특정 '개월 수(Month)'를 더하는 함수입니다. `ADD_MONTHS(order_date, 7)`을 적용하면 7일 뒤가 아니라 **7개월 뒤**가 됩니다.
- **② `ADD_MONTHS(order_date, 0.25)`**: `ADD_MONTHS`의 두 번째 인자로 소수를 넘길 경우 오라클은 소수점을 절삭(버림)하여 정수로 처리하므로 의도한 7일 계산이 되지 않습니다.

</details>

---

## Q5. TO_CHAR() 함수의 목적

**리뷰 테이블에서 평점(rating)을 문자열로 변환하여 출력하려고 한다. TO_CHAR() 함수의 목적을 올바르게 설명한 것은?**

```sql
SELECT review_id, TO_CHAR(rating) AS "평점(문자)"
FROM reviews;
```

① 숫자 → 문자로 변환  
② 문자 → 날짜로 변환  
③ 날짜 → 숫자로 변환  
④ NULL 처리  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ① 숫자 → 문자로 변환**

### **💡 해설**
- `rating` 컬럼은 숫자로 저장된 데이터(예: 1~5점 평점)입니다.
- **`TO_CHAR()`** 함수는 숫자(`NUMBER`) 또는 날짜(`DATE`) 타입의 데이터를 지정된 서식의 **문자열(`VARCHAR2`) 타입으로 변환**할 때 사용합니다.
- 위 쿼리에서는 숫자 타입인 평점 컬럼을 문자로 변환하고 있으므로 ①번이 올바른 설명입니다.

### **❌ 오답 분석**
- **② 문자 → 날짜로 변환**: 문자를 날짜 타입으로 바꿀 때는 **`TO_DATE()`** 함수를 사용합니다.
- **③ 날짜 → 숫자로 변환**: 날짜는 직접 숫자로 바꾸기보다 문자로 변환하거나 일수 연산 차이를 구합니다. (문자를 숫자로 바꿀 때는 `TO_NUMBER()` 사용)
- **④ NULL 처리**: `NULL` 처리는 `NVL`, `NVL2`, `COALESCE` 등의 함수를 사용합니다.

</details>

---

## Q6. 집계/그룹 함수 (AVG)

**상품 테이블에서 모든 상품의 평균 가격을 조회하려고 한다. 빈칸을 채우시오.**

```sql
SELECT ________(price) AS "평균가격"
FROM products;
```

① `SUM`  
② `AVG`  
③ `COUNT`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `AVG`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT AVG(price) AS "평균가격"
  FROM products;
  ```
- **`AVG(컬럼)`** 함수는 지정된 숫자 컬럼의 **평균(Average)** 값을 계산하여 반환하는 대표적인 그룹(집계) 함수입니다.
- 문제에서 "모든 상품의 평균 가격"을 조회하도록 요구하고 있으므로 `AVG`가 정답입니다.

### **❌ 오답 분석**
- **① `SUM`**: 지정된 컬럼 값들의 **합계(총합)**를 계산하는 함수입니다.
- **③ `COUNT`**: 조건을 만족하는 **행(Row)의 개수**를 세는 함수입니다.

</details>

---

## Q7. 카테고리별 그룹화 (GROUP BY)

**상품 테이블에서 카테고리별 총 재고수량을 구하려고 한다. 빈칸을 채우시오.**

```sql
SELECT category_id, SUM(stock_quantity) AS "총재고"
FROM products
________ category_id;
```

① `GROUP BY`  
② `HAVING`  
③ `ORDER BY`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ① `GROUP BY`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT category_id, SUM(stock_quantity) AS "총재고"
  FROM products
  GROUP BY category_id;
  ```
- **`GROUP BY 컬럼명`** 절은 특정 컬럼의 값이 동일한 행들을 묶어서 그룹화할 때 사용합니다.
- `SELECT` 절에 일반 컬럼(`category_id`)과 그룹 집계 함수(`SUM(stock_quantity)`)가 함께 사용되었으므로, 집계 함수가 적용되지 않은 일반 컬럼은 반드시 `GROUP BY` 절에 명시해야 카테고리별 소계를 구할 수 있습니다.

### **❌ 오답 분석**
- **② `HAVING`**: `GROUP BY`로 묶인 그룹 집계 결과에 조건을 부여하여 필터링할 때 사용하는 절입니다 (`HAVING SUM(stock_quantity) >= 100` 등). 기준 컬럼을 지정하는 위치가 아닙니다.
- **③ `ORDER BY`**: 최종 조회 결과를 특정 순서(오름차순/내림차순)로 정렬할 때 사용하는 절입니다.

</details>

---

## Q8. 그룹 필터링 조건 (HAVING)

**상품 테이블에서 카테고리별 평균가격이 20,000원 이상인 경우만 출력하려고 한다. 빈칸을 채우시오.**

```sql
SELECT category_id, AVG(price) AS "평균가격"
FROM products
GROUP BY category_id
________ AVG(price) >= 20000;
```

① `WHERE`  
② `HAVING`  
③ `ORDER BY`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `HAVING`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT category_id, AVG(price) AS "평균가격"
  FROM products
  GROUP BY category_id
  HAVING AVG(price) >= 20000;
  ```
- **`HAVING` 절**은 `GROUP BY`를 통해 그룹화된 집계 결과(그룹 함수 조건)를 필터링할 때 사용합니다.
- 집계 함수(`AVG(price)`)를 이용한 조건식은 `WHERE` 절에서 사용할 수 없으며, 반드시 **`HAVING` 절**에 작성해야 합니다.

### **❌ 오답 분석**
- **① `WHERE`**: 개별 행(Row) 단위로 필터링을 수행하는 절입니다. 그룹 집계 함수(`AVG`, `SUM` 등)를 `WHERE` 절 조건으로 사용하면 `ORA-00934: 그룹 함수는 허가되지 않습니다` 에러가 발생합니다.
- **③ `ORDER BY`**: 결과를 정렬하는 절로, 비교 연산 조건식(`>= 20000`)이 올 수 없습니다.

</details>

---

## Q9. 다차원 소계 및 총계 집계 (ROLLUP)

**상품 테이블에서 카테고리별, 상품별 총 재고수량과 전체합계를 함께 조회하려 한다. 빈칸을 채우시오.**

```sql
SELECT category_id, name, SUM(stock_quantity) AS "총재고"
FROM products
GROUP BY ________(category_id, name);
```

① `ROLLUP`  
② `CUBE`  
③ `GROUPING SETS`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ① `ROLLUP`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT category_id, name, SUM(stock_quantity) AS "총재고"
  FROM products
  GROUP BY ROLLUP(category_id, name);
  ```
- **`ROLLUP(A, B)`** 함수는 인자로 전달된 컬럼의 계층적 순서(오른쪽에서 왼쪽 방향)에 따라 단계별 소계와 최종 전체 합계(총계)를 생성합니다.
  - `(category_id, name)`: 카테고리별 + 상품별 상세 소계
  - `(category_id)`: 카테고리별 소계
  - `()`: 전체 총합계 (Grand Total)
- 문제에서 카테고리별, 상품별 총 재고수량과 함께 계층적 소계 및 전체합계를 산출하고자 하므로 `ROLLUP`이 정답입니다.

### **❌ 오답 분석**
- **② `CUBE`**: 결합 가능한 모든 가능한 컬럼 조합(다차원 큐브 집계: $2^N$ 가지 조합)의 소계를 구하는 함수입니다. `(category_id, name)`, `(category_id)`, `(name)`, `()`를 모두 생성하므로 단순 계층적 단계 집계와는 차이가 있습니다.
- **③ `GROUPING SETS`**: 명시적으로 지정한 컬럼 그룹 단위별로만 개별 집계(Union All 효과)를 생성하는 함수로, 자동으로 계층적 단계 소계나 총계를 누적 생성하지는 않습니다.

</details>


