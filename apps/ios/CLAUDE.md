# IPMax iOS App (SwiftUI) — Claude Context

> iOS Consumer 앱. 루트 `CLAUDE.md` 먼저.

## 역할

iPhone/iPad 사용자에게 IPMax Consumer 경험 제공. 배터리·백그라운드 정책에 특히 민감.

## 스택

- **Swift** + **SwiftUI**
- 최소 지원: **iOS 16** 이상 권장
- 아키텍처: MV/MVVM (`@Observable` 권장)
- HTTP: `URLSession` + `async/await`
- JSON: `Codable`
- 로컬 보안 저장소: **Keychain**
- VPN 연결: **NetworkExtension (`NEPacketTunnelProvider`)** + WireGuardKit
- 로깅: `os.Logger` (프로덕션에서는 PII 마스킹)

## 디렉토리 제안

```
apps/ios/
├── Ipmax.xcodeproj (또는 Package.swift)
├── Ipmax/
│   ├── IpmaxApp.swift
│   ├── Views/
│   ├── ViewModels/
│   ├── Services/
│   └── PacketTunnel/       ← NetworkExtension 타겟
├── IpmaxTests/
└── README.md
```

## 원칙

1. **App Tracking Transparency**·**Privacy Manifest** 명확히 작성
2. VPN 관련 Entitlement: `com.apple.developer.networking.networkextension` (Packet Tunnel) 정확히
3. **백그라운드 실행 정책 준수** — 단순 "계속 연결"을 위해 편법 API 금지
4. 민감값은 Keychain, accessibility 레벨 신중 선택
5. DTO는 OpenAPI 스펙 기반 생성
6. 테스트는 `XCTest` + UI 테스트 셋 최소한 유지

## 실행 (로컬)

```bash
xcodebuild -scheme Ipmax \
  -destination 'platform=iOS Simulator,name=iPhone 15' build
xcodebuild -scheme Ipmax \
  -destination 'platform=iOS Simulator,name=iPhone 15' test
```

## 해서는 안 되는 일

- 사용자 모르게 **Background Modes**를 남용
- App Store 리뷰에서 문제되는 **과도한 Entitlement**
- JWT·WG 키를 UserDefaults·평문 저장
- 정책상 모호한 DPI 우회 기능을 iOS에서 **기본 활성화**로 두기 (기능 플래그·리뷰 대응 별도)

## 자주 참조할 문서

- [API 계약](../../docs/api-contracts/openapi.yaml)
- [시스템 개요](../../docs/architecture/system-overview.md)
