# IPMax — Project Context for Claude Code

> 이 문서는 **모든 OS(Linux 서버, Windows, Mac, Android, iOS)에서 실행되는 Claude Code가 공통으로 읽는 최상위 컨텍스트**입니다.
> 사업의 본질, 용어, 원칙, 디렉토리 맵을 담습니다. 플랫폼별 세부사항은 각 폴더의 `CLAUDE.md`를 참고하세요.

---

## 1. 사업 한 줄 요약

**IPMax는 개인/기업의 유휴 네트워크 자원을 상품화하여 IP를 거래하는 충전형 VPN 마켓플레이스다.**

- **제공자(Provider)**: 집/사무실의 회선을 등록해 트래픽을 판매 (개인 또는 기업)
- **사용자(Consumer)**: 필요한 지역/품질의 IP를 충전된 크레딧으로 초 단위 과금으로 사용 (개인 또는 기업)
- **비즈니스 모델**: B2B2C — IPMax는 마켓플레이스 운영자이자 정산/품질관리 주체

## 2. 핵심 기능

1. **WireGuard 기반 고속 VPN 접속** — 기존 OpenVPN 대비 낮은 레이턴시, 짧은 핸드셰이크
2. **초 단위 트래픽 정산** — 실시간 사용량 계측 → 크레딧 차감/적립
3. **네트워크 프록시** — HTTP(S)/SOCKS5 프록시 모드
4. **DPI 우회(Deep Packet Inspection Bypass)** — 차단 환경에서의 안정 접속
5. **충전형 크레딧** — 선불 크레딧으로 사용자 과금, 제공자 수익 정산

## 3. 전체 시스템 아키텍처

```
[Consumer 앱]                    [Provider 라우터]
Windows / Mac / iOS / Android    OpenWrt + WireGuard
        │                                │
        │  VPN 터널 / 프록시              │  트래픽 실시간 리포팅
        ▼                                ▼
       ┌──────────────────────────────────────┐
       │        IPMax Core (Linux 서버)        │
       │  Spring Boot REST API + Web Console  │
       │  Kafka (이벤트) + Redis (세션/카운터) │
       │  PostgreSQL (원장) + ELK (관측)       │
       └──────────────────────────────────────┘
```

## 4. 기술 스택 (확정)

| 영역 | 스택 |
|---|---|
| **Linux 서버(백엔드/어드민 콘솔)** | Spring Boot, HTML, Vanilla JS |
| **데이터베이스** | PostgreSQL (트랜잭션), ELK (로그/분석) |
| **메시지/상태** | Kafka (이벤트 버스), Redis (세션·카운터·캐시) |
| **Provider 라우터** | OpenWrt + WireGuard |
| **Windows 앱** | WPF (.NET) |
| **Android 앱** | Kotlin + Jetpack Compose |
| **macOS 앱** | SwiftUI |
| **iOS 앱** | SwiftUI |

## 5. 도메인 용어집 (Glossary)

| 용어 | 정의 |
|---|---|
| **Provider** | 네트워크 회선을 판매하는 주체 (개인/기업). OpenWrt 라우터 운영 |
| **Consumer** | IP를 구매해 사용하는 주체 (개인/기업) |
| **Endpoint** | Provider가 제공하는 개별 접속 노드 (IP + 포트 + 지역 메타) |
| **Credit** | 충전형 선불 화폐. 트래픽 소비 시 초 단위 차감 |
| **Session** | Consumer가 특정 Endpoint에 연결된 1회 접속 단위 |
| **Tariff** | 지역·품질·대역폭에 따라 결정되는 크레딧 단가 |
| **Settlement** | Provider에 대한 수익 정산. 일 단위 집계 후 지급 |
| **DPI Bypass** | 특정 국가·망에서 VPN 시그니처 감지를 회피하는 난독화 기능 |
| **Heartbeat** | 라우터 → 서버로 보내는 상태/사용량 리포트 (N초 주기) |

## 6. 디렉토리 맵 (어디에 무엇이 있는지)

Claude는 작업 전에 이 맵을 참고해 필요한 문서/코드만 읽습니다.

```
ipmax-for-registry/
├── CLAUDE.md                       ← 지금 이 파일 (전체 사업 컨텍스트)
├── README.md                       ← 온보딩, 저장소 사용법
│
├── docs/
│   ├── business/
│   │   ├── vision.md               ← 사업 비전, 시장, 경쟁
│   │   ├── roadmap.md              ← 분기별 로드맵
│   │   └── decisions/              ← ADR (아키텍처 결정 기록)
│   │       ├── 0001-wireguard-선택.md
│   │       ├── 0002-초단위-정산-구조.md
│   │       └── 0003-openwrt-provider-라우터.md
│   ├── architecture/
│   │   ├── system-overview.md      ← 전체 시스템 다이어그램·플로우
│   │   └── data-model.md           ← 주요 엔터티 관계
│   └── api-contracts/
│       └── openapi.yaml            ← 서버 ↔ 클라이언트 공통 API 스펙
│
├── server/                         ← Spring Boot 백엔드
│   └── CLAUDE.md
│
├── router/
│   └── openwrt/                    ← Provider 라우터 펌웨어/설정
│       └── CLAUDE.md
│
├── apps/
│   ├── windows/                    ← WPF
│   │   └── CLAUDE.md
│   ├── android/                    ← Kotlin + Compose
│   │   └── CLAUDE.md
│   ├── macos/                      ← SwiftUI
│   │   └── CLAUDE.md
│   └── ios/                        ← SwiftUI
│       └── CLAUDE.md
│
└── shared/
    ├── models/                     ← 언어 중립 데이터 모델 정의
    └── constants/                  ← 에러코드, 단위, 공통 상수
```

## 7. 어느 OS에서 작업할 때의 규칙

### 7.1 작업 시작 전
1. 항상 `git pull` 먼저
2. 루트 `CLAUDE.md` (이 문서) + 작업할 폴더의 `CLAUDE.md` 읽기
3. 관련 ADR(`docs/business/decisions/`) 훑어보기

### 7.2 작업 완료 후
1. 의미 있는 결정이 있었다면 **ADR 추가** (`docs/business/decisions/NNNN-제목.md`)
2. API 스펙이 바뀌었다면 `docs/api-contracts/openapi.yaml` 업데이트
3. 커밋 메시지 컨벤션: `<scope>: <요약>` (예: `server: 크레딧 차감 배치 잡 추가`)
4. `git push`

### 7.3 플랫폼 간 충돌 방지
- **API 변경은 반드시 서버에서 먼저** → OpenAPI 스펙 업데이트 → 클라이언트는 그것을 읽고 따라감
- 도메인 용어가 바뀌면 이 `CLAUDE.md`의 Glossary부터 업데이트
- 에러 코드는 `shared/constants/`에 단일 정의

## 8. 공통 코딩 원칙

- **정산 관련 코드는 반드시 테스트 동반** — 1원·1초도 틀려서는 안 됨
- 금액/크레딧은 정수(최소 단위) 또는 `BigDecimal`로만 다룬다. `float/double` 금지
- 시간은 UTC 저장, UI만 로컬 변환
- 로그는 구조화(JSON)로 남기고 PII(IP 원본, 사용자 식별자)는 마스킹
- 비밀값(.env, keystore 등)은 **절대 커밋 금지** — `.gitignore` 철저히

## 9. 민감정보 취급

- API 키, DB 비밀번호, Provider 라우터 인증키는 **각 OS의 비밀관리**로 분리
  - Windows: DPAPI / Credential Manager
  - macOS/iOS: Keychain
  - Android: EncryptedSharedPreferences / Keystore
  - Linux 서버: 환경변수 + Vault(추후 도입)
- 커밋 전 `git diff`로 비밀값 섞였는지 확인

## 10. Claude에게 (행동 지침)

- 이 문서를 **"진실 공급원"** 으로 간주한다. 여기와 충돌하는 기억이 있다면 이 문서를 우선한다.
- 작업하려는 폴더에 `CLAUDE.md`가 있으면 먼저 읽는다.
- 사업 결정이나 도메인 용어에 해당하는 변경은 **반드시 본 문서 또는 ADR에 반영**한다.
- 모르면 추측하지 말고 질문한다. 특히 정산/과금 로직은 더더욱.
