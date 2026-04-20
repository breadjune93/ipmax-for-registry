# IPMax Android App — Claude Context

> Android Consumer 앱 (Kotlin + Jetpack Compose). 루트 `CLAUDE.md` 먼저.

## 역할

Android 사용자에게 IPMax Consumer 경험 제공. 모바일에서의 배터리·데이터 효율이 특히 중요.

## 스택

- **Kotlin** + **Jetpack Compose**
- 아키텍처: **MVVM** + **UseCase** 레이어
- DI: **Hilt**
- HTTP: **Ktor Client** 또는 **Retrofit + OkHttp** (택1, ADR로 확정)
- JSON: **kotlinx.serialization**
- 로컬 보안 저장소: **EncryptedSharedPreferences** / **Android Keystore**
- VPN 연결: **VpnService** + **WireGuard-Android** 라이브러리
- 로깅: Timber (릴리스 빌드에서 PII 마스킹)

## 디렉토리 제안

```
apps/android/
├── settings.gradle.kts
├── build.gradle.kts
├── app/
│   ├── src/main/java/com/ipmax/android/
│   │   ├── IpmaxApplication.kt
│   │   ├── ui/                 ← Compose 화면
│   │   ├── data/               ← API·로컬 저장소
│   │   ├── domain/             ← UseCase·엔터티
│   │   └── vpn/                ← VpnService 래퍼
│   └── src/test, androidTest
└── README.md
```

## 원칙

1. **메인 스레드 I/O 금지** — `suspend` + `withContext(Dispatchers.IO)`
2. API 모델은 OpenAPI 스펙 기반으로 자동 생성 (`openapi-generator-gradle-plugin` 권장)
3. **최소 권한 원칙**: 필요한 순간에만 권한 요청
4. VPN 연결 중 **Foreground Service + 알림**으로 투명하게 안내
5. 백그라운드 제약(Doze·App Standby) 대응: 배터리 최적화 예외 요청은 **사용자 동의 후**
6. ProGuard/R8 규칙으로 민감 클래스 노출 최소화

## 실행 (로컬)

```bash
./gradlew :app:assembleDebug
./gradlew :app:installDebug
./gradlew test
./gradlew connectedAndroidTest
```

## 해서는 안 되는 일

- 비밀값을 `SharedPreferences` **평문**에 저장
- `INTERNET`·`FOREGROUND_SERVICE_SPECIAL_USE` 외 **불필요한 권한** 추가
- VPN 연결을 **명시 동의 없이** 유지
- 릴리스 빌드에서 디버그 로그·백도어 플래그 유지

## 자주 참조할 문서

- [API 계약](../../docs/api-contracts/openapi.yaml)
- [WireGuard ADR](../../docs/business/decisions/0001-wireguard-선택.md)
