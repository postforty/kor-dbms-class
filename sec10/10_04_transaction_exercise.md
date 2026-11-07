# Oracle SQL 트랜잭션 및 동시성 제어(TCL · ACID · LOCK) 연습문제

본 문서는 [10_01_advanced_objects_guide.md](./10_01_advanced_objects_guide.md) 및 [10_03_transaction_practice.md](./10_03_transaction_practice.md)에서 학습한 트랜잭션의 개념, ACID 원칙, TCL 명령어(`COMMIT`, `ROLLBACK`, `SAVEPOINT`), 트랜잭션 격리 수준, 그리고 락(Lock Wait) 메커니즘의 핵심 원리를 점검하기 위한 연습문제입니다.

각 문제를 먼저 풀어본 후, 하단의 **정답 및 해설**을 펼쳐 올바른 개념과 오답 분석을 확인해 보세요.

---

## Q1. 트랜잭션의 정의에 대한 설명

**트랜잭션의 정의에 대한 설명으로 옳은 것은?**

① 여러 SQL 문을 하나의 논리적 단위로 묶어 처리하는 것을 의미한다.  
② SQL 문이 오류 없이 수행되면 자동으로 COMMIT된다.  
③ 트랜잭션은 DDL 문을 기준으로 자동 시작된다.  
④ 트랜잭션이 종료되면 ROLLBACK을 할 수 없다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ① 여러 SQL 문을 하나의 논리적 단위로 묶어 처리하는 것을 의미한다.</b>

### 💡 해설
- **트랜잭션의 본질적 정의**(Logical Unit of Work):
  - 트랜잭션(Transaction)은 관계형 데이터베이스에서 '더 이상 분할할 수 없는 하나의 논리적 작업 단위'를 의미합니다.
  - 예를 들어 쇼핑몰 주문 처리 시 `orders` 삽입, `order_items` 삽입, `products` 재고 차감이라는 여러 개의 개별 SQL 문이 모두 성공하거나 모두 취소되어야 하나의 온전한 비즈니스 작업으로 완성됩니다.

### ❌ 오답 분석
- **②**: 오라클의 기본 트랜잭션 모드(수동 커밋)에서는 SQL 문이 에러 없이 정상 실행되었다 하더라도 사용자가 명시적으로 `COMMIT;`을 입력하거나 DDL을 실행하기 전까지는 커밋되지 않습니다.
- **③**: 트랜잭션은 DDL이 아니라 DML(`INSERT`, `UPDATE`, `DELETE`) 문이 세션에서 최초로 실행되는 순간 암시적으로 자동 시작됩니다.
- **④**: 트랜잭션이 종료된 후에는 롤백할 작업 자체가 남아있지 않다는 기술적 사실은 맞으나, 트랜잭션의 본질적 '정의'를 묻는 질문에는 하나의 논리적 작업 단위를 뜻하는 ①번이 가장 올바른 설명입니다.

</details>

---

## Q2. TCL(트랜잭션 제어어) 구분

**다음 중 TCL**(트랜잭션 제어어)**에 해당하지 않는 것은?**

① COMMIT  
② ROLLBACK  
③ SAVEPOINT  
④ DELETE  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ④ DELETE</b>

### 💡 해설
- **SQL 명령어 분류 체계**:
  - **TCL**(Transaction Control Language): 트랜잭션의 작업 결과를 영구 반영(`COMMIT`), 취소(`ROLLBACK`), 또는 중간 저장점 설정(`SAVEPOINT`, `ROLLBACK TO`)하는 트랜잭션 제어어입니다.
  - **DML**(Data Manipulation Language): 테이블 내부의 행 데이터를 조회 및 조작하는 명령어로 `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` 등이 속합니다.
- 따라서 ④번 `DELETE`는 트랜잭션 제어어(TCL)가 아니라 **데이터 조작어**(DML)에 해당합니다.

### ❌ 오답 분석
- **①** `COMMIT`: 현재 트랜잭션의 모든 변경 사항을 데이터베이스 디스크에 영구 확정 반영하는 대표적인 TCL입니다.
- **②** `ROLLBACK`: 현재 트랜잭션에서 수행한 모든 데이터 변경 작업을 취소하고 트랜잭션 시작 시점으로 되돌리는 TCL입니다.
- **③** `SAVEPOINT`: 트랜잭션 내부에서 특정 시점을 북마크처럼 지정하여 부분 롤백을 가능하게 해주는 TCL입니다.

</details>

---

## Q3. Oracle에서 COMMIT이 수행되는 경우

**다음 중 Oracle에서 COMMIT이 수행되는 경우로 옳지 않은 것은?**

① 사용자가 명시적으로 COMMIT을 입력했을 때  
② DDL 문이 실행될 때  
③ Auto Commit이 활성화된 상태에서 DML 문이 실행되었을 때  
④ SQL 편집기에서 SELECT 문을 실행했을 때  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ④ SQL 편집기에서 SELECT 문을 실행했을 때</b>

### 💡 해설
- `SELECT` **문의 특성**(DQL / 조회 전용):
  - `SELECT` 문은 데이터베이스의 데이터를 조회만 할 뿐 데이터를 변경하지 않는 질의문(DQL)입니다.
  - 따라서 `SELECT` 문은 트랜잭션을 시작하거나, 커밋하거나, 롤백하는 어떠한 트랜잭션 상태 변화도 일으키지 않습니다.

### ❌ 오답 분석
- **① 명시적 커밋**: 사용자가 `COMMIT;` 명령어를 직접 실행하면 즉시 트랜잭션이 디스크에 영구 반영됩니다.
- **② 암시적 커밋**(DDL 실행): `CREATE`, `ALTER`, `DROP`, `TRUNCATE` 등의 DDL 문장을 실행하면 오라클 엔진이 내부적으로 직전까지의 모든 DML 작업을 자동으로 `COMMIT` 처리합니다.
- **③ 자동 커밋**(Auto-Commit): 클라이언트 환경(DBeaver 등)에서 Auto Commit 모드가 활성화되어 있으면 각 DML 문장이 실행될 때마다 즉시 자동 커밋됩니다.

</details>

---

## Q4. 시퀀스 채번 후 ROLLBACK 실행 결과

**다음 SQL 실행 결과에 대한 설명으로 옳은 것은?**

```sql
01 INSERT INTO orders (order_id, customer_id, order_date, status)
02 VALUES (seq_orders.NEXTVAL, 8, SYSDATE, '결제완료');
03 ROLLBACK;
```

① 주문 데이터는 데이터베이스에 영구 저장된다.  
② 주문 데이터는 취소되어 데이터베이스에 반영되지 않는다.  
③ 주문 데이터와 상관없이 시퀀스 값은 되돌아간다.  
④ 주문 데이터는 ROLLBACK 후에도 수정 가능하다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ② 주문 데이터는 취소되어 데이터베이스에 반영되지 않는다.</b>

### 💡 해설
- `ROLLBACK`**의 데이터 원복 원리**:
  - `INSERT` 문장으로 추가된 주문 데이터는 `COMMIT`되기 전까지 임시 상태(Undo 영역에 변경 전 상태 보관)로 존재합니다.
  - 직후 `ROLLBACK;`을 호출하면 해당 트랜잭션의 변경 사항이 모두 무효화되므로 `orders` 테이블에는 해당 데이터가 반영되지 않고 완전히 사라집니다.

### ❌ 오답 분석
- **①**: `ROLLBACK`이 실행되었으므로 영구 저장되지 않고 취소됩니다.
- **③**: **단, 시퀀스 객체는 트랜잭션 롤백의 영향을 받지 않습니다.** 동시성 처리 성능을 위해 시퀀스가 발행한 `NEXTVAL` 번호는 롤백되더라도 이전 번호로 되돌아가지 않으며 결번(Gap)으로 남습니다.
- **④**: 롤백된 데이터는 테이블에서 완전히 삭제/취소되었으므로 롤백 후 수정(`UPDATE`)할 대상 행 자체가 존재하지 않습니다.

</details>

---

## Q5. 트랜잭션 격리 수준(Isolation Level)의 특성

**다음 중 트랜잭션 격리 수준**(Isolation Level)**에 대한 설명으로 틀린 것은?**

① READ COMMITTED는 커밋된 데이터만 읽을 수 있다.  
② Oracle의 기본 격리 수준은 READ COMMITTED이다.  
③ SERIALIZABLE은 모든 트랜잭션을 순차적으로 실행한 것과 같은 효과를 가진다.  
④ READ COMMITTED는 Non-repeatable Read가 발생하지 않는다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ④ READ COMMITTED는 Non-repeatable Read가 발생하지 않는다.</b>

### 💡 해설
- `READ COMMITTED`**와 Non-repeatable Read**:
  - `READ COMMITTED`는 오직 '커밋이 완료된 최신 데이터'만 조회하도록 보장하는 격리 수준입니다.
  - 따라서 트랜잭션 A가 동일한 행을 두 번 조회하는 도중에, 트랜잭션 B가 해당 행을 수정하고 `COMMIT`하면 트랜잭션 A의 첫 번째 조회 결과와 두 번째 조회 결과가 서로 달라지는 **반복 불가능한 읽기**(Non-repeatable Read) 현상이 **발생할 수 있습니다**.
  - Non-repeatable Read를 방지하려면 `REPEATABLE READ` 또는 `SERIALIZABLE` 수준을 적용해야 합니다.

### ❌ 오답 분석
- **①**: `READ COMMITTED`는 커밋되지 않은 미완료 데이터(Dirty Data)를 읽지 않고 오직 커밋된 데이터만 읽으므로 올바른 설명입니다.
- **②**: 오라클 데이터베이스의 기본(Default) 트랜잭션 격리 수준은 `READ COMMITTED`이므로 올바른 설명입니다.
- **③**: `SERIALIZABLE`은 가장 높은 격리 수준으로, 여러 트랜잭션이 병행 실행되더라도 마치 한 번에 하나씩 직렬(순차적)로 실행된 것과 동일한 완벽한 고립성을 보장하므로 올바른 설명입니다.

</details>

---

## Q6. 미커밋 데이터 조회 현상 (동시성 이상 현상)

**아래의 설명이 나타내는 현상은 무엇인가?**

> 트랜잭션 A가 데이터를 수정하는 동안,  
> 트랜잭션 B가 아직 COMMIT되지 않은 그 데이터를 조회하여 사용하는 현상  

① Dirty Read  
② Non-repeatable Read  
③ Phantom Read  
④ Deadlock  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ① Dirty Read</b>

### 💡 해설
- **Dirty Read**(더티 리드 / 오독):
  - 아직 `COMMIT`되지 않아 언제든지 `ROLLBACK`될 수 있는 임시 변경 데이터를 다른 트랜잭션이 조회하여 참조하는 이상 현상입니다.
  - 오라클은 멀티 버전 동시성 제어(MVCC) 및 Undo 세그먼트 메커니즘을 통해 변경 중인 데이터 대신 변경 전의 커밋된 스냅샷을 제공하므로, 기본적으로 Dirty Read가 구조적으로 발생하지 않습니다.

### ❌ 오답 분석
- **② Non-repeatable Read**(반복 불가능한 읽기): 한 트랜잭션 내에서 같은 데이터를 다시 조회했을 때 다른 트랜잭션의 `UPDATE/DELETE` 커밋으로 인해 데이터 값이 달라지는 현상입니다.
- **③ Phantom Read**(유령 행 읽기): 한 트랜잭션 내에서 동일한 범위 조건으로 다시 조회했을 때 다른 트랜잭션의 `INSERT` 커밋으로 인해 이전에 없던 새로운 행이 나타나는 현상입니다.
- **④ Deadlock**(교착 상태): 둘 이상의 트랜잭션이 서로가 점유한 락이 해제되기를 무한히 대기하여 작업이 영구 중단되는 현상입니다.

</details>

---

## Q7. Oracle의 기본 트랜잭션 격리 수준

**다음 중 Oracle의 기본 트랜잭션 격리 수준은?**

① READ UNCOMMITTED  
② READ COMMITTED  
③ REPEATABLE READ  
④ SERIALIZABLE  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ② READ COMMITTED</b>

### 💡 해설
- **오라클의 기본 격리 수준과 지원 체계**:
  - Oracle Database의 기본 격리 수준은 `READ COMMITTED`입니다.
  - 오라클 엔진은 공식적으로 ANSI/ISO SQL 표준 격리 수준 중 `READ COMMITTED`와 `SERIALIZABLE`, 그리고 오라클 전용의 `READ ONLY` 모드 3가지만 지원합니다.
  - 오라클에서는 다른 세션의 미커밋 변경 데이터를 읽는 `READ UNCOMMITTED` 수준은 데이터 무결성을 위해 아예 지원하지 않습니다.

### ❌ 오답 분석
- **①** `READ UNCOMMITTED`: 오라클에서는 지원하지 않는 격리 수준입니다.
- **③** `REPEATABLE READ`: MySQL(InnoDB) 등의 기본 격리 수준이며, 오라클에서는 별도로 제공하지 않습니다(대신 `SERIALIZABLE` 제공).
- **④** `SERIALIZABLE`: 오라클에서 지원하지만 기본값이 아니며, 필요 시 `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;` 명령으로 명시적 전환해야 합니다.

</details>

---

## Q8. DDL 실행 시 트랜잭션 처리 방식

**다음 중 DDL 문 실행 시의 트랜잭션 처리 방식으로 옳은 것은?**

① 현재 트랜잭션에 포함되어 함께 커밋된다.  
② 현재 트랜잭션을 종료하고 자동으로 COMMIT된다.  
③ 현재 트랜잭션은 ROLLBACK된다.  
④ DDL은 트랜잭션의 영향을 받지 않는다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ② 현재 트랜잭션을 종료하고 자동으로 COMMIT된다.</b>

### 💡 해설
- **DDL 문의 암시적 커밋**(Implicit Auto-Commit) **동작**:
  - `CREATE`, `ALTER`, `DROP`, `TRUNCATE` 등의 DDL 문장은 데이터 사전(Data Dictionary)의 메타데이터를 직접 수정하는 특성을 가집니다.
  - 데이터 딕셔너리의 일관성을 지키기 위해 오라클 엔진은 **DDL 문장을 실행하기 직전에 암시적** `COMMIT`**을 수행하여 이전 트랜잭션을 강제로 정상 종료**시키고, DDL 실행 후에도 다시 한 번 암시적 `COMMIT`을 발생시킵니다.
  - 따라서 DDL이 실행되면 진행 중이던 트랜잭션은 종료되며 이전 작업들은 영구 반영됩니다.

### ❌ 오답 분석
- **①**: DDL 자체가 트랜잭션에 포함되는 것이 아니라, DDL 실행 전후로 자동 커밋이 발생하여 이전 DML 트랜잭션이 확정 종료되는 것입니다.
- **③**: DDL 실행 시 롤백되지 않고 무조건 커밋됩니다.
- **④**: 이전 DML 작업들이 함께 영구 저장되므로 트랜잭션에 강력한 영향을 미칩니다.

</details>

---

## Q9. 동일 행 수정 시 동시성 충돌 현상

**다음 상황에서 발생할 수 있는 문제로 옳은 것은?**

> 트랜잭션 A가 특정 행을 수정하고 아직 커밋하지 않은 상태에서  
> 트랜잭션 B가 동일한 행을 수정하려고 할 때 발생하는 현상  

① Phantom Read  
② Lock Wait  
③ Dirty Read  
④ SAVEPOINT  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ② Lock Wait</b>

### 💡 해설
- **행 락**(Row-level Lock)**과 락 대기**(Lock Wait):
  - 트랜잭션 A가 `UPDATE` 문으로 특정 행의 데이터를 수정하면 데이터베이스는 해당 행에 대해 다른 세션이 변경하지 못하도록 **배타적 행 락**(Exclusive Row Lock)을 자동으로 설정합니다.
  - 이때 트랜잭션 B가 동일한 행을 `UPDATE`하거나 `DELETE`하려고 시도하면, 트랜잭션 A가 `COMMIT` 또는 `ROLLBACK`을 실행하여 락을 해제할 때까지 트랜잭션 B는 아무런 작업을 진행하지 못하고 멈춰 서서 기다리는 **Lock Wait**(락 대기) 상태에 빠집니다.

### ❌ 오답 분석
- **①** `Phantom Read`: 조회 범위 조건 사이에 다른 트랜잭션이 신규 행을 `INSERT`하여 발생하는 조회 이상 현상입니다.
- **③** `Dirty Read`: 미커밋된 변경 데이터를 조회하는 현상으로, 오라클에서는 기본 격리 수준에서도 발생하지 않습니다.
- **④** `SAVEPOINT`: 트랜잭션 내에서 부분 취소를 위해 설정하는 중간 북마크 지점입니다.

</details>

---

## Q10. 트랜잭션의 4대 특성(ACID)

**다음 중 트랜잭션의 4대 특성**(ACID)**에 대한 설명으로 옳지 않은 것은?**

① Atomicity - 전부 수행되거나 전혀 수행되지 않는다.  
② Consistency - 트랜잭션 실행 전후 데이터의 일관성이 유지된다.  
③ Isolation - 동시에 수행되면 항상 직렬화된다.  
④ Durability - 커밋된 변경 내용은 영구적으로 보존된다.  

<details>
<summary>🔍 정답 및 해설 확인</summary>

### <b>정답: ③ Isolation - 동시에 수행되면 항상 직렬화된다.</b>

### 💡 해설
- **고립성**(Isolation)**의 올바른 의미**:
  - **고립성**(Isolation)의 일반적인 정의는 '동시에 여러 트랜잭션이 실행될 때, 각 트랜잭션이 서로 간섭하지 않고 독립적으로 수행되어야 한다'는 원칙입니다.
  - '동시에 수행되면 항상 직렬화(완전 순차 실행)된다'는 설명은 고립성을 실현하는 여러 격리 수준 중 가장 극단적인 최상위 수준인 `SERIALIZABLE`에만 해당하는 설명입니다.
  - 실제 대부분의 RDBMS(오라클 포함)는 동시 처리량과 성능을 확보하기 위해 동시 실행을 폭넓게 허용하면서 일관성을 제어하는 `READ COMMITTED` 수준을 기본으로 채택합니다.

### ❌ 오답 분석
- **① Atomicity**(원자성): All or Nothing. 트랜잭션 내의 모든 작업이 완벽히 수행되거나, 하나라도 실패하면 전혀 수행되지 않은 상태로 되돌아가야 하므로 올바른 설명입니다.
- **② Consistency**(일관성): 트랜잭션 수행 전후에 기본키, 외래키, 도메인 제약조건 등 데이터베이스의 무결성 규칙이 항상 일관되게 만족되어야 하므로 올바른 설명입니다.
- **④ Durability**(지속성 / 영속성): 성공적으로 `COMMIT`된 트랜잭션의 결과는 정전, 시스템 다운, 디스크 장애 등의 재난 상황이 발생하더라도 데이터베이스 로그를 통해 영구적으로 보존되어야 하므로 올바른 설명입니다.

</details>
