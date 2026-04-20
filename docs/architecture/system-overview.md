# IPMax — 시스템 아키텍처 개요

## 구성 요소 한눈에

```
┌─────────────────────────────────────────────────────────────────┐
│                          Consumers                               │
│     Windows(WPF)   Android(Compose)   macOS(SwiftUI)  iOS(SwiftUI) │
└───────┬──────────────┬────────────────┬────────────────┬─────────┘
        │              │                │                │
        │ HTTPS API    │ HTTPS API      │ HTTPS API      │ HTTPS API
        │ WireGuard    │ WireGuard      │ WireGuard      │ WireGuard
        ▼              ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      IPMax Core (Linux)                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Spring Boot — REST API + Web Admin Console (HTML/JS)      │ │
│  └──────┬──────────────┬─────────────────────┬───────────────┘ │
│         │              │                     │                  │
│         ▼              ▼                     ▼                  │
│   ┌──────────┐    ┌──────────┐          ┌─────────┐            │
│   │PostgreSQL│    │  Redis   │  ◄────►  │  Kafka  │            │
│   │(원장/유저) │    │(세션/카운터)│          │(이벤트)  │            │
│   └──────────┘    └──────────┘          └────┬────┘            │
│                                              │                  │
│                                              ▼                  │
│                                       ┌────────────┐            │
│                                       │    ELK     │            │
│                                       │(로그/분석)   │            │
│                                       └────────────┘            │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                 Heartbeat/Control (HTTPS)
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Providers (OpenWrt + WireGuard)                  │
│   - Endpoint 등록 / 상태 보고                                      │
│   - WireGuard 피어 동적 관리                                       │
│   - 트래픽 집계 → N초 주기 Heartbeat                                │
└─────────────────────────────────────────────────────────────────┘
```

## 주요 플로우

### A. Consumer 연결 흐름

1. Consumer 앱 로그인 → 서버 JWT 발급
2. 가용 Endpoint 목록 조회 (지역·단가·품질 스코어 포함)
3. 선택한 Endpoint에 **세션 생성 요청** → 서버가 Provider에 **피어 등록 명령**(제어 채널)
4. 서버 응답으로 WireGuard 구성 파일 수신 → 터널 수립
5. 사용 중 매 N초 Provider Heartbeat로 트래픽 리포트 → Redis 잔액 차감
6. 잔액 소진 또는 사용자 종료 시 세션 종료 → 확정 트랜잭션 기록

### B. Provider 온보딩 흐름

1. 라우터에 OpenWrt + IPMax 에이전트 패키지 설치
2. 사전 발급된 **프로비저닝 토큰**으로 서버 등록
3. 서버가 초기 설정(UCI) 배포, Endpoint 메타 등록
4. 주기적 Heartbeat 시작 → 가용성·트래픽 리포트

### C. 크레딧 차감 흐름

```
Provider 라우터
   └─ trafficReported(endpointId, sessionId, bytes, secondsElapsed)
        → Kafka topic: "usage.events"
             → Consumer(stream): UsageAggregator
                  ├─ Redis: session:{id}.bytes += Δ
                  ├─ Redis: wallet:{user}.credit -= rate × Δ
                  └─ 세션 종료 신호 시 PostgreSQL에 확정 기록
```

### D. 정산(Settlement) 흐름

- 일 1회 배치 집계:
  - PostgreSQL의 확정 세션 기록 → Provider별 수익 계산
  - 정책(수수료·세금)에 따라 지급 가능 금액 산출
  - 정산 작업 큐 → 지급 PG 연동(Phase 4)

## 비기능 요구사항

| 영역 | 목표 |
|---|---|
| 가용성 | 세션 수립 99.9%, API 99.95% |
| 레이턴시 | Endpoint 선택 → 터널 수립 < 3s (p95) |
| 정산 정확도 | 세션 종료 후 10초 이내 원장 반영 |
| 데이터 보존 | 원장 7년, 로그 90일 |
| 보안 | WG 키 HSM/Keystore 보관, 관리자 MFA 필수 |

## 참고 문서

- 데이터 모델: [data-model.md](./data-model.md)
- API 계약: [../api-contracts/openapi.yaml](../api-contracts/openapi.yaml)
- 주요 결정: [../business/decisions/](../business/decisions/)
