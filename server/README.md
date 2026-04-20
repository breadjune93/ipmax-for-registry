# server

IPMax 백엔드 (Spring Boot).
작업 시작 전 [`CLAUDE.md`](./CLAUDE.md)를 반드시 읽을 것.

## 로컬 부팅 (예시)

```bash
docker compose up -d postgres redis kafka elasticsearch
./gradlew bootRun --args='--spring.profiles.active=local'
```

> Spring Boot 프로젝트 초기화는 아직 수행되지 않았습니다.
> Claude Code에게 "server 폴더에 Spring Boot 프로젝트를 초기화해줘. CLAUDE.md와 docs/architecture를 참고해서"라고 지시하세요.
