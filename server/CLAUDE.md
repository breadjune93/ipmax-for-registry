# IPMax Server — Claude Context

> Linux 서버 (Spring Boot). 루트 `CLAUDE.md`를 먼저 읽고 이 문서를 본다.

## 역할

IPMax의 **중심 코어**. REST API, 어드민 웹 콘솔(HTML + Vanilla JS), 세션 관리, 정산 파이프라인.

## 스택

- **Java / Kotlin(선택 후 고정)** + **Spring Boot**
- **PostgreSQL** (원장/유저)
- **Redis** (세션·카운터·캐시)
- **Kafka** (이벤트 버스)
- **ELK** (로그·분석)
- **프론트**: HTML + Vanilla JS (서버 렌더링 or 정적 파일)
- **빌드**: Gradle 권장

## 디렉토리 제안

```
server/
├── build.gradle(.kts)
├── gradle/
├── src/
│   ├── main/
│   │   ├── java|kotlin/com/ipmax/
│   │   │   ├── IpmaxApplication.*
│   │   │   ├── config/
│   │   │   ├── auth/
│   │   │   ├── wallet/
│   │   │   ├── endpoint/
│   │   │   ├── session/
│   │   │   ├── billing/        ← 정산 로직 (민감, 테스트 필수)
│   │   │   ├── provider/       ← OpenWrt heartbeat 수신
│   │   │   └── admin/          ← 어드민 콘솔 컨트롤러
│   │   ├── resources/
│   │   │   ├── application.yml
│   │   │   ├── db/migration/   ← Flyway
│   │   │   └── static/         ← HTML/CSS/JS (어드민 콘솔)
│   └── test/
└── docker-compose.yml          ← 로컬 개발용 (pg/redis/kafka/elk)
```

## 주요 원칙

1. **정산/과금 로직은 반드시 단위 테스트 + 재현 가능한 픽스처 동반**
2. 금액은 `BigDecimal` 또는 정수(최소 단위). `double` 금지
3. 모든 외부 입력에 **유효성 검증** (Bean Validation)
4. DB 접근은 **리포지토리 레이어를 통해서만** — 서비스·컨트롤러에서 직접 쿼리 금지
5. **OpenAPI 스펙(`../docs/api-contracts/openapi.yaml`)이 계약** — 컨트롤러는 이것을 따른다. 큰 변경 시 스펙부터 수정
6. 로그는 JSON 구조화, PII 마스킹. 트레이스 ID 필수
7. 마이그레이션(Flyway) 순번 추가 시 반드시 롤백 시나리오 검토

## 실행 (로컬)

```bash
# 인프라 기동
docker compose up -d postgres redis kafka elasticsearch kibana

# 앱 실행
./gradlew bootRun --args='--spring.profiles.active=local'
```

## 공통 커맨드 (Claude가 자주 쓰는)

```bash
./gradlew build                 # 전체 빌드
./gradlew test                  # 전체 테스트
./gradlew test --tests "*Billing*"   # 정산 모듈만
./gradlew flywayMigrate         # DB 마이그레이션
```

## API 추가 절차

1. `../docs/api-contracts/openapi.yaml` 먼저 업데이트 (엔드포인트·스키마)
2. 컨트롤러/서비스 구현
3. 통합 테스트(가급적 `@SpringBootTest` + Testcontainers)
4. 큰 의미 변경 시 ADR 추가

## 해서는 안 되는 일

- `application-*.yml`에 비밀값 하드코딩
- 정산 레코드 직접 `UPDATE` (반드시 감사 가능한 트랜잭션 로직으로)
- Kafka consumer에서 예외 무시 — 재처리 또는 DLQ로 보낼 것
- 관리자 엔드포인트를 일반 JWT만으로 보호 — **MFA + IP 화이트리스트**

## 자주 참조할 문서

- [시스템 개요](../docs/architecture/system-overview.md)
- [데이터 모델](../docs/architecture/data-model.md)
- [정산 구조 ADR](../docs/business/decisions/0002-초단위-정산-구조.md)
