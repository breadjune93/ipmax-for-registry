# IPMax Provider Router (OpenWrt) — Claude Context

> Provider가 운영하는 **OpenWrt 기반 라우터**의 펌웨어 설정·에이전트·스크립트 저장소. 루트 `CLAUDE.md` 먼저.

## 역할

- **WireGuard 서버**로서 Consumer를 수용
- 서버의 **원격 제어 명령**을 받아 피어 동적 추가/삭제
- **N초 주기 Heartbeat**로 트래픽/상태 리포트
- 비정상 상황(오프라인/과부하)에서 자동 복구

## 스택

- **OpenWrt** (지원 기종 화이트리스트는 별도 문서)
- **WireGuard** (`wireguard-tools`)
- **경량 에이전트** (shell + ubus / lua / 필요 시 Go 정적 바이너리)
- 설정 관리: **UCI**
- 통신: **HTTPS** (서버 측 `/provider/heartbeat`, 제어 채널)

## 디렉토리 제안

```
router/openwrt/
├── agent/
│   ├── ipmax-agent.sh         ← Heartbeat / 피어 관리 스크립트
│   ├── ipmax-agent.init       ← /etc/init.d 서비스 파일
│   └── config/
│       └── ipmax              ← UCI 설정 템플릿
├── packages/                   ← 자체 opkg 패키지 스펙
├── firmware/
│   └── README.md               ← 빌드/배포 절차
└── docs/
    ├── supported-devices.md
    └── provisioning.md         ← 초기 설치·토큰 등록 절차
```

## 원칙

1. **보안 우선** — Provider 라우터는 다수 Consumer를 수용하는 공격 표면. 기본 방화벽 Deny-by-default
2. **최소 권한** — 에이전트는 필요한 UCI 네임스페이스만 수정
3. **원격 복구 가능성** — 잘못된 설정 배포 시 **자동 롤백** 메커니즘 필수
4. 민감키(프로비저닝 토큰, WG private key)는 `/etc/ipmax/secrets/` 0600 + 소유자 root
5. 로그는 서버로 전송 전 **민감정보 마스킹** (고객 IP 등)
6. Heartbeat 실패 N회 누적 시 **안전 모드**(신규 피어 거절)로 전환

## Heartbeat 페이로드 (요약)

```
POST /provider/heartbeat
X-Provider-Token: <token>
{
  "endpointId": "...",
  "windowStart": "2025-06-17T12:00:00Z",
  "windowEnd":   "2025-06-17T12:00:10Z",
  "sessions": [
    { "sessionId": "...", "bytesIn": 12345, "bytesOut": 67890 }
  ]
}
```

상세: [../../docs/api-contracts/openapi.yaml](../../docs/api-contracts/openapi.yaml)

## 공통 커맨드 (현장 디버깅)

```sh
# WireGuard 상태
wg show

# UCI 설정 덤프
uci show ipmax

# 에이전트 로그
logread -e ipmax

# 수동 Heartbeat 테스트
/etc/ipmax/agent/ipmax-agent.sh heartbeat --once
```

## 해서는 안 되는 일

- 라우터에 **고객 사용 로그 원본** 장기 저장 (프라이버시·법적 위험)
- `root` 계정 SSH 비밀번호 로그인 허용 — 키 기반만
- 서버의 지시 없이 **피어 자동 생성** — 모든 피어는 서버 명령으로만 등록

## 관련 ADR

- [0001 WireGuard 선택](../../docs/business/decisions/0001-wireguard-선택.md)
- [0003 OpenWrt Provider 라우터](../../docs/business/decisions/0003-openwrt-provider-라우터.md)
