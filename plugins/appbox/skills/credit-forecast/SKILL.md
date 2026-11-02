---
name: credit-forecast
description: 앱박스 앱의 이번 달 푸시, 인앱 사용 추세와 제공량, 추가 크레딧 잔여로 언제쯤 모자랄지 가늠한다. "크레딧 언제 떨어져?", "이번 달 제공량 넘을까?", "충전해야 해?" 같은 요청에 사용한다. 조회만 한다. 「운영 현황 조회」 권한이 있는 연결에서만 쓸 수 있다.
argument-hint: "[앱 코드]"
allowed-tools: mcp__plugin_appbox_appbox__appbox_list_apps mcp__plugin_appbox_appbox__appbox_get_billing mcp__plugin_appbox_appbox__appbox_get_dashboard mcp__plugin_appbox_appbox__appbox_get_plans
---

# 앱박스 크레딧 소진 예측

사용자가 넘긴 인자: `$ARGUMENTS` (비어 있을 수 있음)

이 스킬은 **조회만** 한다. 결제, 충전, 플랜 변경은 하지 않고 콘솔 결제 화면을 안내한다.

## 절차

1. **사용량** — `appbox_get_billing` 에 `{ app }`. 플랜, 이번 달 발송 수(`push.sent`), 월 제공량(`push.providedThisMonth`), 인앱 MAU 와 제공 MAU, 추가 크레딧 잔여(`pushCredit`, `inappCredit`).
   - `push.usedIncludingCredit` 은 콘솔 사용량 화면 값으로 크레딧 누적 사용이 더해져 있어 이번 달 발송 수와 다르다. 이번 달 사용을 말할 때는 `push.sent` 를 쓴다.
2. **하루 평균** — `appbox_get_dashboard` 의 이번 달 일별 발송(`daily`)으로 하루 평균 발송량을 구한다. 월초라 지난 날이 며칠 안 되면 표본이 적어 크게 틀릴 수 있다고 밝힌다.
3. **예측** — 이 속도면 월 제공량을 넘는 날과, 넘은 뒤 추가 크레딧이 버티는 기간을 계산식과 함께 보여 준다. 추정이라고 밝힌다.
4. **요금** — 필요하면 `appbox_get_plans` 로 요금제를 보여 준다. 부가세와 추가 크레딧 단가는 콘솔 결제 화면을 안내한다.
