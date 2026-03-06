# Messaging Platform Server Design

## Part A. 서버 설계

본 문서는 기존 메시지 시스템을 **확장성과 Provider 독립성**을 고려한
구조로 재설계하기 위한 서버 설계 문서입니다.

설계 목표:

-   Provider 의존성 제거
-   Queue 기반 비동기 메시지 처리
-   채널 확장성 확보 (SMS / MMS / AlimTalk 등)
-   Provider 교체 또는 추가 용이성 확보
-   Webhook 기반 상태 업데이트

------------------------------------------------------------------------

# 1. 전체 시스템 아키텍처

``` mermaid
flowchart LR

Client --> API

API --> MessageService

MessageService --> DB[(Message DB)]

MessageService --> Queue[(Message Queue)]

Queue --> Worker

Worker --> ProviderAdapter

ProviderAdapter --> CloudMsg
ProviderAdapter --> SendTalk

CloudMsg --> Webhook
SendTalk --> Webhook

Webhook --> WebhookHandler

WebhookHandler --> DB
```

설명

  구성 요소         설명
  ----------------- ---------------------------
  API               메시지 생성 요청 수신
  MessageService    메시지 생성 및 Queue 등록
  Queue             비동기 메시지 처리
  Worker            실제 Provider 호출
  ProviderAdapter   Provider API 추상화
  WebhookHandler    메시지 상태 업데이트

------------------------------------------------------------------------

# 2. 메시지 처리 Sequence Diagram

``` mermaid
sequenceDiagram

participant Client
participant API
participant Queue
participant Worker
participant Provider
participant Webhook

Client->>API: Send Message Request

API->>Queue: enqueue message

Queue->>Worker: message job

Worker->>Provider: send message

Provider-->>Worker: send response

Provider->>Webhook: delivery result

Webhook->>API: update message status
```

------------------------------------------------------------------------

# 3. 주요 설계 포인트

## 3.1 Provider 추상화 (Adapter Pattern)

Provider API 구조가 서로 다르기 때문에 Adapter Layer를 도입합니다.

``` mermaid
flowchart TB

MessageService

MessageService --> MessageProvider

MessageProvider --> CloudMsgProvider
MessageProvider --> SendTalkProvider
```

### Provider Interface

``` text
interface MessageProvider {

 sendSMS(message)

 sendMMS(message)

 sendAlimTalk(message)

}
```

Provider 구현체

-   CloudMsgProvider
-   SendTalkProvider

각 Provider는 다음 책임을 가집니다.

-   인증 처리
-   Payload 변환
-   응답 변환
-   오류 처리

------------------------------------------------------------------------

# 4. 메시지 데이터 모델

## Message

  필드        설명
  ----------- ----------------------
  id          메시지 ID
  userId      사용자
  channel     SMS / MMS / ALIMTALK
  provider    선택된 Provider
  content     메시지 내용
  status      메시지 상태
  createdAt   생성 시간

------------------------------------------------------------------------

## UserGroup

  필드   설명
  ------ -----------
  id     그룹 ID
  name   그룹 이름

------------------------------------------------------------------------

# 5. Message State Machine

``` mermaid
stateDiagram-v2

CREATED --> QUEUED
QUEUED --> SENT
SENT --> DELIVERED
SENT --> FAILED
```

  상태        설명
  ----------- ---------------
  CREATED     메시지 생성
  QUEUED      Queue 등록
  SENT        Provider 전달
  DELIVERED   전달 완료
  FAILED      전달 실패

------------------------------------------------------------------------

# 6. Provider 선택 전략

Provider 선택은 **메시지 생성 시점**에 수행됩니다.

선택 기준 예시

-   채널 지원 여부
-   가격
-   Rate Limit
-   장애 여부

예시

``` text
if channel == ALIMTALK
  provider = SendTalk
else
  provider = CloudMsg
```

------------------------------------------------------------------------

# 7. Batch 처리 전략

CloudMsg는 Batch API를 지원합니다.

    POST /messages/batch
    max 1000 messages

Worker 처리 전략

``` text
if provider == CLOUDMSG
  batch send

if provider == SENDTALK
  single send
```

------------------------------------------------------------------------

# 8. Webhook 처리

Provider는 메시지 상태를 Webhook으로 전달합니다.

``` mermaid
flowchart LR

Provider --> WebhookEndpoint

WebhookEndpoint --> Validator

Validator --> StatusUpdater

StatusUpdater --> DB
```

Webhook 검증

-   Signature 검증
-   Timestamp 검증

------------------------------------------------------------------------

# 9. Phone Number Normalization

Provider마다 전화번호 형식이 다릅니다.

내부 표준

    +821012345678

Provider 변환

  Provider   형식
  ---------- ---------------
  CloudMsg   +821012345678
  SendTalk   01012345678

------------------------------------------------------------------------

# 10. 작업 단계

  단계   목표                    산출물
  ------ ----------------------- ----------------
  1      메시지 모델 설계        DB Schema
  2      Message API 구현        REST API
  3      Queue 도입              Worker
  4      Provider Adapter 구현   Provider Layer
  5      Webhook 구현            상태 업데이트

------------------------------------------------------------------------

# 11. 리스크 및 대응

  위험 요소           영향               대응
  ------------------- ------------------ -------------------
  Provider API 장애   메시지 발송 실패   Provider fallback
  Rate Limit          메시지 지연        Queue buffering
  Webhook 누락        상태 불일치        Retry Job

------------------------------------------------------------------------

# 12. 확장성 설계

이 구조는 다음 확장이 가능합니다.

새 Provider 추가

    class TwilioProvider implements MessageProvider

새 채널 추가

    sendWhatsApp()
    sendPush()

------------------------------------------------------------------------

# 13. AI 활용 프롬프트

본 설계 문서는 AI 도구를 활용하여 구조 설계와 문서화를 보조받아
작성되었습니다.

사용한 프롬프트 예시

### 아키텍처 설계

    Design a scalable messaging platform architecture supporting multiple providers such as SMS and AlimTalk.
    Include queue based asynchronous processing and provider abstraction.

### API 설계

    Design REST APIs for a messaging platform that supports message creation, queue processing, and webhook status updates.

### README 구조화

    Write a technical README for a messaging system architecture including diagrams, provider abstraction, queue based processing and risk analysis.

AI는 설계 보조 및 문서 정리에 활용되었으며 실제 구조와 의사결정은
작성자가 검토 후 반영하였습니다.

------------------------------------------------------------------------

# 결론

본 설계는

-   Provider 의존성 제거
-   Queue 기반 확장성
-   Webhook 기반 상태 관리

를 중심으로 메시지 시스템의 **확장성과 안정성**을 확보하는 것을 목표로
합니다.
