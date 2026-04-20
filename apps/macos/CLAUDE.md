# IPMax macOS App (SwiftUI) — Claude Context

> macOS Consumer 앱. 루트 `CLAUDE.md` 먼저.

## 역할

macOS 사용자에게 IPMax Consumer 경험 제공. 메뉴바 빠른 토글과 본 창의 상세 관리를 모두 지원.

## 스택

- **Swift** + **SwiftUI** (AppKit 보완 필요 시 `NSViewRepresentable`)
- 최소 지원: **macOS 13 Ventura** 이상 권장
- 아키텍처: **MV / MVVM** (`@Observable` 또는 `ObservableObject`)
- HTTP: `URLSession` + `async/await`
- JSON: `Codable`
- 로컬 보안 저장소: **Keychain**
- VPN 연결: **NetworkExtension Framework** (`NEPacketTunnelProvider`) + WireGuardKit
- 로깅: `os.Logger`

## 디렉토리 제안

```
apps/macos/
├── Ipmax.xcodeproj (또는 Package.swift)
├── Ipmax/
│   ├── IpmaxApp.swift
│   ├── Views/
│   ├── ViewModels/
│   ├── Services/           ← APIClient, Keychain
│   └── PacketTunnel/       ← NetworkExtension 타겟
├── IpmaxTests/
└── README.md
```

## 원칙

1. 네트워크/디스크 I/O는 **`async`** — 메인 액터 차단 금지
2. DTO는 OpenAPI 스펙으로부터 생성 (swift-openapi-generator 권장)
3. 민감값은 **Keychain** (accessibility: `.afterFirstUnlockThisDeviceOnly` 등 상황에 맞게)
4. **App Sandbox**·**Hardened Runtime** 유지, NetworkExtension Entitlement 정확히 설정
5. 사용자 동의 없이 **로그인 시 자동 실행** 금지 (SMAppService 사용 시 UI 동의 필수)
6. 크래시 리포트/텔레메트리는 opt-in

## 실행 (로컬)

```bash
# xcodebuild 기반 CI 예시
xcodebuild -scheme Ipmax -destination 'platform=macOS' build
xcodebuild -scheme Ipmax -destination 'platform=macOS' test
```

## 해서는 안 되는 일

- JWT·WG 키를 **UserDefaults**나 평문 파일에 저장
- Entitlement를 필요 이상으로 넓게 요청
- NetworkExtension 외의 수단으로 시스템 네트워크 경로 우회
- 서명·공증(Notarization) 없이 배포

## 자주 참조할 문서

- [API 계약](../../docs/api-contracts/openapi.yaml)
- [시스템 개요](../../docs/architecture/system-overview.md)
