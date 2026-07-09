### 보리 체르니의 CALUDE.md 가이드

### 보리스 체르니(Boris Cherny)가 누군가요?

- **Claude Code의 창시자** — Anthropic에서 Claude Code 프로젝트를 처음 시작하고 이끌어 온 사람이에요. 우리가 매일 쓰는 `claude` CLI, Plan 모드, 서브에이전트, `CLAUDE.md` 같은 개념이 다 이분의 설계에서 나왔습니다
- 한 마디로, **"Claude Code를 가장 잘 아는 사람"** 이 본인 작업 환경에 직접 박아두고 쓰는 룰이라고 보시면 돼요

### 어디에 붙여넣나요?

Claude Code는 작업을 시작할 때 두 위치의 `CLAUDE.md`를 자동으로 읽습니다. 본인 상황에 맞춰 골라서 붙여넣으시면 돼요.

|위치|경로|언제 쓰나요?|
|---|---|---|
|**글로벌**|`~/.claude/CLAUDE.md`|모든 프로젝트에서 같은 룰을 쓰고 싶을 때|
|**프로젝트별**|작업 폴더의 `CLAUDE.md`|이 프로젝트에서만 적용하고 싶을 때|

붙여넣는 방법은 둘 중 하나입니다

1. 클로드 코드 안에서 `이 내용을 ~/.claude/CLAUDE.md 에 추가해줘` 라고 말하고 아래 원문을 붙여넣기
2. 직접 에디터로 열어서 복사-붙여넣기

전체 다 쓸 필요 없습니다. 마음에 드는 섹션만 골라 쓰셔도 충분히 효과 있어요.

---

## 원문 (영어 전체)

```jsx
## Workflow Orchestration

### 1. Plan Node Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One tack per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimat Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
```

---

## 분할 해석 — Workflow Orchestration

### 1. Plan Node Default

**원문**

```jsx
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity
```

**한국어 해설**

- 단계가 3개 이상이거나 구조에 영향 주는 작업이면 **무조건 Plan 모드 먼저** 들어가기
- 작업 도중에 일이 꼬이면 "일단 계속 가보자"가 아니라 **즉시 멈추고 다시 계획**
- 만들 때뿐 아니라 **검증할 때도** Plan 모드 활용 ("이걸 어떻게 확인할지" 부터 짜라는 뜻)
- 시작 전에 명세를 자세히 적어둘수록 헛다리 짚는 횟수가 줄어듭니다

### 2. Subagent Strategy

**원문**

```jsx
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One tack per subagent for focused execution
```

**한국어 해설**

- **서브에이전트를 아끼지 말고 적극 사용**해서 메인 대화창을 깔끔하게 유지
- 코드베이스 탐색, 자료 조사, 병렬 분석 같은 "맥락만 잡아먹는 일"은 서브에이전트한테 떠넘기기
- 어려운 문제일수록 서브에이전트 여러 개를 풀어서 "컴퓨팅 파워로 밀어붙이기"
- 한 서브에이전트는 **한 가지 일에만 집중**하게 (잡일 여러 개 동시에 시키지 않기)

### 3. Self-Improvement Loop

**원문**

```jsx
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project
```

**한국어 해설**

- 사용자한테 지적받으면 **무조건** `**tasks/lessons.md**` **파일에 "이런 실수를 했고, 이렇게 막을 거다"** 라고 기록
- 자기 자신에게 적용할 룰로 바꿔서 적기 (다음번에 같은 실수 못 하게)
- 실수율이 떨어질 때까지 **이 lessons 파일을 계속 다듬기** — 한 번 적고 끝이 아님
- 새 세션 시작할 때 관련된 lessons 먼저 훑어보고 작업 들어가기

> 💡 이게 보리스 체르니의 룰 중에서도 가장 핵심이에요. 클로드한테 "앞으로 같은 실수 하지 마" 라고 백 번 말하는 것보다, 파일에 적어두고 매번 읽게 하는 게 훨씬 효과적입니다.

### 4. Verification Before Done

**원문**

```jsx
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness
```

**한국어 해설**

- **돌아가는 걸 증명하기 전까지는 "완료" 라고 하지 마라**
- 필요하면 main 브랜치랑 내 변경사항을 비교해서 동작 차이를 확인
- 스스로한테 물어보기: **"시니어 엔지니어가 이걸 보면 통과시킬까?"**
- 테스트 돌리기, 로그 확인하기, "이게 맞다"는 걸 직접 보여주기

### 5. Demand Elegance (Balanced)

**원문**

```jsx
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it
```

**한국어 해설**

- 사소하지 않은 변경이라면 잠깐 멈추고 **"더 우아한 방법은 없을까?"** 자문
- 수정이 임시방편 같다는 느낌이 들면 → "지금 알게 된 걸 다 알고 있다고 치고, 우아한 해법을 다시 짜라"
- 단순한 수정에는 적용하지 말기 (오버엔지니어링 방지)
- 사용자에게 보여주기 전에 **본인 작업을 스스로 한 번 더 의심**하기

### 6. Autonomous Bug Fixing

**원문**

```jsx
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how
```

**한국어 해설**

- 버그 리포트 받으면 **그냥 고쳐라.** 일일이 "이거 어떻게 해요?" 묻지 말고
- 로그, 에러, 실패한 테스트를 보고 → 직접 해결
- 사용자가 맥락 전환하느라 시간 쓰게 만들지 말기
- 실패한 CI 테스트도 "어떻게 고쳐야 하나요?" 물어보지 말고 그냥 가서 고치기

---

## 분할 해석 — Task Management (할 일 관리)

**원문**

```jsx
1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections
```

**한국어 해설**

|단계|원문|한국어 풀이|
|---|---|---|
|1|Plan First|`tasks/todo.md` 파일에 **체크박스 형태**로 계획 먼저 적기|
|2|Verify Plan|구현 들어가기 전에 사용자한테 "이 계획 맞나요?" 한 번 확인받기|
|3|Track Progress|진행하면서 **항목 하나씩 체크 표시**|
|4|Explain Changes|매 단계마다 "방금 뭘 바꿨는지" 짧게 요약|
|5|Document Results|`tasks/todo.md` 맨 아래에 **review 섹션 추가**해서 결과 정리|
|6|Capture Lessons|사용자 피드백 받으면 `**tasks/lessons.md**` 업데이트|

> 💡 핵심은 `**tasks/todo.md**` 와 `**tasks/lessons.md**` 두 파일을 프로젝트에 두고 클로드가 계속 갱신하게 만드는 거예요. "계획 → 실행 → 회고 → 학습" 루프가 자동으로 돌아갑니다.

---

## 분할 해석 — Core Principles (핵심 원칙)

**원문**

```jsx
- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimat Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
```

**한국어 해설**

- **Simplicity First (단순함 우선)** — 모든 변경을 가능한 한 단순하게. 건드리는 코드 양도 최소로
- **No Laziness (게으름 금지)** — **근본 원인을 찾아서** 해결. 임시방편 금지. 시니어 개발자 수준의 기준 적용
- **Minimal Impact (최소 영향)** — _원문엔 "Minimat" 오타가 있어요_ — 변경은 **꼭 필요한 곳만** 건드리기. 새 버그 들이지 않기

> 💡 이 세 줄이 사실상 위의 모든 룰을 한 줄씩 요약한 거예요. 다른 거 다 빼더라도 이 세 줄만 본인 `CLAUDE.md`에 넣어도 효과를 봅니다.

---

## 이런 분께 추천해요

- 클로드 코드를 한두 달 써봤는데 "왜 이렇게 자꾸 깔끔하게 못 시키지?" 싶으신 분
- 같은 잔소리를 매번 반복하기 지치신 분 ("임시로 때우지 마", "끝나면 검증해" 같은 거)
- 본인만의 `CLAUDE.md` 룰을 **처음부터 만들기 부담스러우신** 분 — 창시자가 직접 쓰는 룰을 시작점 삼아 본인 스타일로 다듬어 가시면 돼요
- 클로드한테 **시니어 개발자처럼** 일을 시키고 싶으신 분

## 관련 문서

- 상위 목차: [[04. Notion]]
- 관련: [[클로드코드 자동화 시스템 5 스킬]]
- 관련: [[LLM Wiki 3-Layer]]
