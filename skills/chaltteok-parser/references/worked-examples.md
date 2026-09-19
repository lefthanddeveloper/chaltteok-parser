# Worked examples

Seven situations, each showing the move that works next to the move that doesn't.
The failure cases matter as much as the successes: over-asking is the way this
skill most often goes wrong in practice, and under-delivering is the second way.

1. [When not to fire at all](#1-when-not-to-fire-at-all)
2. [Scope, after an evidence pass](#2-scope-after-an-evidence-pass)
3. [Destructive and unbounded](#3-destructive-and-unbounded)
4. [The named solution](#4-the-named-solution)
5. [Handing the decision back](#5-handing-the-decision-back)
6. [Ending a loop that isn't converging](#6-ending-a-loop-that-isnt-converging)
7. [Staying in scope without under-delivering](#7-staying-in-scope-without-under-delivering)

---

## 1. When not to fire at all

> **User:** README에 설치 방법 섹션 추가해줘

**Doesn't work:**

> Before I start, a few questions: Which section should it go in? What install
> methods should I document? What level of detail? Should I include
> troubleshooting?

Every one of these is answerable from the repository or has an obvious default.
The user now has four decisions to make for a task they expected to take one
line. This is what makes people disable a skill.

**Works:**

Read `package.json`, check whether the project publishes to npm, look at how
neighbouring sections are written, and write the section. Then:

> README에 Installation 섹션 추가했습니다. npm/pnpm/yarn 세 가지로 적었고,
> 기존 섹션들 톤에 맞췄습니다. Node 18+ 요구사항은 package.json의 engines에서
> 가져왔습니다.

Step 1 produced only one plausible reading, so the skill correctly stays out of
the way. The closing note is a *declaration*, not a question - it takes no turn
from the user but leaves every assumption visible and correctable.

---

## 2. Scope, after an evidence pass

> **User:** auth 모듈 좀 정리해줘

Six questions come to mind immediately: what counts as cleanup, which files,
should behaviour change, should tests be added, is the public API frozen, how
big a change is acceptable.

**Step 2 closes four of them.** `src/auth/` holds three files. Git history shows
two were touched last week; the third hasn't changed in a year. The linter
reports 14 warnings, all in one file. There's a test suite and it passes.
CLAUDE.md says to keep public exports stable. So: the target is
`src/auth/session.ts`, behaviour stays as it is, tests are expected to pass, the
API is frozen.

**Two survive, and they diverge sharply in effort:**

> **AskUserQuestion:**
>
> **범위** - auth 모듈 정리, 어디까지 할까요?
> - **린트 경고 + 죽은 코드만 (추천)** - session.ts의 경고 14개와 미사용 export 3개
>   제거. 동작 변화 없음, 약 40줄.
> - **구조 리팩터링까지** - session.ts가 토큰 검증과 쿠키 관리를 같이 하고 있어서
>   분리 가능. 약 300줄, 내부 호출부 6곳 수정 필요.
> - **의존성 정리 포함** - 위 항목 + `jsonwebtoken`을 `jose`로 교체. 동작은 같지만
>   테스트 전면 재확인 필요.

Note what the options carry: line counts, blast radius, whether behaviour
changes. That's what makes this a budget decision the user can make in three
seconds rather than a vague preference question.

---

## 3. Destructive and unbounded

> **User:** 안 쓰는 브랜치 다 지워줘

Both a completeness quantifier and an irreversible action - the combination that
most deserves a question.

**Doesn't work:**

> 안 쓰는 브랜치 기준이 어떻게 되나요?

It's a fair question, but it puts the work of specifying a set onto the user,
which is the hard direction. They'd have to imagine the branches to define the
rule.

**Works** - enumerate first, then ask for edits:

> 로컬 브랜치 12개 중 main에 머지됐고 30일 이상 방치된 게 7개입니다:
>
> ```
> feat/login-redirect    merged  74일 전
> fix/typo-readme        merged  61일 전
> ...
> ```
>
> 나머지 5개는 미머지라 제외했습니다. 이 7개 지울까요? 빼야 할 게 있으면 알려주세요.
> (원격 브랜치는 건드리지 않습니다 - 필요하면 따로 말씀해주세요.)

The user edits a concrete list instead of authoring a rule, the exclusions are
stated so silence means consent to a known set, and the remote-branch boundary
is declared rather than assumed. One turn, no ambiguity left.

---

## 4. The named solution

> **User:** 캐시 좀 꺼줘. 자꾸 옛날 데이터가 보여서.

The second sentence is the gift - it names the actual problem. The request is a
solution the user picked for it.

**Doesn't work:** disable the cache. It will fix the symptom and quietly cost
them the thing the cache was for.

**Also doesn't work:** "정말 끄시겠어요?" - a confirmation prompt that adds a turn
without adding information.

**Works** - one high-leverage question aimed at the problem:

> 캐시를 끄면 해결되긴 하는데, 목록 페이지 로딩이 200ms → 1.4s로 늘어납니다.
> 확인해보니 stale 데이터가 보이는 건 수정 후 무효화가 안 되기 때문이라, 그쪽을
> 고치면 캐시를 유지한 채로 해결됩니다.
>
> - **무효화 수정 (추천)** - `updatePost` 후 캐시 키 삭제. 3줄, 근본 원인.
> - **TTL 단축** - 5분 → 10초. 한 줄, 여전히 최대 10초 지연.
> - **캐시 끄기** - 요청하신 대로. 확실하지만 느려집니다.

The user's original request is still on the table as an option - they may have
reasons you don't know. What changed is that they're now choosing with the
trade-off visible.

---

## 5. Handing the decision back

> **You:** [three options for the auth cleanup]
> **User:** 아 몰라, 알아서 해

This is an answer, not an evasion. Continuing to ask is the failure here.

**Works:**

> 린트 경고 + 죽은 코드 제거로 갑니다. 동작이 안 바뀌고 되돌리기 쉬워서요.
> 구조 리팩터링은 손대지 않겠습니다.

Then do it. One line naming the choice and the reason, no further questions. If
it turns out wrong, it's a cheap revert - which is precisely why this was the
defensible pick.

---

## 6. Ending a loop that isn't converging

Two rounds in, the answers keep arriving vague. A third open question will not
help; the user has already told you as much as they can put into words.

**Doesn't work:**

> 그럼 정확히 어떤 동작을 원하시는 건가요?

Asking the same thing a third time, slightly rephrased. The user has no better
words available, which is why the first two attempts failed.

**Works** - switch from asking to proposing:

> 제가 이해한 걸로 정리하면 이렇습니다:
>
> - 업로드 실패 시 3회까지 자동 재시도, 지수 백오프
> - 3회 다 실패하면 큐에 남기고 UI에 "나중에 재시도" 표시
> - 파일 크기 제한은 지금처럼 50MB 유지
>
> 이대로 가면 될까요? 틀린 부분만 짚어주시면 됩니다.

A concrete proposal is far easier to correct than an open question is to answer -
people recognise a wrong detail instantly even when they can't produce the right
one from scratch. This is how the persistence loop terminates cleanly instead of
degrading into interrogation.

---

## 7. Staying in scope without under-delivering

> **User:** package.json에 lint 스크립트 추가해줘. eslint 쓰고 있어.

Unambiguous - example 1's lesson applies and no question is warranted. The trap
here is the opposite one: being so careful about scope that what you hand back
doesn't work.

**Doesn't work:**

Add `"lint": "eslint ."` to `scripts` and report it done. The repo has `eslint` in
devDependencies but no config file anywhere, and ESLint 9 requires a flat config,
so `npm run lint` fails the moment it's run. The literal request was satisfied and
the user got nothing usable.

**Works:**

Add the script, create the `eslint.config.js` it needs, run it once to confirm,
and declare the extra file:

> `"lint": "eslint ."` 추가했습니다. 설정 파일이 아예 없어서 `eslint.config.js`도
> 같이 만들었습니다 - ESLint 9은 설정이 없으면 실행 자체가 안 됩니다. 새 패키지는
> 안 깔고 규칙을 직접 적었습니다 (CLAUDE.md의 의존성 제한 때문에). 돌려보니 13개
> 파일 0 에러입니다. 설정 파일이 부담스러우면 지우셔도 됩니다.

The added file is named, the reason is given, and it's offered up for removal -
so scope discipline is preserved without shipping something broken. Note the
verification: "돌려보니 0 에러" is what separates a claim from a result.
