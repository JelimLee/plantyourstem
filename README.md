# 🌱 Plant your STEM

영어 STEM 문서(논문, 교재, 기술 문서 PDF)를 **브라우저에서 읽는 로컬 학습 페이지**로 바꾸는 Claude Code 플러그인. 단어를 누르면 이 문맥에서의 뜻, 문단마다 한국어 번역, 문장 구조 색칠, 다 읽으면 씨앗을 눌러 질문으로 개념을 **발아**시켜 나무로 키운다. API 키·서버 없음, 결과물은 현재 폴더의 `garden/`에 파일로 남는다.

## 설치

```bash
# 이 세션만
claude --plugin-dir <path-to>/plantyourstem

# 항상 (skills-dir 로 링크)
ln -s <path-to>/plantyourstem ~/.claude/skills/plant-your-stem
```

## 사용

```
/plant-your-stem:read paper.pdf              # 문서 지도 + 첫 섹션 → 브라우저에서 열림
/plant-your-stem:read paper.pdf "3 Results"  # 특정 섹션 추가
/plant-your-stem:read paper.pdf all          # 전체
/plant-your-stem:check                       # 터미널 구두시험 (또는 페이지에서 복사한 답안 붙여넣기)
```

`read` 는 `garden/<slug>/data.js`(번역·용어·문장분해·질문)를 쓰고 `template.html`을 `index.html`로 복사해 연다.
페이지에서: 단어 클릭 → 문맥 뜻 · 🇰🇷 번역 토글 · 🧩 S/V/O/M 문장 구조 토글 · 끝까지 읽으면 🌰 씨앗이 깨어남 → 질문에 답하고 핵심 포인트를 체크하면 🌱→🌿→🪴→🌳 로 발아. 진행은 브라우저 localStorage에 저장.

`check` 는 페이지의 "답안 복사"로 가져온 답을 Claude가 자세히 채점하거나, 터미널에서 질문을 하나씩 낸다. 기록은 `garden/<slug>/checks.md`.

## 참고 원본

`Think Out Loud: DevDoc Mastery` (React + Gemini 웹앱). 문장 색칠(S/V/O/M), 내 말로 설명 채점, 새싹 성장 게임화를 가져오고, 하드코딩 문서 22개 대신 임의의 PDF를 받도록 바꿨다.
