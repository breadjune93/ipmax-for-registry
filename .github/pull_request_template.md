## 요약

<!-- 무엇을, 왜 바꿨는지 한두 문장 -->

## 영향 범위

- [ ] server
- [ ] router/openwrt
- [ ] apps/windows
- [ ] apps/android
- [ ] apps/macos
- [ ] apps/ios
- [ ] docs / shared (교차 영향)

## 체크리스트

- [ ] API 변경이라면 `docs/api-contracts/openapi.yaml` 업데이트
- [ ] 도메인/사업 결정 변경이라면 `docs/business/decisions/` 에 ADR 추가
- [ ] 정산/과금 로직 변경이라면 **단위 테스트** 동반
- [ ] 로그에 PII가 노출되지 않는지 확인
- [ ] 비밀값 커밋 없음 (`git diff` 재확인)
- [ ] `CLAUDE.md` (루트/해당 폴더) 갱신이 필요하면 반영

## 테스트

<!-- 어떻게 검증했는지 -->

## 참고

<!-- 이슈 링크, 관련 ADR 등 -->
