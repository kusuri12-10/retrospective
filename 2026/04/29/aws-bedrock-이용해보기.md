# AWS 베드록 이용해보기

> 2026-04-29

---

## 1. 사용하게 된 계기

대회에 출품하기 위한 협업 SaaS를 기획하던 중, 구현해야 하는 기능 중에 **AI 자동 일정 생성**과 **문서 자동화** 두 가지가 있었다. 이 기능들을 직접 구현해야 하는 상황이 됐다.

AI 연동 방법을 찾아보다 보니 선택지가 몇 가지 있었다.

- Spring AI
- OpenAI API 직접 호출
- AWS Bedrock

결국 **AWS Bedrock**을 선택했다.

---

## 2. Bedrock을 선택한 이유

Spring AI도 분명히 매력적인 선택지였다. Spring 생태계에 자연스럽게 녹아들고, 설정도 간단하다. 그런데 실제로 붙여보려고 하니 몇 가지가 마음에 걸렸다.

**Spring AI의 추상화가 너무 과했다.**

내가 원하는 건 SSE(Server-Sent Events) 스트리밍을 직접 제어하는 것이었다. 응답이 생성되는 대로 클라이언트에 밀어주고 싶었는데, Spring AI의 추상화 레이어를 통하면 그 흐름을 내 마음대로 핸들링하기가 까다로웠다.

반면 Bedrock SDK를 직접 쓰면 스트리밍 응답 처리 흐름을 처음부터 끝까지 내가 컨트롤할 수 있었다. 또 기존 인프라가 AWS 위에 올라가 있었기 때문에 IAM 기반 인증을 그대로 활용할 수 있다는 것도 이점이었다.

요약하자면:

- **스트리밍 제어권**이 필요했으나 Spring AI 추상화로는 한계가 있었다.
- **AWS 인프라와의 통합**: IAM 인증을 자연스럽게 연결할 수 있었다
- **모델 선택 유연성**: Bedrock에서 제공하는 여러 모델 중 필요에 따라 고를 수 있었다

---

## 3. 사용 방법

기본적인 흐름은 이렇다.

```
클라이언트 요청
    → Spring 백엔드 (Kotlin)
    → AWS Bedrock SDK 호출 (스트리밍)
    → SSE로 클라이언트에 응답 전송
```

Bedrock SDK에서 `InvokeModelWithResponseStreamRequest`를 사용해 스트리밍 응답을 받고, Spring의 `SseEmitter`를 통해 클라이언트에 청크 단위로 내려줬다.

```kotlin
val request = InvokeModelWithResponseStreamRequest.builder()
    .modelId("anthropic.claude-3-5-sonnet-20241022-v2:0")
    .body(SdkBytes.fromUtf8String(requestBody))
    .contentType("application/json")
    .build()

val response = bedrockClient.invokeModelWithResponseStream(request)

response.subscribe { event ->
    // 청크 처리 후 SseEmitter로 전송
}
```

문서 자동화는 프롬프트를 구조화해서 특정 포맷의 텍스트를 생성하도록 유도했고, 일정 생성은 컨텍스트(문서 내용 등)를 함께 넘겨서 JSON 형식으로 결과를 받아 파싱했다.

---

## 4. 트러블슈팅 (시행착오)

### 스트리밍 응답 파싱이 예상보다 복잡했다

가장 많이 고생한 부분이 여기였다.

Bedrock에서 오는 스트리밍 응답은 Anthropic 메시지 포맷 기준으로 여러 타입의 이벤트가 섞여서 온다.

```
message_start
content_block_start
content_block_delta  ← 실제 텍스트가 여기에
content_block_delta
content_block_stop
message_delta
message_stop
```

처음에는 모든 이벤트를 그냥 파싱해서 텍스트만 뽑으면 되겠거니 했는데, 이벤트 타입마다 JSON 구조가 달랐다. `content_block_delta`에서 `delta.text`를 꺼내야 하는데 다른 타입에서 같은 필드를 접근하면 NPE가 터졌다.

결국 이벤트 타입을 먼저 확인하고 분기 처리하는 구조로 바꿨다.

```kotlin
val eventType = jsonNode["type"]?.asText() ?: return@forEach
when (eventType) {
    "content_block_delta" -> {
        val text = jsonNode["delta"]?.get("text")?.asText() ?: ""
        emitter.send(text)
    }
    "message_stop" -> emitter.complete()
    else -> { /* 무시 */ }
}
```

### SSE 연결이 중간에 끊기는 문제

긴 응답을 스트리밍할 때 중간에 SSE 연결이 끊기는 현상이 있었다. 원인은 Spring 쪽 타임아웃 설정이었다. `SseEmitter`의 기본 타임아웃이 짧게 잡혀 있어서 긴 응답이 오기 전에 연결이 끊겼다.

`SseEmitter(0L)`로 타임아웃을 무제한으로 설정하고, 에러 핸들링도 추가했다.

---

## 5. 느낀 점 및 회고

직접 SDK를 다뤄보니 블랙박스가 하나씩 걷혀가는 느낌이 있었다. Spring AI가 추상화로 감춰놓은 부분들, 즉 스트리밍 이벤트 타입이나 응답 구조 같은 것들을 직접 마주하면서 LLM API가 어떻게 동작하는지 조금 더 구체적으로 이해하게 됐다.

반면 추상화가 없다는 건 그만큼 직접 챙겨야 할 게 많다는 뜻이기도 하다. 파싱, 에러 핸들링, 타임아웃 관리를 전부 직접 해야 했다. 잘 추상화된 라이브러리가 왜 가치 있는지도 역설적으로 느꼈다.

프롬프트 설계도 생각보다 공수가 많이 들었다. 특히 JSON을 뽑아야 할 때 모델이 마크다운 코드 블록으로 감싸서 주는 경우가 있어서 파싱 전처리가 필요했다. 프롬프트에 명시적으로 "JSON만 응답하라"고 써도 간혹 어기는 경우가 있었다.

---

## 6. 앞으로 할 것

- **프롬프트 고도화**: 현재는 단순 텍스트 프롬프트인데, few-shot 예시를 추가해서 응답 품질을 올려보고 싶다
- **Spring AI 재검토**: 이번에는 스트리밍 제어 때문에 직접 SDK를 선택했지만, Spring AI도 버전이 올라가면서 개선되고 있는 것 같아서 다시 한번 살펴볼 예정이다
- **Bedrock Knowledge Base 연동**: RAG(Retrieval-Augmented Generation) 구조를 붙여서 사내 문서 기반 질의응답을 만들어보고 싶다
- **비용 모니터링**: 토큰 사용량이 생각보다 빠르게 쌓이는 걸 느꼈다. CloudWatch로 사용량 대시보드를 만들어 두는 게 좋겠다

---

추상화는 편하지만, 한 번쯤은 그 아래를 들여다봐야 진짜로 아는 것 같다.