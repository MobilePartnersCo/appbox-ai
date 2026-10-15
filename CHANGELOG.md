# 변경 이력

버전은 SemVer 를 따르고, 서버(MCP `appbox`) 버전과 별개입니다. 고객에게는 `main` 브랜치의 버전이 올라갈 때 업데이트가 전달됩니다.

## 1.0.0 — 2026-10-15

공개 첫 판.

- 앱박스 원격 MCP 서버 연결 — 앱박스 계정 로그인(OAuth). 주소만 설정하고 토큰은 다루지 않습니다
- Claude Code · Codex 플러그인 형식(마켓플레이스 이름 `appbox-ai`, 플러그인 이름 `appbox`)
- 스킬 3종: `onboarding`(시작 안내) · `setup-fcm`(FCM 발급·등록 안내) · `check-setup`(푸시 연동 점검)
- MIT 라이선스
- 아이콘: 로고(`assets/logo.png`·`logo.svg`) · 작은 아이콘(`assets/icon-small.svg`) · 대표 색 `#1C1443`(Codex 목록 표시)
