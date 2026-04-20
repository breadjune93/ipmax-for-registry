# ipmax-for-registry

**IPMax** — 개인·기업의 유휴 네트워크 자원을 상품화하는 충전형 VPN 마켓플레이스.

이 저장소는 IPMax의 **모든 플랫폼(서버·라우터·Windows·Android·macOS·iOS)** 코드와 **사업 문서**를 하나로 통합 관리하는 모노레포입니다.

---

## 왜 모노레포인가

사업은 계속 고도화됩니다. 서버 API가 바뀌면 4개 클라이언트가 따라가야 하고, 도메인 용어·정산 규칙이 바뀌면 모든 팀·AI 작업자(Claude Code)가 동시에 알아야 합니다.
- **단일 진실 공급원(SSOT)**: 최상위 [`CLAUDE.md`](./CLAUDE.md) + [`docs/`](./docs/)
- **동일한 도메인 모델**: [`shared/`](./shared/)의 공통 정의를 전 플랫폼이 참조
- **AI 친화적**: Claude Code가 어느 OS에서든 동일한 맥락으로 작업

## 디렉토리 한눈에 보기

```
CLAUDE.md            ← 사업 전체 맥락 (여기부터 읽기)
docs/                ← 비전·로드맵·ADR·API 계약·아키텍처
server/              ← Spring Boot 백엔드 + 웹 콘솔
router/openwrt/      ← Provider 라우터 펌웨어/설정
apps/windows/        ← WPF
apps/android/        ← Kotlin + Jetpack Compose
apps/macos/          ← SwiftUI
apps/ios/            ← SwiftUI
shared/              ← 공통 모델·상수
```

## OS별 작업 시작하기

### 공통 (어느 OS든)
```bash
git clone git@github.com:<your-org>/ipmax-for-registry.git
cd ipmax-for-registry
git pull                    # 항상 최신부터
```

그다음 작업할 폴더로 이동해서 Claude Code 실행:
```bash
cd server          # 또는 apps/windows, apps/android, ...
claude              # Claude Code 시작
```

Claude Code는 루트 `CLAUDE.md` + 해당 폴더의 `CLAUDE.md`를 자동으로 읽습니다.

### 각 플랫폼 상세
- **서버**: [`server/CLAUDE.md`](./server/CLAUDE.md)
- **OpenWrt**: [`router/openwrt/CLAUDE.md`](./router/openwrt/CLAUDE.md)
- **Windows (WPF)**: [`apps/windows/CLAUDE.md`](./apps/windows/CLAUDE.md)
- **Android**: [`apps/android/CLAUDE.md`](./apps/android/CLAUDE.md)
- **macOS**: [`apps/macos/CLAUDE.md`](./apps/macos/CLAUDE.md)
- **iOS**: [`apps/ios/CLAUDE.md`](./apps/ios/CLAUDE.md)

## 변경 제안(ADR) 쓰는 법

아키텍처·사업 결정을 바꿀 땐 [`docs/business/decisions/`](./docs/business/decisions/)에 번호 매긴 마크다운 추가:

```
docs/business/decisions/0004-정산-주기-일-단위-변경.md
```

[기존 ADR 템플릿](./docs/business/decisions/0001-wireguard-선택.md) 참고.

## 로드맵

[`docs/business/roadmap.md`](./docs/business/roadmap.md)

## 라이선스 / 보안

- 내부 프로젝트. 외부 공개 금지.
- 취약점 발견 시 비공개 채널로 보고 (공개 이슈 금지).
