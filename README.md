# Oracle AI Database 실습 과정 (`kor-dbms-class`)

Oracle AI Database 26ai와 DBeaver를 활용한 관계형 데이터베이스(RDBMS) 및 SQL 실습 강의 저장소입니다.  
가상 이커머스 도메인인 `KOREA_MALL`의 6대 핵심 테이블을 기반으로 DDL/DML 기초부터 고급 쿼리까지 단계별로 학습합니다.

---

## 🛠️ 실습 환경 및 접속 정보

| 항목 | 설정값 |
| :--- | :--- |
| **DBMS** | Oracle AI Database 26ai |
| **Tool** | DBeaver ([설치 및 설정 가이드](./dbeaver_guide.md)) |
| **Host / Port** | `localhost` / `1521` |
| **Database / Service** | `FREEPDB1` |
| **실습 계정 / 암호** | `KOREA_MALL` / `1234` |
| **Active Schema** | `KOREA_MALL` |

> ⚠️ **주의**: 사용자 생성 및 권한 설정은 `SYSTEM` 관리자 세션에서 진행하며, 테이블 생성(DDL) 및 데이터 조작/조회(DML/SELECT)는 실습 전용 세션인 `KOREA_MALL` 편집기에서 실행합니다.

---

## 🗂️ 실습 도메인 모델 (`KOREA_MALL`)

쇼핑몰 도메인의 **6대 핵심 테이블**을 중심으로 실습을 진행합니다.  
전체 테이블 스키마 및 샘플 데이터 세트는 [mentormall_db_sample](https://github.com/lecshjin/mentormall_db_sample) 저장소에서 확인할 수 있습니다.

- `customers`: 고객 정보
- `categories`: 상품 카테고리 정보
- `products`: 상품 마스터 정보
- `orders`: 주문 기본 정보
- `order_items`: 주문 상세 품목 정보
- `reviews`: 상품 리뷰 및 평점 정보

---

## 📚 커리큘럼 및 실습 목차

전체 강의 일정과 세부 계획은 [curriculum.md](./curriculum.md)에서 확인할 수 있습니다.

| 차수 | 주제 | 실습 자료 링크 |
| :---: | :--- | :--- |
| **01** | 환경 구축 및 개론 | [DBeaver 가이드](./dbeaver_guide.md) |
| **02** | 데이터정의어 (DDL) | [계정 설정](./sec02/02_01_user_setup.md) · [DDL 가이드](./sec02/02_02_ddl_guide.md) · [DDL 실습](./sec02/02_03_ddl_mission.md) · [트러블슈팅](./sec02/02_04_troubleshooting_ora_01031.md) |
| **03** | 데이터 조회 (SELECT) | [조회 가이드](./sec03/03_01_select_guide.md) · [조회 실습](./sec03/03_02_select_exercise.md) · [더미 데이터](./sec03/dummy_data.sql) |
| **04** | 데이터 조작어 (DML) | [DML 가이드](./sec04/04_01_dml_guide.md) · [DML 실습](./sec04/04_02_dml_exercise.md) |
| **05** | 정렬 및 결과 제한 | [정렬 가이드](./sec05/05_01_order_by_guide.md) · [정렬 실습](./sec05/05_02_order_by_exercise.md) |
| **06** | 제어문 및 단일행함수 | [함수 가이드](./sec06/06_01_function_guide.md) · [함수 실습](./sec06/06_02_function_exercise.md) |
| **07** | 그룹함수 및 집계 | [그룹함수 가이드](./sec07/07_01_group_function_guide.md) · [그룹함수 실습](./sec07/07_02_group_function_exercise.md) |
| **08** | 다중 테이블 (JOIN) | [조인 가이드](./sec08/08_01_join_guide.md) · [조인 실습](./sec08/08_02_join_exercise.md) |
| **09** | 서브쿼리 (Subquery) | [서브쿼리 가이드](./sec09/09_01_subquery_guide.md) · [서브쿼리 실습](./sec09/09_02_subquery_exercise.md) |
| **10** | 고급 활용 (INDEX/VIEW) | [고급 객체 가이드](./sec10/10_01_advanced_objects_guide.md) · [고급 객체 연습문제](./sec10/10_02_advanced_objects_exercise.md) · [트랜잭션 실습](./sec10/10_03_transaction_practice.md) · [트랜잭션 연습문제](./sec10/10_04_transaction_exercise.md) |

---

## 🚀 빠른 시작 가이드

1. **도구 설치 및 접속**: [DBeaver 가이드](./dbeaver_guide.md)를 따라 Oracle 및 DBeaver 연결 설정을 완료합니다.
2. **실습 계정 생성**: `SYSTEM` 세션에서 [계정 설정 가이드](./sec02/02_01_user_setup.md)의 쿼리를 실행하여 `KOREA_MALL` 계정을 생성합니다.
3. **테이블 생성**: `KOREA_MALL` 세션으로 전환 후 [DDL 가이드](./sec02/02_02_ddl_guide.md)에 따라 6대 핵심 테이블을 생성합니다.
4. **실습 데이터 세팅**: `KOREA_MALL` 세션에서 [더미 데이터 스크립트](./sec03/dummy_data.sql)를 실행하여 기초 데이터를 적재합니다.
5. **학습 진행**: 각 섹션별 가이드와 실습 문제를 순서대로 진행합니다.
