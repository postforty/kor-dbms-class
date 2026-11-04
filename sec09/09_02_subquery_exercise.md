# Oracle SQL 서브쿼리(Subquery) 연습문제

본 문서는 [09_01_subquery_guide.md](./09_01_subquery_guide.md)에서 학습한 단일행 서브쿼리, 상관 서브쿼리, 다중행 서브쿼리(`IN`, `ANY`, `ALL`, `EXISTS`, `NOT EXISTS`), 다중 열 서브쿼리, 그리고 인라인 뷰(Top-N)의 핵심 문법과 실무 활용법을 점검하기 위한 실전 연습문제입니다.

각 문제를 먼저 풀어본 후, 하단의 **정답 및 해설**을 펼쳐 올바른 SQL과 실행 결과, 상세 해설을 확인해 보세요.

---

## Q1. 가장 비싼 상품의 이름과 가격 조회하기

**`products` 테이블에서 가장 비싼 상품의 이름(`name`)과 가격(`price`)을 조회하시오.**

* **대상 테이블**: `products`
* **출력 컬럼**: `name`, `price`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 가장 비싼 상품(단일행 서브쿼리: MAX(price))
SELECT name, price
FROM products
WHERE price = (
    SELECT MAX(price)
    FROM products
);
```

### **쿼리 실행 결과**

| NAME | PRICE |
| :--- | :---: |
| 나는 내가 죽었다고 생각했습니다 | 17,000 |

### **💡 해설**
- **단일행 서브쿼리 활용 원리**:
  - `WHERE price = MAX(price)`와 같이 `WHERE` 절에 집계 함수를 직접 작성하면 `ORA-00934: 그룹 함수는 허가되지 않습니다` 에러가 발생합니다.
  - 따라서 서브쿼리 `(SELECT MAX(price) FROM products)`를 통해 전체 상품의 최고 가격(`17,000`)을 먼저 단일 값으로 계산한 뒤, 메인 쿼리에서 해당 가격과 일치하는(`=`) 상품의 이름과 가격을 조회합니다.
- **실행 순서**:
  1. 서브쿼리 먼저 실행 $\to$ 최고 가격 `17,000` 산출
  2. 메인 쿼리 실행 $\to$ `WHERE price = 17000` 조건으로 필터링 후 출력

</details>

---

## Q2. 각 고객의 가장 최근 주문만 조회하기

**각 고객의 가장 최근 주문만 조회하시오.**

* **대상 테이블**: `orders`
* **출력 컬럼**: `order_id`, `customer_id`, `order_date`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 고객별 최근 주문(고객별 MAX(order_date))
SELECT o.order_id, o.customer_id, o.order_date
FROM orders o
WHERE o.order_date = (
    SELECT MAX(o2.order_date)
    FROM orders o2
    WHERE o2.customer_id = o.customer_id
)
ORDER BY o.customer_id ASC;
```

### **쿼리 실행 결과**

| ORDER_ID | CUSTOMER_ID | ORDER_DATE |
| :---: | :---: | :---: |
| 4 | 1 | 2025-06-13 |
| 20 | 2 | 2025-05-08 |
| 13 | 4 | 2025-06-27 |
| 8 | 5 | 2025-05-14 |
| 10 | 6 | 2025-06-16 |
| 6 | 7 | 2025-06-11 |
| 17 | 8 | 2025-06-11 |
| 11 | 9 | 2025-05-23 |
| 12 | 10 | 2025-05-08 |

*(※ 실제 주문 이력이 존재하는 9명의 고객별 가장 최근 주문 내역 9건이 정확히 출력됩니다.)*

### **💡 해설**
- **상관 서브쿼리(Correlated Subquery)의 동작 메커니즘**:
  - 메인 쿼리 `orders o`의 각 행을 읽을 때마다, 해당 행의 고객 ID(`o.customer_id`)를 서브쿼리 내부 조건(`WHERE o2.customer_id = o.customer_id`)으로 전달합니다.
  - 서브쿼리는 **해당 고객이 생성한 주문들 중에서 가장 최근 주문일자**(`MAX(o2.order_date)`)를 계산하여 메인 쿼리에 전달합니다.
  - 메인 쿼리의 주문일자(`o.order_date`)가 고객별 최근 주문일자와 일치하는 행만 최종 결과로 출력됩니다.

</details>

---

## Q3. 리뷰가 작성된 상품의 product_id, name 조회하기

**리뷰가 작성된 상품의 `product_id`, `name`을 조회하시오.**

* **대상 테이블**: `products`, `reviews`
* **출력 컬럼**: `product_id`, `name`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 리뷰가 존재하는 상품(IN)
SELECT p.product_id, p.name
FROM products p
WHERE p.product_id IN (
    SELECT DISTINCT r.product_id
    FROM reviews r
)
ORDER BY p.product_id ASC;
```

### **쿼리 실행 결과**

| PRODUCT_ID | NAME |
| :---: | :--- |
| 1 | 82년생 김지영 |
| 2 | 소년이 온다 |
| 3 | 채식주의자 |
| 5 | 위대한 개츠비 |
| 7 | 몰입 |
| 9 | 아주 작은 습관의 힘 |
| 10 | 소크라테스의 변명 |
| 11 | 니코마코스 윤리학 |
| 13 | 프로이트 심리학 |
| 14 | 나는 내가 죽었다고 생각했습니다 |
| 15 | EQ 감성지능 |
| 17 | 지구 끝의 온실 |
| 19 | 동물농장 |
| 21 | 성공의 정석 |
| 22 | 고급 노트북 |
| 23 | 라인프렌즈 볼펜 |
| 25 | 책상용 독서대 |

*(※ 전체 25개 상품 중 실제 리뷰가 등록된 17개 상품이 중복 없이 정확히 조회됩니다.)*

### **💡 해설**
- **다중행 서브쿼리 `IN` 연산자**:
  - 서브쿼리 `(SELECT DISTINCT r.product_id FROM reviews r)`는 리뷰가 1건이라도 등록된 상품 번호 목록(`1, 2, 3, 5, 7, ...`)을 반환합니다.
  - 메인 쿼리는 `WHERE p.product_id IN (...)` 조건을 통해 상품 테이블에서 해당 목록에 속하는 상품만 필터링합니다.
- **`JOIN` 대비 서브쿼리의 장점**:
  - 일반 `JOIN`을 사용하면 1개 상품에 리뷰가 여러 개 있을 때 행이 중복되어 출력되므로 `DISTINCT`를 메인에 걸어야 하지만, 서브쿼리 `IN`을 사용하면 `products` 기준 1:1 결과가 보장되어 깔끔합니다.

</details>

---

## Q4. 카테고리 7번 상품 중 어느 하나의 가격보다도 비싼 상품 조회하기

**카테고리 7번 상품 중 어느 하나의 가격보다도 비싼 상품을 조회하시오.**

* **대상 테이블**: `products`
* **출력 컬럼**: `product_id`, `name`, `price`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 카테고리 7의 어느 상품 가격보다라도 큰 상품(> ANY => > MIN)
SELECT product_id, name, price
FROM products
WHERE price > ANY (
    SELECT price
    FROM products
    WHERE category_id = 7
)
ORDER BY product_id ASC;
```

### **쿼리 실행 결과 예시** (상위 10건)

| PRODUCT_ID | NAME | PRICE |
| :---: | :--- | :---: |
| 1 | 82년생 김지영 | 14,000 |
| 2 | 소년이 온다 | 12,000 |
| 3 | 채식주의자 | 12,900 |
| 4 | 노르웨이의 숲 | 15,700 |
| 5 | 위대한 개츠비 | 10,800 |
| 6 | 1984 | 7,700 |
| 7 | 몰입 | 16,500 |
| 9 | 아주 작은 습관의 힘 | 9,200 |
| 10 | 소크라테스의 변명 | 12,400 |
| 11 | 니코마코스 윤리학 | 15,700 |

*(※ 카테고리 7번 상품들의 최저가인 7,500원(성공의 정석)보다 비싼 전체 21개 상품이 조회됩니다.)*

### **💡 해설**
- **`> ANY`의 비교 메커니즘**:
  - 카테고리 7번 상품들의 가격 목록은 `[10800, 9200, 12400, 13300, 17000, 7500, 16300]`입니다.
  - `> ANY`는 이 값들 중 **어느 하나라도 만족하면 참**(OR)이 되므로, 실질적으로 **카테고리 7번 상품들의 최솟값(7,500원)을 초과하는 조건(`price > MIN(price)`)과 동일**하게 평가됩니다.
  - 따라서 7,500원보다 비싼 상품 총 21건이 올바르게 추출됩니다.

</details>

---

## Q5. 카테고리 1번의 모든 상품 가격보다 큰 상품 조회하기

**카테고리 1번의 모든 상품 가격보다 큰 상품을 조회하시오.**

* **대상 테이블**: `products`
* **출력 컬럼**: `product_id`, `name`, `price`
* **힌트**: `price > ALL(...)`는 최댓값보다 크다(`> MAX`)와 동일

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 카테고리 1의 모든 상품보다 비싼 상품(> ALL = > MAX)
SELECT product_id, name, price
FROM products
WHERE price > ALL (
    SELECT price
    FROM products
    WHERE category_id = 1
)
ORDER BY product_id ASC;
```

### **쿼리 실행 결과 예시** (상위 10건)

| PRODUCT_ID | NAME | PRICE |
| :---: | :--- | :---: |
| 1 | 82년생 김지영 | 14,000 |
| 2 | 소년이 온다 | 12,000 |
| 3 | 채식주의자 | 12,900 |
| 4 | 노르웨이의 숲 | 15,700 |
| 5 | 위대한 개츠비 | 10,800 |
| 6 | 1984 | 7,700 |
| 7 | 몰입 | 16,500 |
| 8 | 성공하는 사람들의 7가지 습관 | 4,100 |
| 9 | 아주 작은 습관의 힘 | 9,200 |
| 10 | 소크라테스의 변명 | 12,400 |

*(※ KOREA_MALL 샘플 데이터에서 1번 카테고리는 최상위 대분류('도서')이며 상품들은 하위 카테고리(7~11번)에 매핑되어 있으므로 서브쿼리가 공집합(0건)을 반환합니다. SQL 표준상 `> ALL (공집합)`은 항상 참(True)으로 평가되어 전체 25개 상품이 반환됩니다.)*

### **💡 해설**
- **`> ALL`의 비교 메커니즘**:
  - `> ALL`은 서브쿼리가 반환하는 **모든 값보다 커야**(AND) 참이 되므로, 실질적으로 **서브쿼리 결과의 최댓값보다 큰 조건(`price > MAX(price)`)과 동일**하게 평가됩니다.
- 💡 **심화 학습** (서브쿼리 공집합 반환 시 ALL의 특성):
  - 서브쿼리의 결과가 0건(공집합)일 때, `col > ALL (공집합)` 조건은 수학적 진공 참(Vacuous Truth) 원리에 의해 항상 `TRUE`로 평가됩니다.
  - 만약 특정 하위 카테고리(예: `category_id = 11`)를 대상으로 `WHERE price > ALL (SELECT price FROM products WHERE category_id = 11)`을 실행한다면, 11번 카테고리의 최고가(14,900원)보다 비싼 상품들(15,700원, 16,500원 등 총 6건)만 선별되어 출력됩니다.

</details>

---

## Q6. 주문이 존재하는 고객 조회하기

**주문이 존재하는 고객의 `customer_id`, `name`을 조회하시오.**

* **대상 테이블**: `customers`, `orders`
* **출력 컬럼**: `customer_id`, `name`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 주문이 존재하는 고객(EXISTS)
SELECT c.customer_id, c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
)
ORDER BY c.customer_id ASC;
```

### **쿼리 실행 결과**

| CUSTOMER_ID | NAME |
| :---: | :--- |
| 1 | 김하늘 |
| 2 | 박서준 |
| 4 | 정해인 |
| 5 | 신민아 |
| 6 | 이종석 |
| 7 | 김소현 |
| 8 | 남주혁 |
| 9 | 김태리 |
| 10 | 유연석 |

*(※ 전체 10명의 고객 중 주문 내역이 존재하는 9명의 고객이 출력됩니다. 주문 이력이 없는 3번 이보영 고객은 제외됩니다.)*

### **💡 해설**
- **`EXISTS` 연산자의 존재 검증**:
  - 메인 쿼리 `customers c`의 각 행에 대해 서브쿼리가 실행되며, `orders` 테이블에서 해당 고객의 주문(`o.customer_id = c.customer_id`)이 단 1건이라도 발견되는 즉시 `TRUE`를 반환하고 탐색을 종료(Short-Circuit)합니다.
- **`IN` 연산자와의 차이점**:
  - `WHERE c.customer_id IN (SELECT customer_id FROM orders)`로 작성할 수도 있지만, 대용량 주문 테이블 환경에서는 일치 행 발견 시 조기 종료되는 `EXISTS`가 훨씬 효율적입니다.

</details>

---

## Q7. 한 번도 주문하지 않은 고객 조회하기

**한 번도 주문하지 않은 고객의 `customer_id`, `name`을 조회하시오.**

* **대상 테이블**: `customers`, `orders`
* **출력 컬럼**: `customer_id`, `name`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 주문 이력이 없는 고객(NOT EXISTS)
SELECT c.customer_id, c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
)
ORDER BY c.customer_id ASC;
```

### **쿼리 실행 결과**

| CUSTOMER_ID | NAME |
| :---: | :--- |
| 3 | 이보영 |

*(※ 전체 10명의 고객 중 주문 내역이 단 한 건도 없는 3번 고객(이보영) 1건이 정확히 조회됩니다.)*

### **💡 해설**
- **`NOT EXISTS`의 부정 존재 검증**:
  - 메인 쿼리 `customers c`의 각 고객 행에 대해 서브쿼리가 실행되며, `orders` 테이블에서 해당 고객의 주문(`o.customer_id = c.customer_id`)이 **단 한 건도 존재하지 않는 경우에만** `TRUE`를 반환합니다.
- **`NOT IN`과의 결정적 차이** (NULL 안전성):
  - `WHERE c.customer_id NOT IN (SELECT customer_id FROM orders)`로도 작성할 수 있지만, 만약 주문 테이블의 `customer_id`에 `NULL`이 존재한다면 `NOT IN` 조건 전체가 `UNKNOWN(FALSE)`로 평가되어 결과가 0건이 반환되는 치명적인 논리 버그가 발생할 수 있습니다.
  - 반면 `NOT EXISTS`는 서브쿼리 내부 조인 조건(`o.customer_id = c.customer_id`)의 행 존재 여부만을 평가하므로, `NULL` 값의 존재와 무관하게 항상 안전하고 신뢰할 수 있는 결과를 보장합니다.

</details>

---

## Q8. 각 고객의 평균 평점보다 높은 리뷰 조회하기

**각 고객의 평균 평점보다 높은 리뷰만 조회하시오.**

* **대상 테이블**: `reviews`
* **출력 컬럼**: `review_id`, `customer_id`, `product_id`, `rating`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 고객 평균 평점보다 높은 리뷰(상관 서브쿼리)
SELECT r.review_id, r.customer_id, r.product_id, r.rating
FROM reviews r
WHERE r.rating > (
    SELECT AVG(r2.rating)
    FROM reviews r2
    WHERE r2.customer_id = r.customer_id
)
ORDER BY r.review_id ASC;
```

### **쿼리 실행 결과**

| REVIEW_ID | CUSTOMER_ID | PRODUCT_ID | RATING |
| :---: | :---: | :---: | :---: |
| 2 | 8 | 3 | 5 |
| 4 | 8 | 25 | 5 |
| 8 | 4 | 13 | 4 |
| 11 | 6 | 10 | 4 |
| 16 | 4 | 9 | 4 |
| 17 | 6 | 1 | 4 |
| 18 | 6 | 21 | 4 |
| 19 | 6 | 22 | 5 |

*(※ 전체 23건의 리뷰 중, 자신이 남긴 리뷰들의 평균 별점보다 더 높은 점수를 부여한 8건의 리뷰가 정확히 출력됩니다.)*

### **💡 해설**
- **상관 서브쿼리(Correlated Subquery)의 동작 원리**:
  - 메인 쿼리 `reviews r`의 각 행(개별 리뷰)을 검사할 때마다, 해당 리뷰를 작성한 고객 ID(`r.customer_id`)를 서브쿼리로 전달합니다.
  - 서브쿼리는 동일한 고객(`r2.customer_id = r.customer_id`)이 남긴 **모든 리뷰의 평균 별점**(`AVG(r2.rating)`)을 동적으로 계산하여 반환합니다.
  - 메인 쿼리는 현재 리뷰의 별점(`r.rating`)이 고객 본인의 평균 별점보다 큰(`>`) 행만을 결과 집합에 포함시킵니다.
  - 예시: 6번 고객(이종석)은 총 6개의 리뷰(별점: 4, 3, 3, 4, 4, 5)를 남겨 평균이 약 3.83점입니다. 따라서 3.83점보다 높은 4점과 5점 리뷰 4건(리뷰 번호 11, 17, 18, 19)이 출력됩니다.

</details>

---

## Q9. 리뷰가 존재하는 (고객, 상품) 조합의 주문 내역 조회하기

**리뷰가 존재하는 `(고객, 상품)` 조합의 주문 내역을 조회하시오.**

* **대상 테이블**: `orders`, `order_items`, `reviews`
* **출력 컬럼**: `o.order_id`, `o.customer_id`, `oi.product_id`, `o.order_date`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- (고객, 상품) 다중열 튜플 비교
SELECT o.order_id, o.customer_id, oi.product_id, o.order_date
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE (o.customer_id, oi.product_id) IN (
    SELECT r.customer_id, r.product_id
    FROM reviews r
)
ORDER BY o.order_id ASC, oi.product_id ASC;
```

### **쿼리 실행 결과 예시** (상위 10건)

| ORDER_ID | CUSTOMER_ID | PRODUCT_ID | ORDER_DATE |
| :---: | :---: | :---: | :---: |
| 2 | 8 | 1 | 2025-05-25 |
| 2 | 8 | 1 | 2025-05-25 |
| 2 | 8 | 3 | 2025-05-25 |
| 3 | 4 | 5 | 2025-06-11 |
| 5 | 8 | 14 | 2025-04-13 |
| 5 | 8 | 23 | 2025-04-13 |
| 5 | 8 | 25 | 2025-04-13 |
| 6 | 7 | 19 | 2025-06-11 |
| 7 | 4 | 2 | 2025-04-07 |
| 7 | 4 | 13 | 2025-04-07 |

*(※ 실제 주문한 고객이 해당 주문 상품에 대해 리뷰까지 남긴 (고객, 상품) 일치 조합에 해당하는 총 25건의 주문 상세 행이 조회됩니다.)*

### **💡 해설**
- **다중 열(Multi-Column) 서브쿼리와 튜플 비교** (Pairwise):
  - `WHERE (o.customer_id, oi.product_id) IN (...)`와 같이 두 개 이상의 컬럼을 괄호로 묶어 **하나의 쌍**(Pair / Tuple)으로 서브쿼리 결과 집합과 비교합니다.
  - 서브쿼리는 `reviews` 테이블에서 실제 리뷰가 작성된 고객과 상품 번호의 쌍 `(r.customer_id, r.product_id)` 목록을 반환합니다.
- ⚠️ **비쌍 비교(Non-pairwise) 안티패턴 주의**:
  - 만약 컬럼을 분리하여 `WHERE o.customer_id IN (SELECT customer_id FROM reviews) AND oi.product_id IN (SELECT product_id FROM reviews)`로 작성하면:
    - 고객 A가 상품 X에만 리뷰를 남기고, 고객 B가 상품 Y에만 리뷰를 남겼더라도,
    - 고객 A가 상품 Y를 주문한 내역까지 조건에 부합하게 되는 심각한 논리적 오류가 발생합니다.
  - 따라서 관계의 복합 키 매칭에는 반드시 **다중 열 튜플 비교**(`IN (SELECT col1, col2 ...)`)를 사용해야 합니다.

</details>

---

## Q10. 평균 평점이 높은 상품 Top 3 조회하기

**평균 평점이 높은 상품 Top 3를 조회하시오.**

* **대상 테이블**: `reviews`
* **출력 컬럼**: `product_id`, `avg_rating(소수 2자리)`

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답 SQL**

```sql
-- 인라인 뷰 + ROWNUM으로 Top-N
SELECT *
FROM (
    SELECT r.product_id,
           ROUND(AVG(r.rating), 2) AS avg_rating
    FROM reviews r
    GROUP BY r.product_id
    ORDER BY avg_rating DESC
)
WHERE ROWNUM <= 3;
```

### **쿼리 실행 결과**

| PRODUCT_ID | AVG_RATING |
| :---: | :---: |
| 22 | 5 |
| 25 | 4.5 |
| 13 | 4.5 |

*(※ 전체 리뷰 등록 상품 중 평균 별점이 가장 높은 상위 3개 상품인 22번, 25번, 13번 상품이 정확히 출력됩니다.)*

### **💡 해설**
- **인라인 뷰(Inline View)를 활용한 Top-N 처리 원리**:
  - SQL 실행 순서상 `WHERE` 절은 `ORDER BY`보다 먼저 실행됩니다.
  - 만약 인라인 뷰 없이 `FROM reviews WHERE ROWNUM <= 3 ORDER BY ...` 형태로 작성하면, 정렬되기도 전에 임의의 3행을 먼저 추출한 뒤 정렬해 버리는 치명적인 논리 버그가 발생합니다.
  - 따라서 **안쪽 서브쿼리(인라인 뷰)에서 상품별 평균 평점을 계산(`ROUND(AVG(r.rating), 2)`)하고 내림차순 정렬(`ORDER BY avg_rating DESC`)을 완벽히 마친 뒤, 바깥쪽 메인 쿼리에서 `WHERE ROWNUM <= 3`으로 상위 3건을 잘라내는 패턴**을 사용해야 정확한 Top-N 결과를 얻을 수 있습니다.
- **컬럼 별칭(Alias) 활용**:
  - 인라인 뷰 내부에서 정의한 별칭 `avg_rating`은 바깥 메인 쿼리에서 실제 컬럼명처럼 자유롭게 조회할 수 있습니다.

</details>

---

## 📋 서브쿼리(Subquery) 핵심 요약 노트

| 문제 번호 | 평가 핵심 주제 | 사용 연산자/구문 | 핵심 메커니즘 |
| :---: | :--- | :--- | :--- |
| **Q1** | **단일행 서브쿼리** | `=` , `MAX()` | `WHERE` 절 집계 함수 불가 극복 $\to$ `WHERE price = (SELECT MAX(price) ...)` |
| **Q2** | **상관 서브쿼리** | `=` , `MAX()` 상관 조건 | 메인 쿼리의 `customer_id`를 전달받아 고객별 최신일자 비교 (`o2.customer_id = o.customer_id`) |
| **Q3** | **다중행 서브쿼리** | `IN` , `DISTINCT` | 다중행 결과 목록과의 포함 여부 비교 $\to$ `WHERE product_id IN (SELECT DISTINCT ...)` |
| **Q4** | **ANY 연산자** | `> ANY` | 결과 목록 중 하나라도 초과 $\to$ 최솟값 초과(`> MIN`)와 동일 |
| **Q5** | **ALL 연산자** | `> ALL` | 결과 목록의 모든 값 초과 $\to$ 최댓값 초과(`> MAX`)와 동일 |
| **Q6** | **EXISTS 연산자** | `EXISTS` , `SELECT 1` | 조건 만족 행 발견 시 즉시 조기 종료(Short-Circuit) 탐색 |
| **Q7** | **NOT EXISTS 연산자** | `NOT EXISTS` , `SELECT 1` | 일치 행이 전혀 없는 경우만 참 $\to$ NULL에 영향받지 않는 안전한 부정 조건 |
| **Q8** | **상관 서브쿼리 집계** | `>` , `AVG()` 상관 조건 | 각 행의 기준 컬럼(`customer_id`)별 평균값을 동적으로 산출하여 대소 비교 |
| **Q9** | **다중 열 서브쿼리** | `(col1, col2) IN` | 두 컬럼의 조합을 튜플 단위로 비교(Pairwise)하여 데이터 무결성 보장 |
| **Q10** | **인라인 뷰** (Top-N) | `ROWNUM <= 3` , `ORDER BY` | 인라인 뷰에서 집계·정렬을 완료한 후 바깥 쿼리에서 순번 제한으로 정확한 상위 N건 추출 |
