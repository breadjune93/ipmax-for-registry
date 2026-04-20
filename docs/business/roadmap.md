# IPMax — 로드맵

> 분기별 목표 중심. 세부 태스크는 이슈 트래커에서 관리.
> 변경 시 이 문서를 업데이트하고 커밋 메시지에 `docs: roadmap update` 명시.

## 현재 단계

> _여기를 현재 상황에 맞게 갱신하세요._

## Phase 1 — MVP (기반 구축)

**목표**: Provider 라우터 1종 + Consumer 앱 1종 + 서버 코어로 실거래 가능한 최소 구성

- [ ] 서버: 회원/인증, Endpoint 등록 API, 크레딧 지갑, 기본 정산 파이프라인
- [ ] OpenWrt: WireGuard 컨테이너·Heartbeat 에이전트·트래픽 리포트
- [ ] 클라이언트: Windows(WPF) 1종 — 로그인, Endpoint 선택, 세션 연결, 크레딧 확인
- [ ] 관측: ELK 로그 수집, 기본 대시보드
- [ ] 결제: 크레딧 충전 1개 PG 연동

## Phase 2 — 멀티 플랫폼 확장

- [ ] Android 앱 (Compose)
- [ ] iOS 앱 (SwiftUI)
- [ ] macOS 앱 (SwiftUI)
- [ ] 프록시(HTTP/SOCKS5) 모드 지원
- [ ] Provider 대시보드 (수익/트래픽 통계)

## Phase 3 — 품질·신뢰

- [ ] DPI 우회 모듈 (난독화 프로토콜)
- [ ] 지역/품질 스코어링 자동화
- [ ] 비정상 Provider 탐지 (트래픽 위조, 허위 지역)
- [ ] 세션 failover (다중 Endpoint 자동 전환)

## Phase 4 — 기업/SLA

- [ ] Enterprise 전용 API 토큰·할당제
- [ ] 전용 Endpoint 예약 모드
- [ ] 계약 기반 청구·인보이스
- [ ] 감사 로그·컴플라이언스

## Phase 5 — 생태계

- [ ] 서드파티 라우터 호환(공급사 SDK)
- [ ] 파트너 추천/제휴 프로그램
- [ ] 공개 API (Read-only 통계부터)

## 변경 로그

| 날짜 | 변경 | 작성자 |
|---|---|---|
| YYYY-MM-DD | 초안 작성 | — |
