# IPMax — 데이터 모델 개요

> 개념 수준의 엔터티·관계. 실제 스키마는 `server/` 하위의 마이그레이션 파일이 진실이다.
> 이 문서는 변경 시 함께 업데이트한다.

## 핵심 엔터티

### User
계정 주체. 개인 또는 기업 모두.

- `id`, `email`, `passwordHash`, `role` (`CONSUMER` | `PROVIDER` | `BOTH` | `ADMIN`)
- `kycStatus`, `createdAt`
- 관계: 다수의 `Wallet`, `Device`, `Endpoint`(Provider인 경우)

### Wallet
사용자 크레딧 지갑.

- `id`, `userId`, `balance` (BigDecimal, 최소 단위 정수)
- `currency` (내부 크레딧 단위)
- 낙관적 잠금용 `version`

### Endpoint
Provider가 제공하는 개별 접속 노드.

- `id`, `providerUserId`, `region`, `publicIp`, `wgPort`
- `tariffId` (단가 정책), `qualityScore`, `status` (`ACTIVE`|`DEGRADED`|`OFFLINE`)
- `features` (flags: `proxy_http`, `proxy_socks5`, `dpi_bypass` …)

### Session
Consumer 1회 접속.

- `id`, `consumerUserId`, `endpointId`
- `startedAt`, `endedAt`
- `bytesIn`, `bytesOut`, `durationSec`
- `creditCharged` (확정), `status`

### UsageEvent (Kafka / 로그 원천)
라우터에서 N초 주기로 올라오는 집계 이벤트.

- `sessionId`, `endpointId`, `windowStart`, `windowEnd`
- `bytesIn`, `bytesOut`
- 멱등 키: `(sessionId, windowStart)`

### Tariff
지역·품질·대역폭에 따른 단가.

- `id`, `region`, `tier`, `creditPerMB`, `creditPerSec`
- `validFrom`, `validTo`

### Settlement
Provider 수익 정산 기록.

- `id`, `providerUserId`, `period` (예: 2025-06-17)
- `grossCredit`, `feeCredit`, `netCredit`
- `status` (`PENDING`|`PAID`|`FAILED`), `externalPayoutRef`

### Device
사용자가 로그인한 클라이언트 디바이스.

- `id`, `userId`, `platform` (`windows`|`android`|`macos`|`ios`)
- `publicKey` (WireGuard), `lastSeenAt`

## 관계 요약

```
User 1 ─── N Wallet
User 1 ─── N Device
User(Provider) 1 ─── N Endpoint
Endpoint 1 ─── N Session
User(Consumer) 1 ─── N Session
Session 1 ─── N UsageEvent
Endpoint N ── 1 Tariff
User(Provider) 1 ─── N Settlement
```

## 금액/단위 원칙

- **크레딧은 정수(최소 단위)로만 저장** — 화면에만 소수점 변환
- **시간은 UTC**, 초 단위 Long
- **바이트는 Long** (MB·GB 변환은 표시 계층에서만)

## 민감정보 취급

- `User.email` — 로그에 남길 때 해시 또는 마스킹
- `Device.publicKey` — 저장 OK / 로그 금지
- `passwordHash` — Argon2id 권장 (서버 ADR에서 확정)

## 변경 절차

1. 서버에서 마이그레이션 추가
2. 이 문서 엔터티 섹션 업데이트
3. OpenAPI 스펙(`docs/api-contracts/openapi.yaml`) 반영
4. 각 클라이언트는 OpenAPI 기반으로 모델 동기화
