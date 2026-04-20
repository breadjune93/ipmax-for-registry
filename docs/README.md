# docs

IPMax의 **문서 허브**. 코드보다 문서가 먼저 바뀌는 것이 원칙.

## 어디에 무엇이 있나

- [`business/vision.md`](./business/vision.md) — 사업의 한 줄 정의와 방향
- [`business/roadmap.md`](./business/roadmap.md) — 분기별 로드맵
- [`business/decisions/`](./business/decisions/) — **ADR** (아키텍처/사업 결정 기록)
- [`architecture/system-overview.md`](./architecture/system-overview.md) — 시스템 전체 그림
- [`architecture/data-model.md`](./architecture/data-model.md) — 핵심 엔터티/관계
- [`api-contracts/openapi.yaml`](./api-contracts/openapi.yaml) — 서버 ↔ 클라이언트 API 계약(진실 공급원)

## 변경 원칙

- **코드보다 문서 먼저**: 사업·도메인 결정이 바뀌면 이 폴더부터 반영
- **ADR은 누적**: 기존 ADR을 수정하지 말고 새 번호로 `Supersedes ADR-NNNN` 표기
- **API 변경은 OpenAPI 스펙부터**: 스펙을 먼저 고치고, 각 플랫폼이 그걸 읽어 따라감
