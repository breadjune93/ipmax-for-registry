# ADR 0001 — VPN 프로토콜로 WireGuard 채택

## 상태
Accepted

## 일자
프로젝트 착수 시

## 맥락

IPMax는 Consumer 앱이 Provider 라우터에 **빠르고 가볍게** 연결되어야 하며, 모바일 환경에서도 배터리 소모가 적어야 한다. 또한 OpenWrt 라우터에서 다중 피어를 쉽게 관리할 수 있어야 한다.

## 결정

**모든 Provider ↔ Consumer 터널의 기본 프로토콜로 WireGuard를 채택한다.**
OpenVPN·IPSec은 2차(레거시 호환 또는 DPI 우회 전용) 옵션으로만 고려한다.

## 근거

- **성능**: 커널 레벨 구현으로 처리량·레이턴시 우수
- **코드 복잡도**: OpenVPN 대비 매우 작은 코드베이스 → 보안 감사 용이
- **핸드셰이크 속도**: 모바일 네트워크 전환 시 재연결이 매우 빠름
- **다중 피어 관리 용이**: Provider 라우터에서 Consumer 피어를 동적 등록/해제하기 쉬움
- **최신 암호군**: ChaCha20, Curve25519, BLAKE2s 등 검증된 프리미티브

## 결과

- 긍정적
  - 저사양 라우터(OpenWrt) + 모바일에서 모두 부담 적음
  - 구현·디버깅 부담 감소
- 트레이드오프
  - **DPI에 시그니처가 상대적으로 탐지되기 쉬움** → 별도 난독화 레이어(ADR 추후) 필요
  - 기본 동작은 UDP 기반 → 일부 네트워크에서 차단 가능
- 후속 조치
  - Phase 3에서 DPI 우회용 난독화 프로토콜(예: obfuscation over TCP) 도입 ADR 별도 작성

## 대안 (Alternatives considered)

- **OpenVPN** — 성숙·범용이지만 무겁고 모바일 효율 낮음. DPI 우회는 쉽지만 핵심 성능에서 열세
- **IPSec/IKEv2** — iOS·macOS 기본 지원은 강점이지만 라우터 측 동적 피어 관리 복잡
- **자체 프로토콜** — 보안 리스크·운영 부담 과다

## 참고 자료

- WireGuard 공식 문서
- OpenWrt wireguard 패키지 문서
