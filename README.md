# 앱박스 AI 플러그인

앱박스(AppBox) 원격 MCP 서버를 Claude Code · Codex 에 연결하는 플러그인과, 그 플러그인을 담은 마켓플레이스 저장소입니다.
연결은 **앱박스 계정 로그인(OAuth)** 으로 하며, 토큰을 직접 다루지 않습니다.

## 구성

```
.claude-plugin/marketplace.json            Claude 마켓플레이스 매니페스트 (이름: appbox-ai)
.agents/plugins/marketplace.json           Codex 마켓플레이스 매니페스트 (이름: appbox-ai)
plugins/appbox/.claude-plugin/plugin.json  Claude 플러그인 매니페스트 (이름: appbox)
plugins/appbox/.codex-plugin/plugin.json   Codex 플러그인 매니페스트 (표시 이름·설명·카테고리 포함)
plugins/appbox/.mcp.json                    원격 MCP 서버 설정 (Streamable HTTP, 주소만 — 토큰 없음)
plugins/appbox/skills/onboarding/SKILL.md   스킬: 시작 안내 — 지금 상태 진단 → 다음 한 단계
plugins/appbox/skills/setup-fcm/SKILL.md    스킬: FCM 발급·등록 단계 안내 (키 파일은 다루지 않음)
plugins/appbox/skills/check-setup/SKILL.md  스킬: 푸시 연동 점검 절차
plugins/appbox/assets/                      아이콘(로고 · 작은 아이콘) — Codex 목록 표시용
```

- 서버: `https://console.appboxapp.com/api/mcp` (Streamable HTTP, POST 전용)
- 인증: **OAuth** — 처음 쓸 때 Claude Code 가 브라우저로 앱박스 로그인·허용 화면을 연다. 토큰은 Claude Code 가 받아 보관하고 자동으로 갱신한다(사용자는 토큰을 보지 않는다)
- 도구(모두 조회 전용): `appbox_get_onboarding_status` · `appbox_list_apps` · `appbox_check_app_setup` · `appbox_get_build_status`
- 기기 도구: `appbox_find_device`(조회) · `appbox_set_tester`(테스터 지정 — 연결할 때 「추가 권한(선택)」을 켠 경우에만)
- 앱 등록: `appbox_create_app` — 연결할 때 「추가 권한(선택)」의 「앱 등록」을 켠 경우에만. 만든 앱은 그 연결에 바로 허용됩니다
- 문서 도구(로그인 연결에 기본 포함, 서버에서 켜진 경우): `appbox_get_overview` · `appbox_search_docs` · `appbox_get_doc`(가이드) · `appbox_list_examples` · `appbox_get_example`(공개 샘플 소스) · `appbox_get_api_spec`(서버 API 명세)
- 이 저장소에는 **매니페스트와 스킬만** 둡니다. 서버 코드·자격 증명은 넣지 않습니다.

## 1. 설치

```bash
claude plugin marketplace add MobilePartnersCo/appbox-ai   # 마켓플레이스 등록 (이름: appbox-ai)
claude plugin install appbox@appbox-ai                   # 플러그인 설치
claude plugin list                                       # appbox@appbox-ai 가 enabled 인지 확인
```

- 업데이트를 자동으로 받으려면 Claude Code 의 `/plugin` → 마켓플레이스 `appbox-ai` 에서 자동 업데이트를 켭니다(서드파티 마켓플레이스는 기본 꺼짐).
- 이 저장소를 내려받은 위치에서 `claude plugin marketplace add ./` 로 로컬 등록할 수도 있습니다. 플러그인만 잠깐 써 보려면 `claude --plugin-dir ./plugins/appbox`.

제거: `claude plugin marketplace remove appbox-ai` (마켓플레이스와 그 플러그인을 함께 제거)

## 2. 연결 (앱박스 로그인)

1. Claude Code 세션에서 `/mcp` → `plugin:appbox:appbox` 서버를 고르고 **Authenticate**.
2. 브라우저에 앱박스 콘솔이 열리면 로그인합니다(이미 로그인돼 있으면 건너뜀).
3. 허용 화면에서 권한·**허용할 앱**(전체 또는 선택)을 확인하고, 외부 AI 전송 고지에 동의한 뒤 **허용**.
   - 「추가 권한(선택)」은 조회가 아닌 동작(테스터 기기 지정 등)입니다. 필요할 때만 켜세요 — 켜지 않아도 조회 도구는 모두 쓸 수 있습니다.
   - 화면에 "claude.ai 에서 확인된 앱"처럼 확인된 신원이 보이는지 확인하세요. "이름이 확인되지 않았습니다" 경고가 보이면 직접 시작한 연결이 아니라면 거부하세요.
4. 브라우저에 완료가 뜨면 Claude Code 로 돌아옵니다. `/mcp` 에서 connected 로 바뀝니다.

이후 스킬을 씁니다.

| 이럴 때 | 스킬 |
|---|---|
| "앱박스로 푸시 보내고 싶어요", "뭐부터 해야 하나요" | `/appbox:onboarding [앱 코드]` |
| "FCM 등록 어떻게 해요" | `/appbox:setup-fcm [앱 코드]` |
| "푸시가 안 와요" | `/appbox:check-setup [앱 코드]` |

도구의 전체 이름은 `mcp__plugin_appbox_appbox__appbox_list_apps` 형식입니다(권한 규칙에 쓸 때 이 이름 사용).

- 연결 확인·해제: 앱박스 콘솔 **계정 > MCP 연결**. 해제하면 즉시 끊기고, 다시 쓰려면 2번을 다시 합니다.
- AI 앱에서 앱박스 서버(커넥터)만 삭제하면 앱박스 쪽 연결은 남습니다. 끊으려면 콘솔에서 해제하세요.
- 허용할 앱: 연결할 때 체크한 앱만 볼 수 있습니다('전체 앱'은 그때 목록의 전체 선택). 연결한 뒤 만든 앱을 쓰려면 다시 연결해 그 앱을 체크합니다.
- 비밀번호를 바꾸거나 재설정하면 모든 MCP 연결이 해제됩니다(콘솔 알림으로 알려 줍니다).
- 30일 동안 쓰지 않은 연결은 자동으로 해제됩니다. 다시 쓰려면 다시 로그인합니다.

## 3. 다른 AI 앱에 수동 연결

모두 **주소만** 넣으면 처음 쓸 때 앱박스 로그인이 열립니다. 토큰을 넣지 마세요.

### Claude Code (플러그인 없이)

```bash
claude mcp add --transport http appbox https://console.appboxapp.com/api/mcp
```

`/mcp` 에서 appbox → Authenticate.

### Codex (플러그인)

이 저장소는 Codex 마켓플레이스 형식(`.agents/plugins/marketplace.json` · `plugins/appbox/.codex-plugin/plugin.json`)도 담고 있습니다. 스킬 3종이 함께 설치됩니다.

```bash
codex plugin marketplace add MobilePartnersCo/appbox-ai
codex plugin add appbox@appbox-ai
codex mcp login appbox                   # 브라우저에서 앱박스 로그인·허용
```

제거: `codex plugin remove appbox@appbox-ai` → `codex plugin marketplace remove appbox-ai`


### Codex (CLI, 플러그인 없이)

```bash
codex mcp add appbox --url https://console.appboxapp.com/api/mcp
```

추가하면 바로 앱박스 로그인이 시작됩니다(나중에 다시 로그인: `codex mcp login appbox`).

### Cursor

`~/.cursor/mcp.json`(전역) 또는 프로젝트의 `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "appbox": {
      "url": "https://console.appboxapp.com/api/mcp"
    }
  }
}
```

> Cursor 는 연결할 때 앱 이름이 "직접 등록한 앱 — 이름이 확인되지 않았습니다"로 보입니다(정상). 직접 시작한 연결일 때만 허용하세요.

### 로그인 없이 가이드만 (문서 입구)

앱박스 계정이 없어도 공개 가이드를 AI 앱에서 찾아볼 수 있습니다. 문서 도구 3종만 제공하고 로그인 화면이 뜨지 않습니다.

```bash
claude mcp add --transport http appbox-docs https://console.appboxapp.com/api/mcp/docs
```

```bash
codex mcp add appbox-docs --url https://console.appboxapp.com/api/mcp/docs
```

- 한 번에 메시지 하나만 받고(배치 불가), 호출 수 제한이 있습니다(IP 당 분당 60회).
- **Codex 는 이 주소를 추가할 때 앱박스 로그인 화면을 띄울 수 있습니다.** 문서 입구는 로그인이 필요 없으니 닫아도 됩니다(도구는 그대로 동작). 뜨지 않게 하려면 `codex mcp add` 대신 `~/.codex/config.toml` 에 직접 적습니다:

  ```toml
  [mcp_servers.appbox-docs]
  url = "https://console.appboxapp.com/api/mcp/docs"
  ```
- 앱박스 로그인 연결(`/api/mcp`)을 이미 쓰고 있으면 문서 도구가 들어 있으므로 이 주소를 따로 추가할 필요가 없습니다.
- 서버에서 문서 기능이 꺼져 있으면 404 가 옵니다.

### Claude.ai · ChatGPT (웹·데스크톱·모바일 앱)

설정 > 커넥터 > 커스텀 커넥터 추가에 위 주소를 넣고 연결합니다.

## 4. 문제 해결

| 증상 | 확인할 것 |
|---|---|
| `/mcp` 에서 needs authentication · 401 | `/mcp` → appbox → Authenticate 로 다시 로그인. 콘솔에서 연결을 해제했거나 비밀번호를 바꿨으면 다시 연결해야 합니다 |
| 403 `insufficient_scope` | 연결 권한 부족. 다시 연결하고 허용 화면에서 권한을 허용 |
| 403 (그 밖) | 계정 정지 등 — 앱박스 고객지원 문의 |
| 404 | 주소 오류 — 수동 연결이면 주소가 `https://console.appboxapp.com/api/mcp` 인지 확인 |
| `APP_NOT_ALLOWED` | 연결할 때 허용하지 않은 앱(연결한 뒤 만든 앱 포함). 다시 연결하면서 그 앱을 체크하세요 |

## 버전

- 플러그인 `version` 은 SemVer 이며 서버(MCP `appbox`) 버전과 별개입니다. 변경 이력은 [CHANGELOG.md](CHANGELOG.md).
- `version` 이 올라가야 사용자에게 업데이트가 전달됩니다.

## 배포 규칙 (관리자용)

| 브랜치 | 용도 |
|---|---|
| `main` | 고객 배포본 — 버전을 올리고 태그(`v1.0.1` …)를 붙인 것만 |
| `develop` | 수정·시험. 이 브랜치를 내려받은 사본을 로컬 경로로 등록해 시험(`claude plugin marketplace add ./`) |

- `plugins/appbox/.mcp.json` 의 주소는 **운영 주소로 고정**입니다. 시험 서버로 시험할 때는 로컬 사본에서만 주소를 바꾸고 커밋하지 않습니다(시험 방법은 사내 문서).
- **서버 먼저, 플러그인 나중** — 스킬이 새 서버 도구를 쓰면, 그 도구가 운영 서버에 반영된 뒤에 `main` 으로 배포합니다. 순서가 바뀌면 고객 플러그인이 없는 도구를 부릅니다.
- 배포: `develop` → `main` 병합 · 세 매니페스트(`.claude-plugin/marketplace.json` · `plugins/appbox/.claude-plugin/plugin.json` · `plugins/appbox/.codex-plugin/plugin.json`)의 `version` 을 함께 올림 · `CHANGELOG.md` · 태그 · `claude plugin validate --strict ./` 통과.

## 문의

appbox@appboxapp.com

## 라이선스

MIT — [LICENSE](LICENSE). 앱박스 서비스 이용은 [서비스 약관](https://www.appboxapp.com/policy/terms/use)과 [개인정보 처리방침](https://www.appboxapp.com/policy/privacy)을 따릅니다.
