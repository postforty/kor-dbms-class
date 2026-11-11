# Oracle SQL 고급 객체(VIEW · SEQUENCE · INDEX · SYNONYM) 연습문제

본 문서는 [10_01_advanced_objects_guide.md](./10_01_advanced_objects_guide.md)에서 학습한 뷰(View), 시퀀스(Sequence), 인덱스(Index), 동의어(Synonym)의 핵심 문법과 원리를 점검하기 위한 연습문제입니다.

각 문제를 먼저 풀어본 후, 하단의 **정답 및 해설**을 펼쳐 올바른 해설과 오답 분석을 확인해 보세요.

---

## Q1. 뷰(View)의 특징과 제한사항

**다음 중** "뷰(View)"**에 대한 설명으로 옳지 않은 것은?**

① 뷰는 실제 데이터를 저장하지 않고, 정의된 SELECT 문을 통해 결과를 보여준다.  
② 뷰는 기본적으로 INSERT, UPDATE, DELETE 작업이 가능하다.  
③ 뷰 생성 시 WITH CHECK OPTION을 지정하면 뷰의 조건에 맞지 않는 데이터 변경을 차단할 수 있다.  
④ 뷰는 복잡한 조인 연산을 간단히 재사용할 수 있는 장점이 있다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② 뷰는 기본적으로 INSERT, UPDATE, DELETE 작업이 가능하다.**

### **💡 해설**
- **뷰**(View)**는 본질적으로 조회**(SELECT) **목적의 가상 테이블**:
  - 뷰는 물리적인 데이터를 디스크에 저장하지 않고, 뷰를 정의한 `SELECT` 문장(텍스트)만 데이터 딕셔너리에 보관하는 **논리적 객체**입니다.
  - 뷰를 통한 데이터 조작(DML: `INSERT`, `UPDATE`, `DELETE`)은 단일 기본 테이블로 구성된 극히 일부의 '단순 뷰(Simple View)'에서만 제한적으로 허용됩니다.
  - 조인(JOIN)을 포함한 복합 뷰, 그룹 집계 함수(`SUM`, `AVG`, `COUNT`), `GROUP BY`, `DISTINCT` 등이 포함된 뷰는 **기본적으로 DML 작업이 불가능**합니다.
  - 또한 실무에서는 뷰를 통한 데이터 변조 사고를 막기 위해 `WITH READ ONLY` 옵션을 명시하여 조회 전용으로 운영하는 것이 표준 원칙입니다.

### **❌ 오답 분석**
- **①**: 뷰는 실제 데이터 블록을 할당받지 않고, 데이터 딕셔너리에 저장된 `SELECT` 쿼리를 런타임에 기저 테이블과 결합(View Merging)하여 동적으로 결과를 생성하므로 올바른 설명입니다.
- **③**: `WITH CHECK OPTION`을 부여하면 뷰의 `WHERE` 절 조건을 만족하지 않는 데이터의 `INSERT` 및 `UPDATE`를 `ORA-01402` 오류로 원천 차단하므로 올바른 설명입니다.
- **④**: 4~5개 테이블의 복잡한 조인과 상관 서브쿼리를 하나의 뷰로 정의해 두면, 다른 개발자나 분석가가 단순한 `SELECT * FROM 뷰명;` 형태로 손쉽게 재사용할 수 있으므로 올바른 설명입니다.

</details>

---

## Q2. 시퀀스를 활용한 자동 증가 번호 적용

`KOREA_MALL` 스키마에서, `orders` 테이블의 `order_id` 컬럼에 대해 자동 증가 기능을 적용하려 한다. 적절한 SQL 구문은 무엇인가? (단, `seq_orders` 시퀀스는 이미 존재한다고 가정)

① `orders.order_id DEFAULT seq_orders.NEXTVAL`  
② `INSERT INTO orders(order_id…) VALUES (seq_orders.NEXTVAL, …)`  
③ `ALTER TABLE orders MODIFY order_id INT AUTOINCREMENT`  
④ `CREATE INDEX idx_orders_orderid ON orders(order_id)`  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### 정답: ② `INSERT INTO orders(order_id…) VALUES (seq_orders.NEXTVAL, …)`

### **💡 해설**
- **오라클 표준 시퀀스 번호 발급 방식**:
  - 오라클에서 기본키(PK) 등 컬럼에 순차적으로 증가하는 번호를 할당하는 가장 전통적이고 표준적인 방법은 `INSERT` 문장의 `VALUES` 절에 `시퀀스명.NEXTVAL` 의사 컬럼을 직접 지정하는 것입니다.
  ```sql
  -- 신규 주문 추가 시 시퀀스로 order_id 자동 발급
  INSERT INTO orders (order_id, customer_id, order_date, status)
  VALUES (seq_orders.NEXTVAL, 1, SYSDATE, '결제완료');
  ```

### **❌ 오답 분석**
- **①**: 완결된 SQL 구문이 아니며, 테이블 기본값으로 시퀀스를 바인딩하려면 `ALTER TABLE orders MODIFY (order_id DEFAULT seq_orders.NEXTVAL);`와 같이 유효한 DDL 문법을 갖추어야 합니다.
- ③ `AUTOINCREMENT`: MySQL/MariaDB에서 사용하는 자동 증가 속성(`AUTO_INCREMENT`) 문법입니다. 오라클에서는 지원하지 않는 키워드입니다.
- ④ `CREATE INDEX`: 인덱스는 데이터 검색 속도를 향상시키기 위한 색인 객체일 뿐, 일련번호를 자동으로 생성하는 기능과는 무관합니다.

</details>

---

## Q3. 인덱스 실행 계획(Execution Plan) 분석

**다음 SQL 문은 어떤 인덱스 구조를 사용하는지에 대한 실행계획 항목이다. 이 설명으로 옳은 것은?**

```text
01  Execution Plan
02  -------------------------------------
03  0  SELECT STATEMENT
04  1   INDEX RANGE SCAN  IDX_PRODUCTS_NAME
05  2    TABLE ACCESS BY INDEX ROWID  PRODUCTS
```

① 인덱스를 통해 검색하고, 그 결과를 기반으로 테이블 행을 읽는다.  
② FULL TABLE SCAN 방식으로 테이블 전체를 읽은 뒤, 인덱스를 사용한다.  
③ 인덱스가 없을 때의 실행계획이다.  
④ 인덱스는 검색에만 사용되고 테이블 액세스는 전혀 수행되지 않는다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ① 인덱스를 통해 검색하고, 그 결과를 기반으로 테이블 행을 읽는다.**

### **💡 해설**
- **실행 계획의 2단계 계층 구조 분석**:
  1. `1: INDEX RANGE SCAN (IDX_PRODUCTS_NAME)`:
     - 먼저 B-Tree 인덱스인 `IDX_PRODUCTS_NAME`의 색인 트리를 탐색하여, 조건절(`WHERE name = ...` 또는 `LIKE '문자%'`)에 부합하는 레코드의 물리적 주소값인 `ROWID` **목록**을 인덱스 리프 노드에서 빠르게 추출합니다.
  2. `2: TABLE ACCESS BY INDEX ROWID (PRODUCTS)`:
     - 1단계에서 획득한 `ROWID` 포인터를 사용하여 실제 `PRODUCTS` 테이블의 데이터 블록에 단 1회씩 직접 접근함으로써 필요한 나머지 컬럼 데이터들을 읽어옵니다.
  - 따라서 "인덱스를 통해 검색(`INDEX RANGE SCAN`)하고, 그 결과(`ROWID`)를 기반으로 테이블 행을 읽는다(`TABLE ACCESS BY INDEX ROWID`)"는 ①번 설명이 정확합니다.

### **❌ 오답 분석**
- **②**: `TABLE ACCESS FULL`이 발생하지 않았으며, 인덱스 검색을 먼저 수행한 뒤 테이블 블록을 읽습니다.
- **③**: 인덱스가 없을 때는 `TABLE ACCESS FULL`(전체 테이블 스캔)만 나타납니다.
- **④**: 2번 항목에서 보듯 테이블 데이터 블록 접근(`TABLE ACCESS BY INDEX ROWID`)이 명시적으로 수행되고 있습니다. (단, 조회 컬럼이 모두 인덱스에 포함된 커버링 인덱스의 경우에만 테이블 액세스가 생략됩니다.)

</details>

---

## Q4. 동의어(Synonym)의 개념과 보안 특성

**동의어**(Synonym)**에 관한 설명으로 옳은 것은?**

① 동의어는 실제 객체의 데이터를 복사하여 별도 저장한다.  
② PUBLIC SYNONYM은 모든 사용자가 접근 가능하므로 보안 리스크가 있다.  
③ 동의어는 뷰(View)처럼 조건을 가질 수 있다.  
④ 동의어가 없으면 스키마명을 생략할 수 없다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② PUBLIC SYNONYM은 모든 사용자가 접근 가능하므로 보안 리스크가 있다.**

### **💡 해설**
- **공용 동의어**(PUBLIC SYNONYM)의 **보안 고려사항**:
  - `PUBLIC SYNONYM`은 데이터베이스 내의 **모든 사용자**(PUBLIC)가 해당 객체 이름을 스키마 접두사 없이 바로 호출할 수 있도록 열어주는 전역 별칭입니다.
  - 모든 사용자에게 객체의 존재와 이름이 노출되므로, 적절한 권한(`REVOKE/GRANT`) 관리가 동반되지 않으면 비인가 사용자의 접근 시도나 SQL 인젝션 공격에 노출될 수 있어 실무에서는 공용 동의어 남용을 엄격히 통제합니다.

### **❌ 오답 분석**
- **①**: 동의어는 단순한 '객체의 별칭(Alias, 가명)'에 불과하며 실제 데이터를 물리적으로 복사하지 않습니다.
- **③**: 동의어는 객체의 이름만 대체할 뿐, `WHERE` 절 조건이나 계산식을 가질 수 없습니다. 조건이나 계산식이 필요한 가상화는 뷰(View)를 사용해야 합니다.
- **④**: 자신이 소유한 스키마 내부의 테이블은 동의어가 없어도 `SELECT * FROM products;`처럼 스키마명을 당연히 생략할 수 있습니다. (타인의 객체에 접근할 때만 스키마명이 필요함)

</details>

---

## Q5. 시퀀스 CYCLE 옵션의 순환 동작

**다음 시퀀스 옵션 중 최대값에 도달하면 다시 시작값으로 돌아가도록 하는 옵션은?**

① NOCACHE  
② CYCLE  
③ MAXVALUE  
④ INCREMENT  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### **정답: ② CYCLE**

### **💡 해설**
- `CYCLE` **옵션의 순환 메커니즘**:
  - 시퀀스를 생성할 때 `CYCLE` 옵션을 부여하면, 일련번호가 지정된 `MAXVALUE`(최댓값)에 도달했을 때 번호 발급을 멈추거나 에러를 내지 않고, 다음 호출 시 다시 시작값(`MINVALUE`, 기본값 1)으로 리셋되어 순환 발급됩니다.
  ```sql
  -- 1~3까지 발급 후 다시 1로 순환하는 시퀀스
  CREATE SEQUENCE seq_product_serial
      START WITH 1
      INCREMENT BY 1
      MAXVALUE 3
      CYCLE
      NOCACHE;
  ```

### **❌ 오답 분석**
- ① `NOCACHE`: 메모리(SGA)에 번호를 미리 할당해 두지 않고 매번 디스크를 갱신하도록 하는 캐시 제어 옵션입니다.
- ③ `MAXVALUE`: 시퀀스가 발급할 수 있는 최댓값을 지정하는 옵션입니다.
- ④ `INCREMENT`: 번호가 증가하는 단위(`INCREMENT BY`)를 지정하는 옵션입니다.

</details>

