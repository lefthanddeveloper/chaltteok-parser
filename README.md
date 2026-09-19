<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/Claude_Code-Skill-purple.svg" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/README-EN%20%7C%20KO%20%7C%20JA-lightgrey" alt="i18n">
</p>

# chaltteok-parser

**English** | [한국어](README_KO.md) | [日本語](README_JA.md)

A Claude Code skill that refuses to guess what you meant.

> **개떡같이 말해도 찰떡같이 알아듣는다**
> *"Even when it's said sloppily, understand it perfectly."* — a Korean saying

The saying is usually read as a compliment about intuition. This skill takes the
unromantic view: the reliable way to understand a sloppy request is to **ask**,
not to guess well.

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

> **You:** Extend the session token expiry to 30 days.

Clear enough — there's a number and a target. But reading the codebase turns up
something you didn't ask about: `logout()` writes tokens to a `revoked_tokens`
table that `verifyToken()` never reads. Logout doesn't actually revoke anything.
At a two-hour expiry that's a two-hour window. At 30 days, a leaked token stays
alive for a month with no way to kill it.

Instead of quietly doing it, or refusing, the skill puts the cost in front of you:

> Logout doesn't currently revoke anything — `verifyToken()` never consults the
> `revoked_tokens` table. At two hours that's a small window; at 30 days a leaked
> token stays valid for a month and can't be killed.
>
> - **Go to 30 days anyway (recommended)** — accept the risk. One line, plus one
>   test to update.
> - **Refresh tokens** — two-hour access token, 30-day refresh token. Safer, but
>   the mobile client pins the auth API, so it needs a coordinated release.
> - **Fix revocation first** — have `verifyToken()` check the revocation list.

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
- you said stop asking, or handed it the decision ("you decide" is a real answer)

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
