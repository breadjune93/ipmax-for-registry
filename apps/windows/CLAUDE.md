# IPMax Windows App (WPF) — Claude Context

> Windows 데스크톱 Consumer 앱. 루트 `CLAUDE.md` 먼저.

## 역할

Windows 사용자에게 IPMax Consumer 경험 제공: 로그인, Endpoint 선택, 세션 연결, 크레딧 현황, 설정.

## 스택

- **WPF** (.NET 8 LTS 권장)
- 언어: **C#**
- 아키텍처: **MVVM** (`CommunityToolkit.Mvvm` 권장)
- DI: `Microsoft.Extensions.DependencyInjection`
- HTTP: `HttpClient` + Polly (재시도/타임아웃)
- WireGuard 연동: **WireGuard for Windows** 의 Tunnel Service 또는 자체 번들 (전략은 ADR로 확정 예정)
- 로컬 보안 저장소: **DPAPI** (사용자 단위) 또는 **Windows Credential Manager**
- 로깅: Serilog (JSON) — PII 마스킹

## 디렉토리 제안

```
apps/windows/
├── Ipmax.Windows.sln
├── src/
│   ├── Ipmax.Windows/           ← WPF 앱 본체
│   │   ├── App.xaml
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   ├── Services/            ← API 클라이언트, 보안 스토리지
│   │   └── Wireguard/
│   └── Ipmax.Windows.Tests/
└── README.md
```

## 원칙

1. **UI 스레드 블로킹 금지** — 네트워크·디스크 I/O는 항상 `async/await`
2. 모든 DTO는 **OpenAPI 스펙(`../../docs/api-contracts/openapi.yaml`)로부터 생성** (NSwag / Kiota 중 택1)
3. 민감값(토큰·WG 개인키)은 **DPAPI로 보호** — 평문으로 설정 파일 기록 금지
4. 크래시 리포트 수집 시 PII 마스킹 후 전송
5. 사용자 동의 없이 시스템 시작 시 자동 실행·네트워크 가로채기 금지

## 실행 (로컬)

```powershell
dotnet restore
dotnet build
dotnet run --project src/Ipmax.Windows
```

## 공통 커맨드

```powershell
dotnet test
dotnet format
```

## 주요 화면 흐름(초안)

1. 로그인 → 2. 지역/필터로 Endpoint 검색 → 3. 선택 후 연결 버튼 → 4. 진행률/상태 표시 → 5. 연결 중 크레딧/트래픽 실시간 표시 → 6. 종료

## 해서는 안 되는 일

- WG 개인키·JWT를 **사용자 디렉토리 평문**에 저장
- 에러 발생 시 예외 정보를 **사용자에게 raw**로 표시 (서버 스택트레이스 노출)
- 관리자 권한을 **항상 요구**하는 설치자(필요 시에만 UAC)
- 백그라운드에서 사용자 몰래 **연결 유지** (명시 동의 없이)

## 자주 참조할 문서

- [API 계약](../../docs/api-contracts/openapi.yaml)
- [시스템 개요](../../docs/architecture/system-overview.md)
