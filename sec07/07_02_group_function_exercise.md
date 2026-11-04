# Oracle SQL 그룹 함수(Group Functions) 및 집계 연습문제

본 문서는 [07_01_group_function_guide.md](./07_01_group_function_guide.md)에서 학습한 그룹 함수(`COUNT`, `SUM`, `AVG`, `MAX`, `MIN`), `GROUP BY` 절을 활용한 소그룹화, `HAVING` 절을 이용한 집계 필터링, 그리고 다차원 집계(`ROLLUP`)의 기본 문법과 활용법을 점검하기 위한 연습문제입니다.

각 문제를 먼저 풀어본 후, 하단의 **정답 및 해설**을 펼쳐 올바른 해설과 오답 분석을 확인해 보세요.

---

## Q1. 집계/그룹 함수 (AVG)

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
- `AVG(컬럼)` 함수는 지정된 숫자 컬럼의 **평균**(Average) 값을 계산하여 반환하는 대표적인 그룹(집계) 함수입니다.
- 문제에서 "모든 상품의 평균 가격"을 조회하도록 요구하고 있으므로 `AVG`가 정답입니다.

### **❌ 오답 분석**
- **① `SUM`**: 지정된 컬럼 값들의 **합계**(총합)를 계산하는 함수입니다.
- **③ `COUNT`**: 조건을 만족하는 **행**(Row)의 개수를 세는 함수입니다.

</details>

---

## Q2. 카테고리별 그룹화 (GROUP BY)

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
- `GROUP BY 컬럼명` 절은 특정 컬럼의 값이 동일한 행들을 묶어서 그룹화할 때 사용합니다.
- `SELECT` 절에 일반 컬럼(`category_id`)과 그룹 집계 함수(`SUM(stock_quantity)`)가 함께 사용되었으므로, 집계 함수가 적용되지 않은 일반 컬럼은 반드시 `GROUP BY` 절에 명시해야 카테고리별 소계를 구할 수 있습니다.

### **❌ 오답 분석**
- **② `HAVING`**: `GROUP BY`로 묶인 그룹 집계 결과에 조건을 부여하여 필터링할 때 사용하는 절입니다 (`HAVING SUM(stock_quantity) >= 100` 등). 기준 컬럼을 지정하는 위치가 아닙니다.
- **③ `ORDER BY`**: 최종 조회 결과를 특정 순서(오름차순/내림차순)로 정렬할 때 사용하는 절입니다.

</details>

---

## Q3. 그룹 필터링 조건 (HAVING)

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

## Q4. 다차원 소계 및 총계 집계 (ROLLUP)

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
- `ROLLUP(A, B)` 함수는 인자로 전달된 컬럼의 계층적 순서(오른쪽에서 왼쪽 방향)에 따라 단계별 소계와 최종 전체 합계(총계)를 생성합니다.
  - `(category_id, name)`: 카테고리별 + 상품별 상세 소계
  - `(category_id)`: 카테고리별 소계
  - `()`: 전체 총합계 (Grand Total)
- 문제에서 카테고리별, 상품별 총 재고수량과 함께 계층적 소계 및 전체합계를 산출하고자 하므로 `ROLLUP`이 정답입니다.

### **❌ 오답 분석**
- **② `CUBE`**: 결합 가능한 모든 가능한 컬럼 조합(다차원 큐브 집계: $2^N$ 가지 조합)의 소계를 구하는 함수입니다. `(category_id, name)`, `(category_id)`, `(name)`, `()`를 모두 생성하므로 단순 계층적 단계 집계와는 차이가 있습니다.
- **③ `GROUPING SETS`**: 명시적으로 지정한 컬럼 그룹 단위별로만 개별 집계(Union All 효과)를 생성하는 함수로, 자동으로 계층적 단계 소계나 총계를 누적 생성하지는 않습니다.

</details>

---

## Q5. 고유값 건수 집계 (COUNT DISTINCT)

**리뷰 테이블(reviews)에서 한 번이라도 리뷰를 작성한 '순수 고유 고객(Unique Customers) 수'를 구하고자 한다. 올바른 SQL은?**

① `SELECT COUNT(customer_id) FROM reviews;`  
② `SELECT COUNT(*) FROM reviews;`  
③ `SELECT COUNT(DISTINCT customer_id) FROM reviews;`  
④ `SELECT DISTINCT COUNT(customer_id) FROM reviews;`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ③ `SELECT COUNT(DISTINCT customer_id) FROM reviews;`**

### **💡 해설**
- 한 명의 고객이 여러 건의 상품 리뷰를 작성했을 수 있으므로, 단순 `COUNT(customer_id)`를 실행하면 동일한 고객 ID가 중복 카운트됩니다.
- 집계 함수 괄호 안에 `DISTINCT` 키워드를 명시(`COUNT(DISTINCT customer_id)`)하면, 중복된 `customer_id`를 1건으로 처리하여 **실제로 리뷰를 작성한 고유 고객 수**를 정확히 산출할 수 있습니다.

### **❌ 오답 분석**
- **① `SELECT COUNT(customer_id) FROM reviews;`**: `customer_id`가 `NULL`이 아닌 모든 행의 개수를 세므로, 중복 고객이 포함된 전체 리뷰 건수가 계산됩니다.
- **② `SELECT COUNT(*) FROM reviews;`**: 테이블의 물리적 전체 행 수를 집계하므로 작성자 수가 아닌 전체 리뷰 등록 건수가 반환됩니다.
- **④ `SELECT DISTINCT COUNT(customer_id) FROM reviews;`**: `COUNT()` 집계가 먼저 실행되어 이미 1개의 숫자(총 건수)로 요약된 후 `DISTINCT`가 적용되므로 중복 제거 효과가 전혀 없습니다.

</details>

