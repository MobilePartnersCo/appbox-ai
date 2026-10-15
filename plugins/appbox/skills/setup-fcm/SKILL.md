---
name: setup-fcm
description: 앱박스 앱에 FCM(Firebase 서비스 계정 키)을 발급·등록하는 과정을 단계별로 안내한다. "FCM 등록 어떻게 해요", "Firebase 키 어디서 받아요", "푸시 키 등록", "FCM 미등록이라고 나와요" 같은 요청이나, 시작 안내에서 다음 단계가 FCM 일 때 사용한다. 조회만 하고 키 파일은 다루지 않는다.
argument-hint: "[앱 코드]"
allowed-tools: mcp__plugin_appbox_appbox__appbox_get_onboarding_status mcp__plugin_appbox_appbox__appbox_list_apps mcp__plugin_appbox_appbox__appbox_check_app_setup mcp__plugin_appbox_appbox__appbox_search_docs mcp__plugin_appbox_appbox__appbox_get_doc
---

# 앱박스 FCM 발급·등록 안내

Firebase 에서 서비스 계정 키(JSON)를 받아 앱박스 콘솔에 등록할 때까지 **한 단계씩** 안내한다.

사용자가 넘긴 인자: `$ARGUMENTS` (비어 있을 수 있음)

## 반드시 지킬 것

- **키 파일(JSON)·APNs 키(.p8)의 내용을 대화로 받지 않는다.** 고객이 붙여 넣거나 파일을 첨부하려 하면 멈추게 하고, 앱박스 콘솔 화면에서 직접 「파일 선택」으로 올리게 한다. 이미 붙여 넣었다면 그 키를 Firebase 에서 삭제하고 새로 발급하도록 권한다.
- Google·Apple 로그인, 2단계 인증은 고객이 직접 한다.
- 한 번에 한 단계만 안내하고, 고객이 끝냈다고 하면 다음으로 넘어간다.
- 가이드 주소와 콘솔 주소는 도구가 준 `guideUrls`·`consoleUrl` 을 그대로 쓴다.
- 화면 이름이 아래 절차와 다르다고 하면 `appbox_search_docs`(예: `FCM`, `비공개 키`, `iOS 앱 등록`) → `appbox_get_doc` 으로 최신 가이드를 읽어 맞춘다(문서 도구가 없으면 `guideUrls` 를 안내).

## 절차

### 0. 대상 앱과 현재 상태

`appbox_get_onboarding_status` 를 호출한다(인자가 앱 코드면 `{ "app": "<앱 코드>" }`).

- `CHOOSE_APP` 이면 앱을 고르게 한 뒤 다시 호출한다. `NO_APP` 이면 `onboarding` 스킬로 넘긴다.
- `steps` 의 `FCM` 항목이 `DONE` 이면 "이미 등록돼 있다"고 알리고 멈춘다. 키를 바꾸려는 것이면 6단계 화면에서 다시 올리면 된다고 안내한다.
- `FCM` 항목의 `consoleUrl`(FCM 설정 화면)과 `guideUrls`(Android 앱 등록 · iOS 앱 등록 · 비공개 키 등록 가이드)를 기억해 둔다.

시작 전에 두 가지를 묻는다: **Android·iOS 중 무엇을 내는지**, **앱에서 Google 로그인을 쓸 것인지**.

### 1. Firebase 프로젝트

Firebase 콘솔(console.firebase.google.com)에 로그인해 프로젝트를 만든다. 이미 있으면 그 프로젝트를 고른다.

### 2. Firebase 에 앱 추가

| 플랫폼 | 할 일 |
|---|---|
| Android | 「앱 추가」› Android › **패키지명** 입력 › 앱 등록. 설정 파일 다운로드 화면은 건너뛰어도 된다 |
| iOS | 「앱 추가」› iOS › **번들 ID** 입력 › 앱 등록. 먼저 Apple Developer 에 같은 번들 ID 가 있어야 한다 |

- 이미 스토어에 낸 앱이 있으면 그 앱의 패키지명·번들 ID 를 쓴다.
- 새로 앱박스 빌드로 만드는 앱이면 패키지명을 여기서 정한다. 도메인을 거꾸로 쓴 형태(예 `com.example.app`)가 일반적이다. **iOS 번들 ID 에는 `_` 를 쓸 수 없다.**
- 이 단계를 건너뛰면 6단계에서 "Android 패키지명 또는 iOS 번들 ID가 존재하지 않습니다" 오류가 난다.

### 3. (Google 로그인을 쓸 때만) Firebase Authentication

Firebase › Authentication › 로그인 방법 › Google › 사용 설정. **키를 앱박스에 올리기 전에** 켠다. 이미 올린 뒤에 켰다면 6단계 화면에서 「파일 선택」으로 창을 열어 다시 「적용」한다.

### 4. (iOS 만) APNs 인증 키

1. Apple Developer › Certificates, Identifiers & Profiles › Keys › 「+」
2. 키 이름 입력, **Apple Push Notification service** 체크, Configure 에서 환경을 **Sandbox & Production** 으로 선택 › Save › Continue › Register › Download
3. **Key ID 와 Team ID 를 적어 둔다.** .p8 파일은 **한 번만** 받을 수 있다.
4. Firebase › 프로젝트 설정 › 클라우드 메시징 › Apple 앱 구성 › APNs 인증 키 업로드. **같은 .p8 을 개발·프로덕션 두 칸에 모두** 올리고 Key ID·Team ID 를 넣는다.

.p8 은 앱박스가 아니라 **Firebase 에** 올린다. 대화에 올리지 않는다.

### 5. 서비스 계정 키(JSON) 발급

1. Firebase › 프로젝트 설정 › 클라우드 메시징 탭에서 **Firebase Cloud Messaging API(V1)** 가 "사용 설정됨"인지 확인한다. 꺼져 있으면 Google Cloud Console › API 에서 켠다.
2. 서비스 계정 탭 › **새 비공개 키 생성** › 키 생성. JSON 파일이 PC 에 내려받아진다.

**막히는 경우** — "Key creation is not allowed" 류의 오류: 회사 Google 계정(조직)에서 서비스 계정 키 생성이 정책으로 막혀 있다. 고객사의 Google Cloud 조직 정책 관리자에게 이 프로젝트의 서비스 계정 키 생성 제한(`iam.disableServiceAccountKeyCreation`) 예외를 요청해야 한다. 개인 Google 계정의 프로젝트에서는 보통 생기지 않는다.

### 6. 앱박스 콘솔에 등록

1. 0단계에서 받은 FCM 설정 화면(`consoleUrl`)을 연다. `null` 이었으면 콘솔 › 앱 선택 › 앱 빌드 › FCM 설정.
2. 「[FCM] 비공개 키 (JSON 파일)」의 **파일 선택**을 눌러 5단계에서 받은 JSON 을 **고객이 직접** 고른다.
3. 창에 Firebase 에 등록한 앱 목록이 뜬다. 「Android 패키지」·「iOS 번들」을 하나 이상 고르고 **적용**.
4. 「FCM 설정이 저장되었습니다.」가 나오면 끝.

- 앱 소유자만 저장할 수 있고, 데모 앱은 바꿀 수 없다.
- 파일은 `.json`, 1MB 이하여야 한다.

### 7. 확인

`appbox_get_onboarding_status` 를 다시 호출한다. `FCM` 이 `DONE` 이면 "등록 완료"를 알리고 응답의 `nextStep`(보통 빌드 또는 SDK 연동)을 한 줄로 안내한다. 이어서 진행하려면 `onboarding` 스킬을 쓴다.

여전히 `TODO` 면 6단계에서 「적용」까지 눌렀는지, 저장 완료 문구를 봤는지 확인한다.

## 자주 막히는 곳

| 증상 | 원인과 안내 |
|---|---|
| 패키지명·번들 ID 가 존재하지 않는다는 오류 | Firebase 에 앱을 추가하지 않았다 → 2단계 |
| 키 생성이 허용되지 않는다는 오류 | 조직 정책 → 5단계 "막히는 경우" |
| iOS 만 푸시가 안 온다 | APNs 키를 Firebase 에 안 올렸거나 한 칸만 올렸다 → 4단계 |
| Google 로그인이 앱에서 안 된다 | Authentication 을 키 등록 뒤에 켰다 → 3단계대로 다시 「적용」 |
| 저장 버튼이 안 눌린다 | 앱 소유자가 아니거나 데모 앱이다 |

## 오류 처리

도구 오류는 `onboarding`·`check-setup` 스킬의 오류 표를 따른다. `appbox_get_onboarding_status` 가 도구 목록에 없으면 `appbox_list_apps` → `appbox_check_app_setup` 으로 FCM 상태를 확인하고, 화면은 "콘솔 › 앱 빌드 › FCM 설정"으로 안내한다.

## 하지 말 것

- 키 파일·.p8·비밀번호를 대화로 받지 않는다.
- 브라우저를 대신 조작하거나 파일을 대신 올리지 않는다.
- 등록이 됐다고 추측해 말하지 않는다. 7단계의 도구 응답으로만 판단한다.
