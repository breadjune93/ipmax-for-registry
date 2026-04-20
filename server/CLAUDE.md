# IPMax Server — Claude Context

> Linux 서버 (Spring Boot). 루트 `CLAUDE.md`를 먼저 읽고 이 문서를 본다.

## 역할

IPMax의 **중심 코어**. 아래 5개의 독립 서버로 구성된다.

| 서버 | 설명 |
|------|------|
| **웹 서버** | 소비자·제공자 대상 홈페이지 (HTML + Vanilla JS) |
| **제공자 백오피스** | 라우터 제공자 전용 관리 콘솔 |
| **관리자 백오피스** | 운영원 전용 내부 관리 콘솔 |
| **API 서버** | 외부 클라이언트(앱·라우터) 대상 REST API |
| **릴레이 서버** | WireGuard 홀펀칭 전용 서버 |

## 스택

- **Java + Spring Boot**
- **PostgreSQL** (원장/유저)
- **Redis** (세션·카운터·캐시)
- **Kafka** (이벤트 버스)
- **ELK** (로그·분석)
- **보안**: Spring Security + JWT
- **문서**: Swagger (springdoc-openapi)
- **로깅**: SLF4J
- **유틸**: Lombok (필수)
- **프론트**: HTML + Vanilla JS (서버 렌더링 or 정적 파일)
- **빌드**: Gradle

## 코딩 규칙

### 구조
- 디렉토리는 **도메인 단위**로 구성한다 (예: `user/`, `billing/`, `session/`, `provider/` …)
- 각 도메인 패키지 안에 `Controller`, `Service`, `Repository`, `Entity` 등을 함께 둔다
- **Interface / Impl 패턴 금지** — 서비스 클래스를 직접 작성한다

### DTO
- DTO는 항상 **사용하는 클래스의 inner class**로 선언한다

```java
// 예시
@RestController
public class UserController {

    @PostMapping("/users")
    public Response<Void> create(@RequestBody Request request) { ... }

    @Getter
    public static class Request {
        private String email;
        private String password;
    }

    @Getter
    @Builder
    public static class Response<T> {
        private T data;
        private String message;
    }
}
```

### Lombok
- 모든 클래스에서 Lombok을 적극 활용한다
- `@Getter`, `@Builder`, `@RequiredArgsConstructor`, `@Slf4j` 등 사용
- `@Data`는 Entity에 사용하지 않는다 (의도치 않은 `equals`/`hashCode` 방지)

### JPA
- **단순 조회, insert, update, delete**에만 사용한다
- 복잡한 집계·통계 쿼리는 Native Query 또는 QueryDSL을 사용한다
- 서비스·컨트롤러에서 직접 쿼리 금지 — 반드시 Repository 레이어를 통한다

### 보안
- **Spring Security + JWT** 방식으로 인증·인가를 처리한다
- Access Token / Refresh Token 분리
- 관리자 백오피스 엔드포인트는 역할(Role) 기반 접근 제어 + IP 화이트리스트 적용
- `application-*.yml`에 비밀값 하드코딩 금지

### 로깅
- 반드시 **SLF4J** 사용 (`@Slf4j` 어노테이션 활용)
- PII(개인정보) 마스킹 필수
- 트레이스 ID 포함

### Swagger
- 모든 API 서버에 **Swagger(springdoc-openapi) 필수 적용**
- 컨트롤러 및 DTO에 `@Operation`, `@Schema` 등 어노테이션으로 문서화
- 운영 환경에서는 Swagger UI 접근을 내부망으로 제한한다

## 주요 원칙

1. 정산·과금 로직은 반드시 단위 테스트 + 재현 가능한 픽스처 동반
2. 금액은 `BigDecimal` 또는 정수(최소 단위). `double` 금지
3. 모든 외부 입력에 유효성 검증 (Bean Validation)
4. 마이그레이션(Flyway) 순번 추가 시 반드시 롤백 시나리오 검토
5. Kafka consumer에서 예외 무시 금지 — 재처리 또는 DLQ로 보낼 것
6. 정산 레코드 직접 `UPDATE` 금지 — 반드시 