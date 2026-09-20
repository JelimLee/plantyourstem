---
name: read
description: 영어 STEM 문서(PDF/텍스트)를 브라우저에서 읽는 로컬 학습 페이지로 만든다. 단어를 누르면 문맥 뜻, 문단마다 한국어 번역, 문장 구조 색칠, 다 읽으면 씨앗을 눌러 질문으로 개념을 발아시켜 나무로 키운다. "이 PDF 공부하자", "읽는 것 도와줘", "번역 페이지 만들어줘"에 발동. 인자는 파일 경로와 선택적 범위(예 `paper.pdf 3-5`, `paper.pdf "2 Method"`, `paper.pdf all`).
---

# read: 영어 STEM 문서 → 로컬 학습 페이지

읽는 사람은 영어가 부담스러운 한국 공대생이다. 결과물은 `garden/<slug>/index.html` 한 장. 학생은 이 페이지에서 원문을 읽고, 모르는 단어를 눌러 **이 문맥에서의 뜻**을 보고, 막히면 번역과 문장 구조를 켜고, 끝까지 읽으면 씨앗을 눌러 질문에 답하며 개념을 발아시킨다(씨앗→새싹→화분→나무).

## 절차

1. **텍스트 확보.** `<slug>`는 파일명에서 확장자를 뗀 소문자 kebab-case. `mkdir -p garden/<slug>` 후, PDF면 `pdftotext -layout "<pdf>" garden/<slug>/source.txt` (`\f`가 페이지 구분). `pdftotext`가 없거나 수식·표가 깨져 못 읽겠으면 Read 도구로 PDF를 직접 읽는다(20쪽씩). `.txt`/`.md`면 그대로 쓴다.
2. **처음이면 data.js 헤더 작성.** `garden/<slug>/data.js`가 없을 때만: 목차(제목·번호 헤딩)를 뽑아 아래 `window.DATA` 헤더를 쓴다. 지도의 `summary`는 영어 제목만 보고도 무슨 얘긴지 알게 한 줄.
3. **범위 정하기.** 인자에 범위가 있으면 그것, `all`이면 전부, 없으면 지도에서 아직 data.js에 없는 첫 섹션 하나. 한 섹션 = 원문 2~4쪽. `all`이면 섹션마다 3~4단계를 반복한다.
4. **섹션 데이터 append.** 아래 스키마대로 `DATA.sections.push({...});` 블록 하나를 `cat >>`로 data.js 끝에 붙인다. **유효한 JS**여야 한다(문자열 안 따옴표·줄바꿈 이스케이프, 후행 쉼표 금지).
5. **페이지 열기.** `cp "${CLAUDE_PLUGIN_ROOT}/template.html" garden/<slug>/index.html && open garden/<slug>/index.html`. 이미 열려 있었으면 새로고침하라고 한 줄만 말한다.
6. **완료 조건.** 그 섹션의 모든 문단이 `paragraphs`에 있고, 용어표가 아래 기준으로 빠짐없고, `breakdown`의 문장이 원문 문단 안에 **글자 그대로** 들어 있고, 질문이 3개 이상이다. 마지막으로 페이지 경로와 다음 명령(`/plant-your-stem:read <파일> <다음 섹션>`)을 한 줄씩 보여준다.

## data.js 스키마

```js
window.DATA = {
  title: "문서 제목", file: "paper.pdf",
  map: [{ title: "1 Introduction", pages: "1-2", level: 1, summary: "한 줄" }],   // level 1~3
  sections: []
};
DATA.sections.push({
  title: "2 Method", pages: "3-5", level: 2,
  summary: "이 섹션이 무엇을 하고 왜 필요한지, 앞 섹션과의 연결. 1~2문장.",
  prereq: [{ term: "기울기(gradient)", note: "모르면: 함수값이 가장 빨리 커지는 방향의 화살표" }],
  glossary: [{ en: "learning rate", ko: "학습률", meaning: "이 문서에서: 한 번의 갱신에서 얼마나 크게 움직일지 정하는 양수 상수 η" }],
  paragraphs: [{ en: "원문 문단 그대로", ko: "한국어 번역", note: "(선택) 보충 설명" }],
  breakdown: [{
    segments: [{ t: "Gradient descent", type: "S" }, { t: " ", type: null }, { t: "is", type: "V" }, { t: " ", type: null }, { t: "an iterative procedure", type: "O" }, { t: " that ...", type: "M" }, { t: ".", type: null }],
    skeleton: "경사하강법은 반복 절차다", note: "M(that절)은 procedure를 꾸밈"
  }],
  intuition: "비유나 구체적 예시 하나로 '왜 이렇게 하는가'.",
  misconception: "흔한 오해 하나와 바로잡기.",
  questions: [{ type: "설명형", q: "X를 후배에게 설명한다면?", points: ["핵심 포인트 1", "포인트 2", "포인트 3"], model: "모범 답 3~5문장" }]
});
```

## 내용 기준

- **glossary가 이 페이지의 핵심이다.** 전문용어뿐 아니라 공대 1학년이 사전을 찾을 법한 단어를 전부 넣는다: 학술 동사·접속어(admit, quantify, whereas, namely, thereby), 이 분야에서 뜻이 달라지는 흔한 단어(kernel, bias, moment, trivial), 2~3단어 구(design matrix, full column rank). `meaning`은 사전 뜻이 아니라 **이 문장에서 무슨 뜻인지**. 단어는 원문 표기 그대로(영국식 철자 포함), 구는 소문자.
- **paragraphs.en은 원문 그대로**(오타도 유지). `ko`는 문장 단위로 대응시키고, 전문용어는 `한국어(English)` 병기. 수식은 그대로 두고 기호 하나하나를 말로 푼다("여기서 θ는 ~"). 원문이 말하지 않은 것을 보탤 땐 `note`에 넣는다.
- **breakdown**은 섹션에서 가장 꼬인 문장 1~3개. `segments`의 `t`를 모두 이어 붙이면 어떤 문단의 `en` 부분 문자열과 정확히 같아야 한다(공백·구두점 포함). type은 S/V/O/M 또는 null.
- **questions**는 설명형·적용형·연결형을 섞어 3~5개. `points`는 답에 들어 있어야 하는 핵심 3~4개로, 학생이 자기 답과 대조해 체크할 수 있게 구체적으로.
- 톤은 해요체. 고등 수학·기초 프로그래밍은 설명하지 않는다.
