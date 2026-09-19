# chaltteok-parser

A Claude Code skill that refuses to guess what you meant.

> **개떡같이 말해도 찰떡같이 알아듣는다**
> *"Even when it's said sloppily, understand it perfectly."*

The saying is usually read as a compliment about intuition. This skill takes the
unromantic view: the reliable way to understand a sloppy request is to **ask**,
not to guess well.

[한국어 설명은 아래에](#한국어)

---

## What it does

When a request could be read more than one way, Claude usually picks the reading
that seems most likely and gets on with it. When that guess is right, it saves
you a turn. When it's wrong, you find out after the work is finished.

This skill makes one thing illegal: **silently choosing a reading.** Claude can
look up the answer, it can ask you, or it can tell you which reading it picked —
but it can't quietly decide and build on it.

| Move | When |
|---|---|
| **Resolve** — find the answer in the repo, the session, the config | The answer already exists somewhere |
| **Ask** — put the fork to you before starting | The readings lead to different work *and* being wrong is expensive |
| **Declare** — state the reading it picked, in one line, then proceed | The readings differ but a wrong guess is cheap to undo |

Two of those three don't cost you a turn, which is the point. The skill is built
to close most ambiguity by itself and spend your attention only where it matters.

## Example

> **You:** 세션 토큰 만료를 30일로 늘려줘
> *(extend the session token expiry to 30 days)*

Clear enough — there's a number and a target. But reading the codebase turns up
something you didn't ask about: `logout()` writes tokens to a `revoked_tokens`
table that `verifyToken()` never reads. Logout doesn't actually revoke anything.
At a 2-hour expiry that's a two-hour window. At 30 days, a leaked token is alive
for a month with no way to kill it.

Instead of doing it, or refusing, the skill puts the cost in front of you:

> 로그아웃이 지금 실제로는 토큰을 무효화하지 못합니다 (`verifyToken`이
> `revoked_tokens`를 조회하지 않음). 2시간이라 지금은 큰 문제가 아닌데, 30일로
> 늘리면 유출된 토큰을 한 달간 못 막습니다.
>
> - **그대로 30일로 (추천)** — 위험을 감수. 1줄 + 테스트 1개 수정.
> - **리프레시 토큰** — 액세스 토큰은 2시간, 갱신용 토큰 30일. 더 안전하지만
>   모바일 클라이언트 배포 순서 조율 필요.
> - **무효화부터 수정** — `verifyToken`에 폐기 목록 검사 추가.

Your original request stays on the table as the recommended option. What changed
is that you're choosing with the trade-off visible.

## When it stays quiet

Over-asking is how a skill like this fails, so roughly half of it is about *not*
asking. It stays out of the way when:

- the answer is in the repo — it goes and reads it
- both readings lead to the same work
- you already answered, here or in `CLAUDE.md`
- there's a conventional default and no sign you want otherwise
- a wrong guess is cheap to undo — it declares and proceeds
- you said stop asking, or handed it the decision ("알아서 해" is a real answer)

It also won't narrow a task into something that doesn't run. "Add a lint script"
means a lint script that works, not one line that errors on first use.

## Install

**As a plugin (recommended):**

```
/plugin marketplace add lefthanddeveloper/chaltteok-parser
/plugin install chaltteok-parser@chaltteok
```

**As a plain skill:**

```bash
git clone https://github.com/lefthanddeveloper/chaltteok-parser.git /tmp/chaltteok
cp -r /tmp/chaltteok/skills/chaltteok-parser ~/.claude/skills/
```

Use `.claude/skills/` inside a project instead of `~/.claude/skills/` to scope it
to that project.

The skill triggers on its own when a request looks ambiguous. There's no command
to remember.

## What's inside

```
skills/chaltteok-parser/
├── SKILL.md                          the core loop
└── references/
    ├── ambiguity-patterns.md         11 recurring shapes of ambiguity
    └── worked-examples.md            7 examples, each with the failure beside it
```

`SKILL.md` is what loads when the skill fires. The reference files load only when
the skill decides it needs them.

## Feedback

If you hit a case where it asks too much, or too little, that's the interesting
bug. Please open an issue with the prompt.

## License

MIT. See [LICENSE](LICENSE).

---

## 한국어

**요청이 애매할 때 혼자 넘겨짚지 않는 Claude Code 스킬입니다.**

"개떡같이 말해도 찰떡같이 알아듣는다"는 보통 눈치 빠름을 칭찬하는 말로 쓰이지만,
이 스킬은 좀 더 건조하게 봅니다 — 개떡같은 말을 제대로 알아듣는 확실한 방법은
**잘 찍는 게 아니라 묻는 것**이라고요.

### 무엇을 하나요

요청이 두 가지로 읽힐 때, Claude는 보통 더 그럴듯한 쪽을 골라 진행합니다. 맞으면
시간이 절약되지만, 틀리면 **일이 다 끝난 뒤에** 알게 됩니다.

이 스킬은 딱 하나를 금지합니다: **말없이 하나를 골라서 진행하기.** 직접 찾아보거나,
물어보거나, "이렇게 이해하고 갑니다"라고 밝히거나 — 셋 중 하나는 해야 합니다.

| 수 | 언제 |
|---|---|
| **직접 해소** — 코드·대화·설정에서 답을 찾음 | 답이 이미 어딘가에 있을 때 |
| **질문** — 시작 전에 갈림길을 사용자에게 | 해석에 따라 할 일이 달라지고, **틀리면 비쌀 때** |
| **선언** — 고른 해석을 한 줄로 밝히고 진행 | 해석은 갈리지만 되돌리기 쉬울 때 |

셋 중 둘은 사용자의 시간을 쓰지 않습니다. 대부분의 애매함은 스킬이 알아서 닫고,
정말 중요한 곳에만 질문을 씁니다.

### 언제 조용히 있나요

이런 스킬이 망하는 방식은 **질문 과잉**입니다. 그래서 내용의 절반은 "묻지 말아야
할 때"에 대한 것입니다.

- 답이 코드에 있으면 → 가서 읽습니다
- 어느 쪽으로 읽어도 할 일이 같으면 → 그냥 합니다
- 이미 답한 내용이면 (대화나 `CLAUDE.md`에) → 다시 묻지 않습니다
- 관례적인 기본값이 있으면 → 그걸 씁니다
- 틀려도 쉽게 되돌릴 수 있으면 → 밝히고 진행합니다
- "그만 물어봐" 또는 "알아서 해"는 **답변으로 취급**합니다

범위를 좁힌다고 **작동 안 하는 걸 내놓지도 않습니다.** "lint 스크립트 추가해줘"의
뜻은 *돌아가는* lint 스크립트지, 실행하면 에러 나는 한 줄이 아닙니다.

### 설치

**플러그인으로 (권장):**

```
/plugin marketplace add lefthanddeveloper/chaltteok-parser
/plugin install chaltteok-parser@chaltteok
```

**스킬 파일만 직접:**

```bash
git clone https://github.com/lefthanddeveloper/chaltteok-parser.git /tmp/chaltteok
cp -r /tmp/chaltteok/skills/chaltteok-parser ~/.claude/skills/
```

특정 프로젝트에서만 쓰려면 `~/.claude/skills/` 대신 그 프로젝트의
`.claude/skills/`에 넣으세요.

설치 후엔 따로 부를 필요 없이, 요청이 애매해 보이면 알아서 작동합니다.

### 제보

너무 많이 묻거나 너무 안 묻는 경우를 만나시면, 그게 가장 쓸모 있는 제보입니다.
해당 프롬프트와 함께 이슈로 남겨주세요.

### 라이선스

MIT. [LICENSE](LICENSE) 참고.
