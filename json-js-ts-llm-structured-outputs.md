> **Keywords:** #JSON #JavaScript #TypeScript #LLM #PromptEngineering #StructuredOutputs #AppsScript
> **Date/Time:** 2026-04-03 11:50 AM (Asia/Seoul)
> **Summary:** JSON은 데이터, JavaScript/TypeScript는 그 데이터를 다루는 코드이며, LLM 문항 생성에서는 입력은 읽기 쉬운 구조화 텍스트, 출력은 JSON Schema 기반 구조화 출력이 가장 안정적입니다.

아래처럼 이해하시면 가장 쉽습니다.

## 1) JSON, JavaScript, TypeScript 차이

### JSON

JSON은 **데이터만 담는 형식**입니다.
계산도 못 하고, 함수도 못 넣고, 그냥 값만 담습니다.
앱끼리 데이터를 주고받을 때 가장 많이 씁니다. ([OpenAI 개발자][1])

```json
{
  "cefrLevel": "B1",
  "wordCount": 5,
  "questionType": "multiple_choice"
}
```

### JavaScript

JavaScript는 **실제로 동작하는 코드**입니다.
조건문, 반복문, 함수, 계산이 가능합니다.

```javascript
function add(a, b) {
  return a + b;
}
```

### TypeScript

TypeScript는 JavaScript에 **"이 값은 문자다 / 숫자다" 같은 설명표**를 더한 것입니다.
즉, 코드를 더 안전하게 쓰게 도와주는 언어입니다.

```typescript
function add(a: number, b: number): number {
  return a + b;
}
```

핵심만 말하면:

* **JSON = 데이터**
* **JavaScript = 실행 코드**
* **TypeScript = 더 안전한 실행 코드**

---

## 2) 문항 생성할 때 어떤 형식이 제일 좋은가

가장 실무적으로 좋은 방식은 이것입니다.

**입력(prompt)은 사람이 읽기 쉬운 구조화 텍스트**
**출력(response)은 JSON Schema로 강제된 JSON**

이 방식이 가장 안정적입니다.
OpenAI는 `json_schema` 기반 Structured Outputs를 통해 모델이 지정한 스키마를 따르도록 할 수 있다고 안내하고 있고, `json_object`는 더 오래된 방식이라고 설명합니다. ([OpenAI 개발자][1])
Gemini도 Structured Outputs에서 JSON Schema를 지원하며, 최종 응답을 특정 구조로 맞춰야 할 때는 Function Calling보다 Structured Outputs를 쓰라고 안내합니다. 또한 Gemini 3 계열은 길고 복잡한 프롬프트보다 **직접적이고 명확한 지시**를 더 잘 따른다고 설명합니다. ([Google AI for Developers][2])
Claude도 구조화 출력이 스키마 준수를 보장하는 방향으로 제공되고 있습니다. ([Claude API Docs][3])

즉, 현재 기준으로는:

* **사람이 쓰는 프롬프트 본문** → 자연어 + 섹션 구조
* **모델이 내놓는 결과** → JSON Schema 기반 구조화 JSON

이 조합이 가장 좋습니다. ([OpenAI 개발자][1])

---

## 3) 왜 프롬프트 자체를 JSON으로만 쓰지 않는가

초보자 입장에서는 "그냥 다 JSON으로 쓰면 더 깔끔한가?"라고 생각하기 쉽습니다.
그런데 문항 생성에서는 보통 **프롬프트 내용 자체는 설명문 형태**가 더 잘 먹힙니다.

이유는 간단합니다.

LLM은 원래 **자연어 지시를 이해하는 데 강한 모델**이기 때문입니다.
그래서 아래처럼 쓰는 편이 좋습니다.

* 역할
* 목표
* 입력 데이터
* 제약 조건
* 출력 형식
* 예시

이런 식의 **구조화된 글**이 가장 다루기 쉽습니다.
그리고 결과만 JSON으로 받으면, 앱에서 바로 저장·검사·재사용하기 좋습니다.
이건 OpenAI와 Gemini가 모두 텍스트 생성과 구조화 출력(JSON Schema)을 함께 지원하는 흐름과도 잘 맞습니다. ([OpenAI 개발자][4])

---

## 4) 문항 생성용 최적 형식: 제가 추천하는 표준

### 입력은 이렇게

프롬프트는 **섹션형 텍스트**로 만드세요.

```text
[ROLE]
You are an English assessment expert.

[TASK]
Generate 3 multiple-choice vocabulary questions.

[LEARNER PROFILE]
CEFR: B1
Target grade: Korean high school

[INPUT WORDS]
- analyze: to examine carefully
- contrast: to compare differences
- maintain: to keep something in good condition

[CONSTRAINTS]
- 4 options each
- only 1 correct answer
- use natural school-level English
- avoid ambiguous distractors

[OUTPUT]
Return valid JSON only, following the given schema.
```

이 형식이 좋은 이유는:

1. 사람이 읽기 쉽고 수정이 쉽고
2. 프롬프트 디버깅이 쉽고
3. 모델도 역할과 제약을 더 잘 이해하기 때문입니다.
   Gemini는 특히 직접적이고 명확한 지시를 권장합니다. ([Google AI for Developers][5])

---

## 5) 출력은 이렇게

출력은 **JSON Schema 기반**으로 강제하는 것이 가장 좋습니다.

예를 들면 결과를 이런 모양으로 받는 것입니다.

```json
{
  "items": [
    {
      "question": "Choose the word closest in meaning to 'analyze'.",
      "options": ["ignore", "examine", "damage", "repeat"],
      "answer": "examine",
      "explanation": "Analyze means to examine something carefully."
    }
  ]
}
```

이 방식의 장점은 아주 큽니다.

* Apps Script에서 `JSON.parse()`로 바로 읽기 쉽습니다.
* Google Sheets에 바로 저장하기 쉽습니다.
* 필수 항목 누락 여부를 검사하기 쉽습니다.
* 정답, 해설, 선택지 개수 같은 규칙을 자동 검증하기 쉽습니다.

OpenAI는 `json_schema`가 스키마 준수를 보장하는 방식이라고 안내하고 있고, `json_object`보다 권장합니다. ([OpenAI 개발자][1])

---

## 6) 당신의 프로젝트 기준으로 가장 좋은 조합

당신처럼 **Google Sheets + Apps Script + 문항 생성 자동화**를 할 때는 아래 조합이 제일 좋습니다.

### 추천 조합

* **시트 저장 데이터**: JSON처럼 다루기 쉬운 열 구조
* **프롬프트 생성 코드**: JavaScript 또는 TypeScript
* **LLM 입력**: 구조화된 텍스트 프롬프트
* **LLM 출력**: JSON Schema 기반 JSON
* **앱 내부 타입 관리**: TypeScript 타입 또는 유사한 검사 로직

즉,

**TypeScript는 만드는 쪽**
**JSON은 주고받는 쪽**

이라고 생각하시면 됩니다.

---

## 7) 한 줄 결론

문항 생성에서 가장 좋은 형식은
**"입력은 읽기 쉬운 구조화 텍스트, 출력은 JSON Schema로 강제된 JSON"** 입니다.
이 방식이 지금 가장 안정적이고, 자동 저장·검증·재생성에도 가장 유리합니다. ([OpenAI 개발자][1])

원하시면 다음 답변에서 바로
**"영어 문항 생성용 최적 프롬프트 템플릿 + JSON 스키마 예시"**를 당신의 Google Sheets / Apps Script 구조에 맞춰 만들어드리겠습니다.

[1]: https://developers.openai.com/api/docs/guides/structured-outputs/?utm_source=chatgpt.com "Structured model outputs | OpenAI API"
[2]: https://ai.google.dev/gemini-api/docs/structured-output "Structured outputs  |  Gemini API  |  Google AI for Developers"
[3]: https://docs.anthropic.com/en/release-notes/api "Claude Platform - Claude API Docs"
[4]: https://developers.openai.com/api/docs/guides/text/?utm_source=chatgpt.com "Text generation | OpenAI API"
[5]: https://ai.google.dev/gemini-api/docs/gemini-3 "Gemini 3 Developer Guide"
