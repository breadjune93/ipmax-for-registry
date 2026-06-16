# router/openwrt — IPMAX Provider 라우터 펌웨어

> OpenWrt 24.10 / MediaTek MT7981B 기반 Provider 라우터 에이전트.
> 소스 코드: [`breadjune93/ipmax-for-router`](https://github.com/breadjune93/ipmax-for-router) — `relay/` 디렉토리

---

## 목차

1. [역할 및 개요](#1-역할-및-개요)
2. [소스 디렉토리 구조](#2-소스-디렉토리-구조)
3. [핵심 컴포넌트](#3-핵심-컴포넌트)
4. [설정 파일](#4-설정-파일)
5. [CLI 명령어 (ipmax-wg)](#5-cli-명령어-ipmax-wg)
6. [데몬 (ipmax-agent)](#6-데몬-ipmax-agent)
7. [WebSocket 릴레이 에이전트 (websocket/)](#7-websocket-릴레이-에이전트-websocket)
8. [클라이언트 접속 티어 (T1–T6)](#8-클라이언트-접속-티어-t1t6)
9. [빌드 및 배포](#9-빌드-및-배포)
10. [운영 명령어](#10-운영-명령어)

---

## 1. 역할 및 개요

IPMAX Provider 라우터는 단순 VPN 서버가 아닌 **Exit Node** 장비다.

| 역할 | 설명 |
|---|---|
| **WireGuard 서버** | Consumer(클라이언트)를 개별 WireGuard 인터페이스로 수용 |
| **Exit Node** | 모든 VPN 트래픽이 Provider WAN IP를 통해 외부로 나감 |
| **NAT/SNAT** | nftables로 직접 처리 — 중앙 서버를 통한 중계 없음 |
| **상태 보고** | 1분 주기 Kafka 전송으로 운영 데이터 보고 |
| **자동 복구** | WAN 끊김 / IP 변경 / WireGuard 다운 자동 감지 및 복구 |

---

## 2. 소스 디렉토리 구조

```
relay/                              ← ipmax-for-router/relay/ (구 ipmax-router)
├── cmd/
│   ├── ipmax-wg/
│   │   └── main.go                 ← WireGuard 관리 CLI
│   ├── ipmax-agent/
│   │   └── main.go                 ← 통합 데몬 (메인 프로세스)
│   └── ipmax-client/               ← Windows 클라이언트 빌더
│       ├── main.go
│       ├── tier.go                 ← 티어 선택 로직
│       ├── t1_direct.go            ← T1: 직접 WireGuard
│       ├── t2_punch.go             ← T2: NAT 홀 펀칭
│       ├── t3_relay.go             ← T3: UDP 릴레이
│       ├── t4_wgws.go              ← T4: WireGuard over WebSocket
│       └── t5_vless.go             ← T5/T6: VLESS over WebSocket/TLS
├── internal/
│   ├── config/config.go            ← 설정 구조체 및 파일 I/O
│   ├── agentcfg/config.go          ← 에이전트 전용 설정
│   ├── wg/wg.go                    ← WireGuard 인터페이스 관리
│   ├── nft/nft.go                  ← nftables NAT 규칙 관리
│   ├── nat/nat.go                  ← NAT 홀 펀칭 / 엔드포인트 감지
│   ├── kafka/producer.go           ← 비동기 Kafka 프로듀서 (버퍼링)
│   ├── collector/                  ← 상태 수집
│   │   ├── collector.go            ← 통합 스냅샷 생성
│   │   ├── system.go               ← CPU / 메모리
│   │   ├── wan.go                  ← WAN 상태 / Public IP
│   │   ├── vpn.go                  ← WireGuard 세션
│   │   └── process.go              ← 프로세스 상태
│   ├── session/tracker.go          ← 실시간 세션 추적
│   ├── watchdog/watchdog.go        ← WAN / WG / NAT 자동 복구
│   ├── signaling/signaling.go      ← 시그널링 서버 통신
│   └── api/                        ← Web UI + REST API
│       ├── server.go
│       ├── handler.go
│       ├── ui.go
│       └── ui.html                 ← 내장 Web UI
├── build/
│   └── build_clients.sh            ← 클라이언트 EXE 빌드 스크립트
└── scripts/
    └── ipmax-wg                    ← procd init 스크립트 (OpenWrt 서비스)
```

```
websocket/                          ← ipmax-for-router/websocket/ (구 ipmax-ws-agent)
└── main.go                         ← WebSocket 릴레이 에이전트 (T4~T6 지원)
```

---

## 3. 핵심 컴포넌트

### 3.1 ipmax-wg (CLI)

WireGuard 인터페이스와 NAT 규칙을 설정하는 관리 도구.
클라이언트(peer)마다 **독립된 WireGuard 인터페이스**와 **UDP 포트**를 할당한다.

- 각 peer: `wg0`, `wg1`, `wg2`... — 고유 인터페이스
- VPN 주소 대역: `10.100.<index>.0/30` — 서버 `.1`, 클라이언트 `.2`
- UDP 포트: BasePort(51820)부터 순차 할당

### 3.2 ipmax-agent (데몬)

라우터의 핵심 상주 프로세스. 아래를 통합 관리한다.

| 기능 | 주기 |
|---|---|
| WireGuard 인터페이스 활성화 | 시작 시 1회 |
| 세션 상태 추적 | 3초 |
| Kafka 상태 전송 | 60초 (설정 가능) |
| 시그널링 서버 등록 | 60초 |
| NAT 홀 펀칭 폴링 | 1초 |
| 워치독 점검 | 30초 |
| Web UI / REST API | 상시 (`0.0.0.0:8080`) |

### 3.3 Kafka 프로듀서 (`internal/kafka`)

- **비동기 비차단(non-blocking)** — Kafka 장애가 VPN 트래픽을 막지 않음
- **인메모리 버퍼** — 최대 120개 메시지 보관 (기본값, ~2시간분)
- Kafka 복구 시 버퍼 자동 플러시 (오래된 것부터)
- 버퍼가 꽉 차면 가장 오래된 메시지 폐기 (ring-buffer)

### 3.4 워치독 (`internal/watchdog`)

30초마다 아래를 점검하고 자동 복구한다.

| 감지 항목 | 복구 동작 |
|---|---|
| WAN 끊김 (`8.8.8.8` ping 실패) | 이벤트 발행, WireGuard 점검 재개 대기 |
| Public IP 변경 | 이벤트 발행 (재등록은 60초 사이클에서 처리) |
| nftables NAT 테이블 소실 | 자동 재생성 + 모든 peer 규칙 재추가 |
| WireGuard 인터페이스 다운 | `wg-quick up` + NAT 규칙 재추가 |

### 3.5 Web UI / REST API (`internal/api`)

- 주소: `http://<라우터IP>:8080`
- 브라우저에서 세션 상태, 트래픽, peer 목록 확인
- peer 추가/삭제, 트래픽 제한 설정 가능

---

## 4. 설정 파일

### 4.1 WireGuard 설정 (`/etc/ipmax/wg.json`)

```json
{
  "wan_interface": "eth1",
  "base_port": 51820,
  "api_listen": "0.0.0.0:8080",
  "max_peers": 50,
  "max_mbps": 100.0,
  "signaling_server": "http://211.48.64.195:5000",
  "peers": [
    {
      "id": "client01",
      "interface": "wg0",
      "listen_port": 51820,
      "server_keys": {
        "private_key": "...",
        "public_key": "..."
      },
      "client_pub_key": "...",
      "server_vpn_ip": "10.100.0.1",
      "client_vpn_ip": "10.100.0.2",
      "vpn_subnet": "10.100.0.0/30",
      "traffic_limit_pct": 0,
      "enabled": true,
      "created_at": 1718500000
    }
  ]
}
```

### 4.2 에이전트 설정 (`/etc/ipmax/agent.json`)

```json
{
  "device_id": "router-001",
  "wg_config_path": "/etc/ipmax/wg.json",
  "interval_sec": 60,
  "watch_procs": ["ipmax-agent", "ipmax-ws-relay"],
  "kafka": {
    "brokers": ["kafka.example.com:9092"],
    "topic": "ipmax.router.status",
    "max_buf": 120
  }
}
```

---

## 5. CLI 명령어 (ipmax-wg)

```sh
# 기본 설정 초기화
ipmax-wg init -wan eth1 -port 51820

# 모든 WireGuard 인터페이스 + NAT 활성화
ipmax-wg up

# 모든 인터페이스 + NAT 비활성화
ipmax-wg down

# 현재 상태 출력 (인터페이스, peer, NAT)
ipmax-wg status

# 새 클라이언트 추가 (키 자동 생성, 인터페이스/포트 자동 할당)
ipmax-wg peer add -id client01

# 클라이언트 목록
ipmax-wg peer list

# 클라이언트 제거
ipmax-wg peer remove client01

# 설정 파일 경로 지정
ipmax-wg -config /etc/ipmax/wg.json up
```

`peer add` 실행 시 출력 예시:
```
Peer added: client01
Interface:  wg0
Port (UDP): 51820
Server VPN: 10.100.0.1
Client VPN: 10.100.0.2

--- Client WireGuard config ---
[Interface]
PrivateKey = <client-private-key>
Address = 10.100.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = <server-public-key>
AllowedIPs = 0.0.0.0/0
Endpoint = <SERVER_PUBLIC_IP>:51820
PersistentKeepalive = 25
```

---

## 6. 데몬 (ipmax-agent)

```sh
# 에이전트 설정 초기화
ipmax-agent init

# 데몬 실행 (포그라운드)
ipmax-agent run

# 현재 상태 JSON 출력
ipmax-agent status

# Kafka 연결 테스트
ipmax-agent ping

# 설정 파일 지정
ipmax-agent -config /etc/ipmax/agent.json run
```

### OpenWrt procd 서비스 등록

```sh
# /etc/init.d/ipmax-agent 에 스크립트 복사 후:
/etc/init.d/ipmax-agent enable
/etc/init.d/ipmax-agent start
```

---

## 7. WebSocket 릴레이 에이전트 (websocket/)

클라이언트의 T4~T6 연결을 지원하기 위해 라우터에서 실행하는 별도 프로세스.

```sh
# 실행
ipmax-ws-relay -config /etc/ipmax/wg.json

# 동작 방식
# wg.json의 모든 peer에 대해 goroutine을 생성하고
# 각 peer가 릴레이 서버(T4: WebSocket, T5/T6: WebSocket+TLS)에 연결 유지
# 연결 끊김 시 지수 백오프(5→10→20→...→120초)로 자동 재연결
```

---

## 8. 클라이언트 접속 티어 (T1–T6)

클라이언트는 네트워크 환경에 따라 6가지 접속 방식을 순차적으로 시도한다.

| 티어 | 방식 | 조건 | 설명 |
|---|---|---|---|
| **T1** | WireGuard 직접 | 라우터 WAN 직접 접근 가능 | 최고 성능, 직접 UDP |
| **T2** | NAT 홀 펀칭 | CGNAT 환경 | 시그널링 서버 경유 홀 펀칭 후 직접 |
| **T3** | UDP 릴레이 | T2 실패 시 | VPS UDP 릴레이 경유 |
| **T4** | WireGuard over WS | T3 실패 / TCP 환경 | WebSocket 터널로 WireGuard UDP 포워딩 |
| **T5** | VLESS over WS | T4 실패 | VLESS 프로토콜 + WebSocket |
| **T6** | VLESS over WSS | T5 실패 | VLESS + WebSocket + TLS (443 포트) |

T4~T6은 `ipmax-ws-relay`가 릴레이 서버와의 WebSocket 연결을 유지해야 동작한다.

---

## 9. 빌드 및 배포

### 라우터 바이너리 크로스 컴파일 (Linux ARM64 / OpenWrt)

```sh
# ipmax-wg (WireGuard 관리 CLI)
GOOS=linux GOARCH=arm64 go build -o ipmax-wg ./cmd/ipmax-wg

# ipmax-agent (통합 데몬)
GOOS=linux GOARCH=arm64 go build -o ipmax-agent ./cmd/ipmax-agent

# WebSocket 릴레이 에이전트
cd websocket/
GOOS=linux GOARCH=arm64 go build -o ipmax-ws-relay-arm64 .
```

### 라우터에 배포

```sh
# 바이너리 전송
scp ipmax-wg ipmax-agent root@<ROUTER_IP>:/usr/bin/
scp ipmax-ws-relay-arm64 root@<ROUTER_IP>:/usr/bin/ipmax-ws-relay

# 설정 디렉토리 생성
ssh root@<ROUTER_IP> "mkdir -p /etc/ipmax && chmod 700 /etc/ipmax"

# 초기화
ssh root@<ROUTER_IP> "ipmax-wg init && ipmax-agent init"

# peer 추가 (라우터에서 직접)
ssh root@<ROUTER_IP> "ipmax-wg peer add -id client01"
```

---

## 10. 운영 명령어

```sh
# WireGuard 상태 확인
wg show

# 모든 인터페이스 상태
ipmax-wg status

# 에이전트 상태 스냅샷 (JSON)
ipmax-agent status

# Kafka 연결 확인
ipmax-agent ping

# WebSocket 릴레이 연결 확인 (TCP 8084 포트 연결 수)
ss -tnp | grep :8084 | grep ESTAB | wc -l

# 에이전트 로그 (systemd)
journalctl -u ipmax-agent -f

# nftables NAT 테이블 확인
nft list table inet ipmax_nat

# 실시간 트래픽 (인터페이스별)
vnstat -l -i wg0

# Web UI
# 브라우저에서: http://<ROUTER_IP>:8080
```

---

## 관련 링크

- 소스 코드: [breadjune93/ipmax-for-router](https://github.com/breadjune93/ipmax-for-router)
  - `relay/` — 라우터 펌웨어 (이 문서의 주제)
  - `websocket/` — WebSocket 릴레이 에이전트
- API 스펙: [`../../docs/api-contracts/openapi.yaml`](../../docs/api-contracts/openapi.yaml)
- ADR: [`0003-openwrt-provider-라우터.md`](../../docs/business/decisions/0003-openwrt-provider-라우터.md)
