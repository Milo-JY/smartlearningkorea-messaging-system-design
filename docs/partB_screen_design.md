# Part B. Admin Configuration UI Design

## 1. 목표 (Objective)

조직 관리자가 자신의 조직에서 사용할 메시지 전송 Provider를
설정하고 관리할 수 있는 관리자 설정 화면을 설계한다.

이 화면의 주요 목적은 다음과 같다.

- 조직별 메시지 Provider 설정
- Provider 인증 정보 관리
- 설정 오류 방지
- 메시지 발송 안정성 확보

---

# 2. User Flow

관리자는 다음 흐름을 통해 Provider 설정을 변경한다.

Organization Settings  
→ Messaging Provider Settings  
→ Provider 선택  
→ Provider 인증 정보 입력  
→ Test Connection  
→ Save Settings

---

# 3. UI Design

## 3.1 Provider Selection

관리자는 사용할 메시지 Provider를 선택한다.

지원 Provider 예시

- CloudMsg
- SendTalk

UI 방식
(●) CloudMsg
( ) SendTalk

Provider 선택 시 해당 Provider의 설정 입력 폼이 표시된다.

---

## 3.2 Provider Configuration Form

Provider별로 필요한 인증 정보 입력 폼이 동적으로 표시된다.

### CloudMsg

Access Key
Secret Key
Sender Phone Number
Sender Profile ID

### SendTalk

API Key
Default Sender Phone

---

## 3.3 Connection Test

설정 저장 전 Provider 연결 테스트 기능을 제공한다.

[ Test Connection ]

검증 항목

- API 인증
- 발신 번호 유효성
- Provider 연결 상태

---

## 3.4 Save Settings

설정 저장 시 다음 절차가 수행된다.

1. 입력값 Validation
2. Provider 인증 검증
3. 설정 저장

---

# 4. UX Design Decisions

## Provider Selection UI

### Option A — Dropdown

### Option B — Radio Button

선택  
**Radio Button**

이유

- Provider 수가 많지 않음
- Admin UX에서 실수 감소
- 선택 상태 명확

---

## Configuration Input 방식

### Option A — 별도 페이지

### Option B — Dynamic Form

선택  
**Dynamic Form**

이유

- 페이지 이동 감소
- 설정 흐름 단순화
- Admin UX 개선

---

## 설정 저장 방식

### Option A — 바로 저장

### Option B — Test 후 저장

선택  
**Test Connection 후 저장**

이유

- 잘못된 API Key 방지
- 운영 오류 감소
- 메시지 발송 실패 예방

---

# 5. Edge Cases

## 잘못된 API Key

Authentication Failed
Invalid API Key

---

## 발신 번호 미등록

Invalid Sender Phone Number

---

## Provider 변경 시 설정 충돌

Changing provider will require new configuration.
Continue?

---

## 저장 후 메시지 발송 실패

Message delivery errors detected.

Webhook 기반 모니터링으로 감지 가능

---

# 6. UX Improvements

### Test Message

관리자가 실제 메시지를 테스트 발송할 수 있다.

[ Send Test Message ]

---

### Audit Log

Provider 설정 변경 기록

2024-01-15
Provider changed
SendTalk → CloudMsg

---

### Input Validation

- 전화번호 형식 검증
- 필수값 체크
- API Key 검증

---

# 7. Wireframe

(기존 wireframe 유지)

---

# 8. AI 활용

본 화면 설계 문서는 AI 도구를 활용하여 UX Flow 정리와
문서 구조화를 보조받아 작성되었습니다.

AI는 다음 작업에 활용되었습니다.

- Admin UI 흐름 설계
- UX Edge Case 도출
- Markdown 기반 설계 문서 정리

사용한 프롬프트 예시

### UX Flow 설계

Design an admin configuration UI for selecting and configuring
a messaging provider such as SMS or AlimTalk.
Include provider selection, authentication configuration,
test connection and save flow.

### UX Edge Case 분석

List UX edge cases when an admin configures
a messaging provider including authentication failure,
invalid sender number and provider change scenarios.

### 문서 구조화

Write a structured design document for an admin configuration UI
including user flow, design decisions, edge cases and wireframe.

AI는 설계 보조 및 문서 정리에 활용되었으며  
최종 설계와 UX 결정 사항은 작성자가 검토 후 반영하였습니다.
