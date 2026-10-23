# AI Agent 지침서 (AGENTS.md)

본 저장소(`kor-dbms-class`)의 모든 실습 문서, SQL 스크립트 작성 및 수정 시 적용되는 표준 가이드라인입니다.

---

## 1. 표준 스키마 & 계정 (`KOREA_MALL`)
* **표준 명칭**: **`KOREA_MALL`** (대문자 원칙, 이전 명칭 `MANTORMALL`, `MENTORMALL` 사용 금지)
* **USER = SCHEMA**: Oracle 특성에 따라 일반 사용자 생성으로 실습 스키마가 구성됩니다.
  ```sql
  -- SYSTEM 관리자 세션에서 실행
  CREATE USER KOREA_MALL IDENTIFIED BY "1234";
  GRANT CONNECT, RESOURCE TO KOREA_MALL;
  GRANT UNLIMITED TABLESPACE TO KOREA_MALL;
  ```

---

## 2. 작업 세션 분리 및 접속 정보
* **관리자(`SYSTEM`) 세션**: 사용자 생성 및 인프라 권한 설정 전용
* **실습용(`KOREA_MALL`) 세션**: 모든 DDL/DML/트랜잭션 실습 진행 (카탈로그 오염 방지)
* **DBeaver 표준 연결값**:
  * **Host**: `localhost` | **Port**: `1521` | **Service**: `FREEPDB1`
  * **Username / Password**: `KOREA_MALL` / `1234`
  * **Active Schema**: `KOREA_MALL`

---

## 3. 실습 도메인 모델
* 전 섹션(sec01~10) 공통 6대 핵심 테이블 기준으로 실습 진행:
  * `customers`, `categories`, `products`, `orders`, `order_items`, `reviews`

---

## 4. 에이전트 준수 체크리스트
- [ ] 스키마/계정명이 오직 **`KOREA_MALL`**로 통일되어 있는가?
- [ ] 실습 쿼리는 활성 스키마 기반으로 **접두사 없이 테이블명(예: `FROM customers`)을 간결하게 작성**했는가?
- [ ] DDL/DML 실습을 일반 세션(`KOREA_MALL`) 편집기에서 실행하도록 명시했는가?
- [ ] 내부 파일 참조 링크는 `[filename](file:///절대경로)` 형식(슬래시 `/`)을 준수하는가?
