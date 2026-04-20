# Session 상태 전이

> 세션 라이프사이클은 모든 플랫폼에서 **동일한 상태 머신**을 따른다.
> 구현 언어가 달라도 상태명·전이 규칙은 이 문서가 진실.

## 상태

| 상태 | 의미 |
|---|---|
| `PENDING` | 서버가 생성 요청을 받아 Provider에 피어 등록 명령을 보낸 직후 |
| `ACTIVE` | Provider가 피어 등록을 확인하고, 클라이언트가 터널을 성공적으로 수립 |
| `TERMINATING` | 종료 요청(사용자·크레딧 소진·관리자) 수신 → 정리 진행 중 |
| `CLOSED` | 정리 완료. 확정 트랜잭션이 PostgreSQL 원장에 기록됨 |
| `FAILED` | 수립 실패 또는 비정상 종료 (원인 코드 병기) |

## 전이 다이어그램

```
                 ┌────────────┐
   create ─────► │  PENDING   │
                 └─────┬──────┘
                       │ setupOk
                       ▼
                 ┌────────────┐     terminate       ┌─────────────┐
                 │  ACTIVE    │ ──────────────────► │ TERMINATING │
                 └─────┬──────┘                     └─────┬───────┘
                       │ setupFail / tunnelLost            │ cleanupOk
                       ▼                                   ▼
                 ┌────────────┐                     ┌────────────┐
                 │  FAILED    │                     │  CLOSED    │
                 └────────────┘                     └────────────┘
```

## 전이 규칙

- `PENDING → ACTIVE`: Provider의 피어 등록 ack + 클라이언트의 터널 수립 ping 둘 다 수신 시
- `PENDING → FAILED`: 일정 시간(예: 15초) 내 수립 실패
- `ACTIVE → TERMINATING`: 사용자 종료 / 크레딧 소진 / 관리자 강제 종료
- `ACTIVE → FAILED`: 연속 Heartbeat 누락 + 복구 실패
- `TERMINATING → CLOSED`: 라우터에서 피어 제거 확인 + 확정 트랜잭션 기록 성공
- `TERMINATING → FAILED`: 정리 실패 (수동 개입 필요, 알림 발송)

## 금지된 전이

- 한 번 `CLOSED` 또는 `FAILED`에 도달한 세션은 **재개 불가**. 새 세션을 생성해야 한다.
- `ACTIVE` 상태에서 `CLOSED`로 바로 가는 경로는 없다 — 반드시 `TERMINATING` 경유.

## 클라이언트 반영 가이드

- UI는 `PENDING` 동안 "연결 중…" 표시, 취소 버튼 제공
- `ACTIVE` 진입 시에만 트래픽 카운터 시작
- `TERMINATING` 중 사용자 조작을 차단(중복 요청 방지)
- `FAILED` 시 원인 코드에 따라 차등 안내
