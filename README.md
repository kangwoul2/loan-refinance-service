# Loan Refinance Service

대환대출이 이득인지 총비용으로 계산하고, 금융상품 데이터를 동시 수정과 재시도에서도 일관되게 관리하려는 Java 17 / Spring Boot 백엔드 개인 프로젝트입니다.

- 기존 웹 데모(Next.js, 이전 단계): https://pay-off-loan.vercel.app/
- 이전 README의 설계 설명 전문: [docs/design/DESIGN_NOTES.md](docs/design/DESIGN_NOTES.md)
- 앞으로 할 일과 검증 계획: [docs/ROADMAP.md](docs/ROADMAP.md)

이 README는 "코드에 있는 것"과 "테스트나 측정으로 확인한 것"을 구분해서 적습니다.

## 현재 상태 (근거 표)

상태 표기: IMPLEMENTED = 코드가 있음, MEASURED = 측정 결과가 있음, PLANNED = 아직 안 함, NOT VERIFIED = 코드는 있으나 자동 검증이 없음.

| 항목 | 상태 | 근거와 한계 |
|---|---|---|
| BigDecimal 기반 총비용 계산 (원리금균등, 원금균등) | IMPLEMENTED | `simulation/` 패키지. 단위 테스트 2개 클래스(`EqualPrincipalCalculatorTest`, `RefinanceSimulationServiceTest`)가 있고, 원리금균등 계산기 단독 테스트는 없음 |
| 상품 금리 수정의 낙관적 락 (`@Version`, 충돌 시 409) | IMPLEMENTED / NOT VERIFIED | `LoanProduct`, `ProductService.changeRate`. 동시 요청 테스트가 없음. Spring이 던지는 `ObjectOptimisticLockingFailureException`은 핸들러에 없어 500이 날 수 있다는 의심이 있으나 재현하지 않음 |
| `Idempotency-Key` 기반 상품 적재 | IMPLEMENTED / NOT VERIFIED | `ProductService.importProduct`는 조회 후 저장 방식이라 동시에 같은 키가 들어오면 경쟁이 남음. 같은 키에 다른 본문이 오는 경우를 감지하지 않음. 동시성 테스트 없음 |
| `(bank_name, product_name)` 유일성 제약 | IMPLEMENTED | `V1__loan_service.sql`. 단 CI 테스트는 H2를 쓰므로 이 마이그레이션이 CI에서 적용되지 않음 |
| 상품 목록 캐시 (`@Cacheable` / `@CacheEvict`) | IMPLEMENTED / NOT VERIFIED | 수정과 적재 시 전체 무효화. 무효화 시점(커밋 전후)과 오래된 값 노출 가능성을 테스트하지 않음 |
| Redis 프로필 | IMPLEMENTED | `application-redis.yml`. 로컬 캐시와의 성능 비교는 하지 않음 |
| 실험 스크립트 (`experiments/`) | IMPLEMENTED | 스크립트만 있고 실행 결과를 저장하지 않았음. 따라서 MEASURED 항목 없음 |
| GitHub Actions CI | IMPLEMENTED | `mvn -B verify`만 실행. PostgreSQL과 Flyway를 쓰는 통합 테스트는 없음 |
| PostgreSQL + Testcontainers 통합 테스트 | PLANNED | ROADMAP 참고 |
| 동시성, 멱등성 재현 테스트 | PLANNED | ROADMAP 참고 |

성능 수치(p50, p95 등)는 측정한 적이 없으므로 이 문서에 적지 않습니다.

## 구조

```text
Next.js 화면(이전 단계)        Python 수집기(crawling/)
        │                          │
        └────────────┬─────────────┘
                     ▼
        Spring Boot (backend/)
          ├─ simulation: 총비용 계산, 상환 방식별 계산기
          └─ product   : 상품 조회, 금리 수정, 멱등 적재
                     │
              PostgreSQL (Flyway) / Redis 선택
```

- Spring Boot 백엔드(`backend/`)가 현재 개발 대상입니다.
- Next.js 화면과 Python 수집기는 이전 단계 코드이며 Spring API로 이전되지 않았습니다.

## 주요 API

| 메서드 | 경로 | 설명 |
|---|---|---|
| POST | `/api/v1/simulations` | 대환 전후 총비용, 절감액, 추천 여부 계산 |
| GET | `/api/v1/products` | 상품 목록 조회 (캐시 대상) |
| POST | `/api/v1/products/imports` | `Idempotency-Key` 헤더와 함께 상품 적재 |
| PATCH | `/api/v1/products/{id}/rate` | `expectedVersion`을 포함한 금리 수정 |

## 실행

```bash
cd backend
mvn spring-boot:run          # 기본 실행
mvn test                     # 단위 테스트
mvn verify                   # CI와 동일
mvn spring-boot:run -Dspring-boot.run.profiles=redis   # Redis 캐시 프로필
```

## 알려진 한계

1. 인증과 인가가 없습니다.
2. 실제 은행의 모든 수수료 정책을 반영한 계산 엔진이 아닙니다.
3. 수집 작업의 일정 실행과 재시도는 별도 작업 처리자로 분리하지 않았습니다.
4. 위 근거 표에서 NOT VERIFIED와 PLANNED로 표시한 항목은 아직 주장할 수 없는 내용입니다.

## 저장소 구성

- `backend/` Spring Boot 서버, `src/` Next.js 화면, `crawling/` Python 수집기
- `docs/` 설계 문서. `ARCHITECTURE_DECISIONS.md`, `EXPERIMENT_PLAN.md`, `INTERVIEW_GUIDE.md` 등
- `experiments/` 낙관적 락, 캐시 확인용 스크립트
