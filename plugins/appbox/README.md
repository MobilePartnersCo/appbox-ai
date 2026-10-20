# AppBox

앱박스(AppBox) 계정에 AI 앱을 연결해, 내 앱의 푸시 준비 상태(앱 등록 · FCM · 빌드 · 기기)를 확인하고 다음에 할 일을 단계별로 안내받는 플러그인입니다. 모든 도구는 **조회 전용**이며, 설정 변경과 푸시 발송은 앱박스 콘솔에서 직접 합니다.

## 들어 있는 것

- **원격 MCP 서버 연결** — `https://mcp.appboxapp.com/mcp` (Streamable HTTP). 이 플러그인은 주소만 설정하며 토큰이나 비밀값을 담지 않습니다.
- **스킬 3종**
  - `onboarding` — 지금 상태를 진단하고 다음 한 단계를 안내합니다.
  - `setup-fcm` — Firebase(FCM) 키 발급과 콘솔 등록 단계를 안내합니다. 키 파일 자체는 다루지 않습니다.
  - `check-setup` — 푸시가 오지 않을 때 연동 상태를 순서대로 점검합니다.
- 실행하는 스크립트, 훅, 로컬 프로그램은 없습니다.

## 설치

Claude Code 에서 한 줄로 설치합니다(마켓플레이스 `MobilePartnersCo/appbox-ai` 를 함께 등록합니다).

```
/plugin install appbox --marketplace MobilePartnersCo/appbox-ai
```

터미널에서는 `claude plugin marketplace add MobilePartnersCo/appbox-ai` 다음 `claude plugin install appbox@appbox-ai` 입니다.

## 연결 방법

1. 플러그인을 설치한 뒤 Claude Code 에서 `/mcp` 를 열고 `plugin:appbox:appbox` → **Authenticate** 를 고릅니다.
2. 브라우저에 앱박스 콘솔이 열리면 로그인하고, 허용 화면에서 **허용할 앱**을 고른 뒤 외부 AI 전송 고지에 동의하고 **허용**합니다.
3. Claude Code 로 돌아와 「내 앱 목록 보여 줘」, 「푸시가 안 와요」처럼 요청하면 됩니다.

앱박스 계정이 필요합니다. 계정은 [앱박스](https://www.appboxapp.com)에서 만들 수 있습니다. 화면별 안내는 [앱박스 가이드 › AI 연결(MCP)](https://www.appboxapp.com/guide/setup/AI%20%EC%97%B0%EA%B2%B0%20%EC%86%8C%EA%B0%9C).

## 도구 (모두 조회 전용)

| 묶음 | 도구 |
|---|---|
| 내 앱 | 시작 단계 안내 · 내 앱 목록 · 앱 연동 점검 · 빌드 상태 · 기기 찾기 |
| 가이드·예제 | 앱박스 소개와 가이드 목차 · 가이드 검색 · 가이드 본문 · 예제 목록 · 예제 파일 본문 · 서버 API 명세 |

## 주고받는 데이터

- 이 플러그인이 접속하는 곳은 위 앱박스 MCP 서버 한 곳뿐입니다. 다른 곳으로 데이터를 보내지 않습니다.
- 로그인은 OAuth 로 하며, 토큰은 AI 앱이 받아 보관하고 자동으로 갱신합니다.
- 서버는 **연결할 때 허용한 앱**의 조회 결과(앱 정보 · 연동 설정 · 기기 수 · 빌드 상태, 요청한 경우 기기 단위 상태)만 AI 앱에 돌려줍니다. 계정 이메일, 이름, 푸시 토큰 값은 보내지 않습니다.
- 연결은 앱박스 콘솔 **계정 설정 › MCP 연결**에서 언제든 해제할 수 있습니다. AI 앱에서 커넥터만 삭제하면 앱박스 쪽 연결은 남으니 콘솔에서 해제하세요. 30일 동안 쓰지 않은 연결은 자동으로 해제됩니다.
- 자세한 내용: [개인정보 처리방침](https://www.appboxapp.com/policy/privacy) 제11조 「AI 앱 연결에 따른 개인정보 제공」 · [서비스 약관](https://www.appboxapp.com/policy/terms/use)

## English summary

AppBox connects your AI app to your AppBox account so you can check the push-notification readiness of your apps (app registration, FCM, builds, devices) and get step-by-step guidance. The plugin only configures the remote MCP server `https://mcp.appboxapp.com/mcp` and ships three skills (onboarding, setup-fcm, check-setup). It contains no scripts, hooks, or credentials. Sign-in uses OAuth with your AppBox account, every tool is read-only, and the server returns data only for the apps you allow when connecting. Privacy policy: https://www.appboxapp.com/policy/privacy

## 문의 · 라이선스

- 문의: appbox@appboxapp.com
- 라이선스: MIT
