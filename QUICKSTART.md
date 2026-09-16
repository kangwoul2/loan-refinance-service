# 빠른 시작

이 문서는 기존 Python 데이터 수집 기능을 실행하는 방법을 정리합니다. Java/Spring 백엔드 실행 방법은 `README.md`를 참고합니다.

## 1. Python 가상환경 만들기

### Windows

```bash
python -m venv venv
venv\Scripts\activate
```

### macOS / Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

## 2. 패키지 설치

```bash
pip install -r requirements.txt
```

## 3. Supabase 설정

1. Supabase 프로젝트 생성
2. SQL 편집기에서 `database_schema.sql` 실행
3. `.env.example`을 `.env`로 복사
4. 프로젝트 URL과 API 키 설정

```text
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=...
```

## 4. 데이터 수집 실행

```bash
python -m crawling.main
```

실행 과정에서는 다음 순서로 처리됩니다.

```text
외부 금융상품 데이터 수집
  ↓
필수 값 검사
  ↓
금리 범위 검사
  ↓
중복 제거
  ↓
DB 적재
```

## 5. 데이터 확인

Supabase의 `loan_products` 테이블에서 적재 결과를 확인합니다.

## 6. Git 사용

현재 상태 확인:

```bash
git status
git log --oneline
```

변경 후 커밋 예시:

```bash
git add crawling/bank_crawlers/kb_crawler.py
git commit -m "fix: KB 수집 로직 수정"
```

## 7. 자주 발생하는 문제

### Chrome 드라이버 오류

```bash
pip install --upgrade webdriver-manager
```

### Supabase 연결 오류

- `.env`의 URL과 API 키 확인
- Supabase 프로젝트 활성 상태 확인

### 패키지 설치 오류

가상환경을 다시 만든 뒤 패키지를 재설치합니다.

## 8. 프로젝트 설명 기준

데이터 수집은 사용자 요청 경로와 분리합니다. 외부 사이트는 지연이나 실패가 발생할 수 있기 때문에 사용자 API가 수집 완료까지 기다리도록 만들지 않습니다.

재시도가 필요한 저장 요청은 Spring 상품 적재 API의 멱등성 처리와 연결하는 방향으로 확장할 수 있습니다.