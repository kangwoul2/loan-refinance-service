# ROADMAP

각 항목은 "무엇을 확인하면 완료인지"를 기준으로 적습니다. 완료 전에는 README 근거 표에서 PLANNED 또는 NOT VERIFIED로 둡니다.
PR 하나에는 항목 하나만 담고, 실패하는 테스트 커밋과 수정 커밋을 분리합니다.

| 번호 | 항목 | 완료 기준 | 현재 상태 |
|---|---|---|---|
| A1 | PostgreSQL + Testcontainers 통합 테스트 기반 | CI에서 실제 PostgreSQL에 Flyway V1이 적용된 상태로 테스트가 돈다 | PLANNED |
| A2 | 같은 `Idempotency-Key` 동시 요청 | N개 동시 요청 후 상품 행이 1개이고 응답이 모두 같은 결과를 가리킨다. 현재 조회 후 저장 구조에서는 실패할 것으로 예상 | PLANNED |
| A3 | 같은 키에 다른 본문 | 본문 해시를 비교해 충돌로 응답한다 (예: 409 또는 422) | PLANNED |
| B1 | 낙관적 락 예외 매핑 | `ObjectOptimisticLockingFailureException`도 409로 응답한다. 먼저 500이 나는지 재현한다 | PLANNED |
| B2 | 동시 금리 수정 | 같은 version으로 N개 요청 시 성공 1건, 나머지 409, 최종 version 일치 | PLANNED |
| B3 | 캐시 무효화 순서 | 커밋 이후에 무효화되는지, 커밋 전 무효화로 오래된 값이 다시 캐시되지 않는지 확인 | PLANNED |
| C | 원리금균등 계산기 단독 테스트 | 경계값(0% 금리, 1개월 등) 포함 | PLANNED |
| D | 실험 스크립트 결과 기록 | 실행 환경, 횟수, 결과를 `docs/`에 저장. 저장 전까지 성능 수치는 주장하지 않는다 | PLANNED |

Spring Boot 4.1.1과 Testcontainers 등 추가 의존성의 호환성은 도입 시점에 확인합니다 (미확인).
