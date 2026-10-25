# Oracle SQL 데이터 정렬(ORDER BY), 중복 제거(DISTINCT), 행 수 제한(ROW LIMITING) 연습문제

본 문서는 [05_01_order_by_guide.md](file:///c:/Users/dandycode/Documents/GitHub/kor-dbms-class/sec05/05_01_order_by_guide.md)에서 학습한 정렬 방향(`ASC`, `DESC`), 다중 컬럼 정렬, `NULL` 정렬 제어(`NULLS FIRST`, `NULLS LAST`), `DISTINCT`를 통한 고유 데이터 추출, 그리고 SQL 표준 행 수 제한(`FETCH FIRST`, `OFFSET`)과 전통적인 `ROWNUM` 가상 컬럼의 동작 원리를 점검하기 위한 연습문제입니다.

각 문제를 먼저 풀어본 후, 하단의 **정답 및 해설**을 펼쳐 올바른 해설과 오답 분석을 확인해 보세요.

---

## Q1. ORDER BY 정렬 기본 구문 작성

**products 테이블에서 상품명을 오름차순으로 정렬하여 전체 목록을 조회하는 SQL을 작성하시오.**

```sql
01  -- 작성할 SQL : 상품명을 오름차순으로 정렬하여 전체 목록을 조회하는 SQL
02  SELECT _________ , _________
03  FROM _________
04  ORDER BY _________ ASC;
```

① `name, price, products, name`  
② `product_id, price, orders, product_id`  
③ `name, product_id, products, price`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ① `name, price, products, name`**

### **💡 해설**
- **빈칸 완성 결과**:
  ```sql
  SELECT name, price
  FROM products
  ORDER BY name ASC;
  ```
- 문제 요구사항인 **"products 테이블에서 상품명을 오름차순으로 정렬하여 전체 목록을 조회"**를 충족하기 위해:
  1. 조회 대상 테이블(`FROM` 절): **`products`**
  2. 정렬 기준 컬럼(`ORDER BY` 절): 상품명 컬럼인 **`name`**
  3. 조회 대상 컬럼(`SELECT` 절): 상품 정보인 **`name, price`**
- 따라서 네 개의 빈칸에 순서대로 `name, price, products, name`이 채워진 ①번이 올바른 정답입니다.
- *(참고: `ORDER BY` 절에서 `ASC`(오름차순)는 기본값이므로 생략할 수 있지만, 문제 구문에 `ASC;`가 이미 주어져 있으므로 기준 컬럼명 `name`만 올바르게 입력하면 됩니다.)*

### **❌ 오답 분석**
- **② `product_id, price, orders, product_id`**: 대상 테이블이 `orders`(주문)로 잘못 지정되었으며, 정렬 기준도 상품명이 아닌 `product_id`로 지정되어 문제 요구사항과 맞지 않습니다.
- **③ `name, product_id, products, price`**: 대상 테이블은 `products`가 맞지만, `ORDER BY`의 정렬 기준 컬럼 자리에 상품명(`name`)이 아닌 단가(`price`)가 입력되어 상품명 오름차순 정렬이 되지 않습니다.

</details>

---

## Q2. 다중 컬럼 정렬 (Multi-Column ORDER BY)

**리뷰 테이블에서 별점이 높은 순으로 정렬하되, 별점이 같을 경우 작성일이 오래된 순으로 정렬하려고 한다. 올바른 SQL을 고르시오.**

① `SELECT * FROM reviews ORDER BY rating, created_at;`  
② `SELECT * FROM reviews ORDER BY rating DESC, created_at ASC;`  
③ `SELECT * FROM reviews ORDER BY created_at DESC, rating;`  
④ `SELECT * FROM reviews ORDER BY rating DESC, created_at DESC;`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `SELECT * FROM reviews ORDER BY rating DESC, created_at ASC;`**

### **💡 해설**
- **다중 정렬(Multi-Column Sort) 기준 설정**:
  1. **1차 정렬 기준 (별점이 높은 순)**: 큰 점수부터 작은 점수 순이므로 내림차순인 **`rating DESC`**를 적용합니다.
  2. **2차 정렬 기준 (별점이 같을 때 작성일이 오래된 순)**: 과거 날짜부터 최신 날짜 순(오래된 순)이므로 날짜 값이 작은 것부터 큰 것으로 나열하는 오름차순인 **`created_at ASC`**를 적용합니다.
- `ORDER BY` 절에서 컬럼들을 쉼표(`,`)로 구분하여 나열하면 왼쪽부터 우선순위가 부여되므로 `ORDER BY rating DESC, created_at ASC;`가 완벽하게 일치합니다.

### **📅 날짜 컬럼 정렬 방향 핵심 정리**
| 정렬 키워드 | 정렬 방향 | 날짜 해석 | 비즈니스 표현 |
| :---: | :---: | :---: | :---: |
| **`ASC`** | 작은 값 → 큰 값 | 과거 날짜 → 최신 날짜 | **오래된 순**, 등록순 |
| **`DESC`** | 큰 값 → 작은 값 | 최신 날짜 → 과거 날짜 | **최신순**, 최근 등록순 |

### **❌ 오답 분석**
- **① `ORDER BY rating, created_at;`**: `rating`과 `created_at` 모두 `ASC`가 기본 적용되어 별점이 낮은 순(1점→5점)으로 정렬되므로 요구사항(별점이 높은 순)과 반대로 동작합니다.
- **③ `ORDER BY created_at DESC, rating;`**: 1차 정렬 기준이 작성일(`created_at DESC`)이 되어 작성일자가 최신인 순서로 먼저 정렬되므로 요구사항의 우선순위와 완전히 어긋납니다.
- **④ `ORDER BY rating DESC, created_at DESC;`**: 1차 기준은 맞지만, 2차 기준이 `created_at DESC`로 지정되어 별점이 같을 때 '최신순'으로 정렬됩니다. 문제의 조건은 '오래된 순'이므로 오답입니다.

</details>

---

## Q3. 중복 데이터 제거 (DISTINCT)

**products 테이블에서 등록된 카테고리 번호를 중복 없이 조회하는 SQL로 올바른 것은?**

① `SELECT category_id FROM products;`  
② `SELECT category_id FROM products GROUP BY category_id;`  
③ `SELECT DISTINCT category_id FROM products;`  
④ `SELECT UNIQUE category_id FROM products;`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ③ `SELECT DISTINCT category_id FROM products;`**

### **💡 해설**
- **`DISTINCT` 키워드의 역할**:
  - `SELECT` 절 맨 앞(컬럼명 앞)에 위치하여 결과 집합에서 **중복된 행을 제거하고 고유한(Unique) 값만 1행씩 추출**합니다.
  - 여러 상품이 동일한 카테고리 번호(`category_id`)를 가지고 있더라도, `SELECT DISTINCT category_id FROM products;`를 실행하면 중복이 제거된 고유 카테고리 번호 목록만 깔끔하게 반환됩니다.
- 이는 SQL-92 표준 문법이며 모든 관계형 데이터베이스(Oracle, MySQL, PostgreSQL, SQL Server 등)에서 공통적으로 지원하는 표준 방식입니다.

### **❌ 오답 분석**
- **① `SELECT category_id FROM products;`**: 중복 제거 키워드가 없어 등록된 모든 상품 행의 `category_id`가 반복 출력됩니다.
- **② `SELECT category_id FROM products GROUP BY category_id;`**: 결과적으로 중복이 합쳐진 카테고리 목록이 나오긴 하지만, `GROUP BY`는 본래 집계 함수(`COUNT`, `SUM` 등)와 함께 통계 그룹을 생성하기 위한 절입니다. 단순히 고유 목록을 추출하기 위한 목적에는 `DISTINCT`가 문법적 의도에 부합합니다.
- **④ `SELECT UNIQUE category_id FROM products;`**: Oracle Database는 과거 하위 호환성을 위해 `UNIQUE`를 `DISTINCT`의 동의어로 허용하긴 하지만, 이는 ANSI SQL 표준 문법이 아닌 Oracle 고유의 레거시(구형) 비표준 문법입니다. 실무 및 자격시험에서는 표준 키워드인 **`DISTINCT`**가 정답입니다.

</details>

---

## Q4. 행 수 제한과 Top-N 쿼리 (복수정답 허용)

**리뷰 테이블에서 가장 최근에 작성된 5개의 리뷰만 조회하고자 한다. 올바른 SQL은? (복수정답 허용)**

① `SELECT * FROM reviews ORDER BY created_at DESC FETCH FIRST 5 ROWS ONLY;`  
② `SELECT * FROM reviews WHERE ROWNUM <= 5 ORDER BY created_at DESC;`  
③ `SELECT * FROM (SELECT * FROM reviews ORDER BY created_at DESC ) WHERE ROWNUM <= 5;`  
④ `SELECT * FROM reviews WHERE rating >= 3 AND ROWNUM <= 5;`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ①, ③**

### **💡 해설**
- **가장 최근에 작성된 5개 리뷰(Top-5)**를 조회하는 올바른 방법은 두 가지입니다:
  1. **①번 (최신 SQL 표준 방식, Oracle 12c+)**:
     - `ORDER BY created_at DESC FETCH FIRST 5 ROWS ONLY;`
     - 정렬이 완료된 최종 결과에서 상위 5건만 가져오는 직관적이고 표준적인 Row Limiting 문법입니다.
  2. **③번 (전통적 서브쿼리 방식, Oracle 전 버전 공통)**:
     - `SELECT * FROM (SELECT * FROM reviews ORDER BY created_at DESC) WHERE ROWNUM <= 5;`
     - 안쪽 서브쿼리(인라인 뷰)에서 전체 리뷰를 작성일 최신순(`created_at DESC`)으로 먼저 확실하게 정렬한 뒤, 바깥쪽 쿼리에서 이미 정렬된 결과 집합에 1번부터 차례대로 `ROWNUM <= 5`를 적용하므로 정확한 최신 5건이 보장됩니다.

### **⚠️ ②번이 오답인 이유 (SQL 실행 순서와 ROWNUM의 함정)**
```sql
-- [잘못된 쿼리] 절대 이렇게 작성하면 안 됩니다!
SELECT * FROM reviews
WHERE ROWNUM <= 5
ORDER BY created_at DESC;
```
- SQL 문장의 내부 실행 순서는 **`FROM` → `WHERE` → `SELECT` → `ORDER BY`** 순서입니다.
- 따라서 `WHERE ROWNUM <= 5`가 `ORDER BY created_at DESC`보다 **먼저 실행**됩니다.
- 결과적으로 전체 테이블에서 **무작위 5개 행을 먼저 뽑아온 뒤, 그 5개 데이터만을 가지고 최신순으로 정렬**하게 됩니다. 전체 데이터 중 가장 최근에 작성된 5개가 보장되지 않는 대표적인 안티패턴입니다.

### **❌ 오답 분석**
- **②**: `WHERE` 절의 `ROWNUM`이 `ORDER BY`보다 먼저 실행되어 무작위 5건만 정렬되는 치명적인 논리 오류입니다.
- **④**: 작성일 기준 정렬(`ORDER BY created_at DESC`)이 전혀 없으며, 평점 3점 이상인 리뷰 중 무작위 5건을 조회하는 쿼리입니다.

</details>

---

## Q5. 페이징 처리 (OFFSET과 FETCH NEXT)

**products 테이블에서 가격이 높은 순으로 정렬된 상품 목록 중, 6번째부터 5개 행만 출력하고자 한다. 적절한 SQL은?**

① `OFFSET 6 ROWS FETCH FIRST 5 ROWS ONLY`  
② `OFFSET 5 ROWS FETCH NEXT 5 ROWS ONLY`  
③ `ROWNUM BETWEEN 6 AND 10`  
④ `WHERE ROWNUM >= 6 AND ROWNUM <= 10`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② `OFFSET 5 ROWS FETCH NEXT 5 ROWS ONLY`**

### **💡 해설**
- **`OFFSET n ROWS`**: 결과 집합의 앞부분에서 **건너뛸(Skip) 행의 개수**를 지정합니다.
  - **"6번째 행부터"** 출력하려면 앞선 **1번째부터 5번째까지 총 5개 행을 건너뛰어야** 하므로 **`OFFSET 5 ROWS`**가 되어야 합니다. (`OFFSET 6 ROWS`는 6개를 건너뛰어 7번째부터 출력됨)
- **`FETCH NEXT n ROWS ONLY`**: 건너뛴 위치에서부터 **가져올 행의 개수**를 지정합니다.
  - 5개 행을 가져와야 하므로 **`FETCH NEXT 5 ROWS ONLY`** (또는 `FETCH FIRST 5 ROWS ONLY`)를 사용합니다.
- 완성된 페이징 쿼리:
  ```sql
  SELECT name, price
  FROM products
  ORDER BY price DESC
  OFFSET 5 ROWS FETCH NEXT 5 ROWS ONLY;
  ```

### **⚠️ ③, ④번이 오답인 이유 (ROWNUM의 1-based 시작 특성)**
- `ROWNUM`은 행이 추출될 때 `1`부터 번호가 부여되며, 조건 검사를 통과해야 다음 번호가 1씩 증가합니다.
- `WHERE ROWNUM >= 6` 또는 `ROWNUM BETWEEN 6 AND 10`을 직접 지정하면:
  1. 첫 번째 행을 읽고 `ROWNUM = 1`을 검사합니다.
  2. `1 >= 6`은 거짓(False)이므로 첫 번째 행이 버려집니다.
  3. 두 번째 행을 읽을 때도 이전 행이 채택되지 않았으므로 여전히 `ROWNUM = 1`로 검사합니다.
  4. 역시 거짓이므로 버려집니다.
  5. 결국 모든 행이 버려져 **결과가 0건(Empty Result)** 반환되는 치명적인 논리 오류가 발생합니다.

### **❌ 오답 분석**
- **① `OFFSET 6 ROWS ...`**: 6개 행을 건너뛰므로 6번째가 아닌 **7번째 행부터** 출력됩니다.
- **③, ④ `ROWNUM BETWEEN 6 AND 10`, `WHERE ROWNUM >= 6 ...`**: 인라인 뷰 없이 메인 쿼리의 `WHERE` 절에 직접 1보다 큰 시작 값을 지정하면 아무런 행도 조회되지 않습니다(0건 반환).

</details>

---

## 📋 ORDER BY · DISTINCT · Row Limiting 핵심 요약 노트

| 기능 분류 | 주요 키워드 및 구문 | 동작 원리 및 핵심 주의사항 |
| :--- | :--- | :--- |
| **정렬 방향** | `ORDER BY 컬럼 ASC` (오름차순)<br>`ORDER BY 컬럼 DESC` (내림차순) | • `ASC`는 기본값이므로 생략 가능 (`ORDER BY 컬럼`)<br>• 날짜 컬럼에서 `ASC`는 오래된 순, `DESC`는 최신순 |
| **다중 정렬** | `ORDER BY 컬럼1 DESC, 컬럼2 ASC` | • 왼쪽 컬럼이 1차 정렬 기준, 1차 기준이 동점일 때만 2차 기준 적용<br>• 컬럼마다 정렬 방향(`ASC`/`DESC`)을 독립적으로 지정 가능 |
| **NULL 정렬 제어** | `NULLS FIRST`<br>`NULLS LAST` | • Oracle 기본 동작: 오름차순 시 `NULLS LAST`, 내림차순 시 `NULLS FIRST`<br>• `NULLS LAST` 명시 시 내림차순 정렬에서도 NULL을 맨 아래로 배치 |
| **중복 제거** | `SELECT DISTINCT 컬럼1, 컬럼2` | • 선택한 모든 컬럼의 조합이 완벽히 일치하는 중복 행을 제거<br>• `SELECT` 바로 뒤에 위치하며, 대량 데이터 시 정렬/해시 부하 고려 필요 |
| **최신 행 수 제한** | `FETCH FIRST n ROWS ONLY` | • Oracle 12c+ SQL 표준 문법<br>• `ORDER BY` 정렬이 완료된 최종 결과 집합에서 상위 `n`건만 추출 |
| **표준 페이징** | `OFFSET n ROWS FETCH NEXT m ROWS ONLY` | • 앞의 `n`개 행을 건너뛰고(Skip) 그다음 `m`개 행을 반환<br>• 예: 6번째부터 5개 조회 시 `OFFSET 5 ROWS FETCH NEXT 5 ROWS ONLY` |
| **전통 Top-N (ROWNUM)** | `SELECT * FROM (SELECT ... ORDER BY ...) WHERE ROWNUM <= n;` | • `WHERE` 절이 `ORDER BY`보다 먼저 실행되므로 반드시 **인라인 뷰(서브쿼리)**로 먼저 정렬 후 외부에서 `ROWNUM` 적용 필수 |
