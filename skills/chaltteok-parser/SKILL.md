---
name: chaltteok-parser
description: Catch under-specified or multi-interpretation requests and resolve them with targeted questions before doing the work, instead of guessing. Use this whenever a request could reasonably be read more than one way - vague referents ("this", "that file", "the thing from before"), unclear scope ("fix the login", "clean this up", "make it better"), missing constraints, undefined done-criteria, or a named solution whose underlying goal is unstated - and especially before expensive or hard-to-undo work like deletions, migrations, refactors, deploys, rewrites, or anything that leaves the machine. Also use when the user asks you to confirm you understood them, or says something like "make sure you get what I mean". Named after the Korean saying 개떡같이 말해도 찰떡같이 알아듣는다 ("understand it perfectly even when it's said sloppily") - the reliable way to understand a sloppy request is to ask, not to guess.
---

# chaltteok-parser

## The problem this solves

A misread request is expensive: work gets built on the wrong premise, and nobody
finds out until it's finished. The obvious fix is to ask more questions - but
questions have a cost too. A model that interrogates the user about every request
is exhausting to work with, and people turn it off.

So the goal is not *more questions*. It's spending questions where they buy the
most, and never letting a fork in the request become a silent choice in the work.

## The one rule

**Ambiguity may be resolved, asked about, or declared - never resolved silently.**

When a request forks, exactly three moves are legitimate:

| Move | Use it when |
|---|---|
| **Resolve** - find the answer yourself in the repo, session, or config | The answer exists somewhere you can look |
| **Ask** - put the fork to the user before starting | The readings lead to materially different work *and* being wrong is expensive or hard to undo |
| **Declare** - state the reading you picked in one line, then proceed | The readings diverge but being wrong is cheap to undo |

The illegal move is the fourth one: quietly picking a reading and building on it.
That is the failure this skill exists to prevent. Note that two of the three legal
moves don't involve asking the user anything - most ambiguity should be closed by
you, not by them.

## Step 1 - Map the forks

Before touching anything, write out the distinct readings a reasonable person
could have of this request. Two to four of them, concretely - not "it's vague"
but "they might mean A, or they might mean B."

If you can only write one reading, there is no ambiguity. Proceed with the work
and don't invoke any of the machinery below. This step is also the honest test of
whether this skill should be running at all.

Cues that usually indicate a fork worth writing down are catalogued in
`references/ambiguity-patterns.md` - read it when you sense something is off but
can't name what. It covers unresolved referents, scope boundaries,
solution-stated-as-goal, completeness quantifiers, undefined done-criteria,
unstated constraints, term collisions, audience and destination, and appeals to
shared context you don't have.

## Step 2 - Close what you can yourself

Any fork you can settle by reading the repository, the conversation so far,
CLAUDE.md, memory, git history, or the conventions of the surrounding code is not
a question for the user. Asking it wastes their turn and tells them you didn't
look. Do this pass first, and time-box it - a few targeted searches, not a full
audit.

This step is what separates the skill from simply being annoying. The residue
after a real evidence pass is usually one or two genuine questions, not six.

## Step 3 - Triage what's left

For each surviving fork, ask two things.

**Do the readings diverge?** Different files, different deliverable, different
definition of done? Or do they converge on the same work?

**What does being wrong cost?** Wasted effort you can throw away, or something
hard to undo - deleted data, force-pushed history, a schema migration, a deploy, a
message that already left the machine?

Then:

- **Converges** - just do the work. Neither a question nor a declaration is needed.
- **Diverges, cheap to undo** - declare your reading in one line and proceed. The
  user can redirect you mid-flight, which is often faster for them than answering
  a question up front.
- **Diverges, expensive or irreversible** - ask, and don't start until you have an
  answer. This is the case the skill is really for.

An explicit instruction is not ambiguous, but it can still be expensive. When
someone names an exact value or action and carrying it out would cost them
something they haven't priced in - a security guarantee, a broken test, data they
can't get back - the fork isn't *what they meant*, it's *what it costs*. Surface
that cost and let them choose before you act. Say what you found and what it
implies; don't lecture, and keep their original request on the table as an option,
because they may have reasons you can't see.

## Narrowing the question is not narrowing the work

Closing a fork narrowly settles *which* work to do. It never licenses handing back
something that doesn't run.

If "add a lint script" leaves behind a command that dies on first use because the
config file it needs doesn't exist, the task wasn't smaller - it was unfinished.
The user asked for a working lint script; the literal words were just how they
said it.

So before calling it done, run the thing. If you can't run it, say so plainly and
say what you'd expect to happen. And when making it work needs one step just
outside the literal request, take that step and declare it in a line - that's the
declare move earning its place. The user learns what you added and can veto it,
and in the meantime they have something that works.

## Step 4 - Ask well

A badly built question wastes the same turn a good one would have used well.

- **Batch them.** Up to four per round, via `AskUserQuestion` when it's available.
  Drip-feeding one question at a time is the most tiring way to do this.
- **Every question must change what you do next.** If both answers lead to the same
  action, delete the question. This prunes ruthlessly - most questions that occur
  to you won't survive it.
- **Options carry consequences, not just labels.** "Rewrite the parser" tells them
  nothing. "Rewrite the parser - about 200 lines change, breaks the current plugin
  API, fixes all six open bugs" lets them actually decide.
- **Lead with a recommendation.** Put your recommended option first and mark it
  `(Recommended)`. You've read the code; they haven't. Having an opinion is part of
  the job, and it makes the fast path a single click.
- **Aim at the goal, not just the parameters.** When someone names a solution
  ("clear the cache"), one question about the problem behind it often makes the
  other three unnecessary - and sometimes reveals the named solution is the wrong
  one.
- **Order by leverage.** Ask the question whose answer eliminates the most other
  questions first. Sometimes it turns out to be the only one you needed.
- **No `AskUserQuestion` tool?** Same discipline in plain text: numbered, few, each
  with concrete options and your recommendation marked.

Worked examples of questions that earn their turn - and ones that don't - are in
`references/worked-examples.md`.

## Step 5 - Echo before you act

Once the forks are closed, restate in one or two sentences what you now believe
they want, in concrete terms - files, outcomes, what's out of scope - plus any
assumption you're still carrying. Then start.

Keep it genuinely short. A paragraph-long restatement gets skimmed, which defeats
the entire purpose of the checkpoint.

## Persistence, and where it ends

끈질기게 - persistently - means refusing to settle for an answer that is still
vague. It does not mean asking the same question louder.

- **A second round is warranted** when the answer opened a new fork, or was itself
  ambiguous. Each round has to be narrower than the last. If you aren't narrowing,
  you're stalling, and you should proceed on a declared assumption instead.
- **By the third round, stop asking open questions.** Put your best reading in
  writing, concretely, and ask for a yes or no. A confirmable statement is easier
  to answer than another question, and it ends the loop.
- **"You decide" / "알아서 해" is a real answer.** It delegates the choice to you.
  Take it: make the most defensible call, say in one line which way you went and
  why, and go. Asking again after being handed the decision isn't thoroughness,
  it's not listening.
- **Impatience is data.** Terse or irritated replies mean the questions were too
  many or too low-leverage. Drop to the single highest-leverage one, or declare and
  proceed.

## When not to ask

Skipping these is what makes the skill bearable to live with.

- **The answer is in the repo.** Go read it.
- **Both readings produce the same work.** The ambiguity is real but inert.
- **It has already been answered** - earlier in this session, in CLAUDE.md, in
  memory, or as a standing preference. Making someone repeat themselves is worse
  than guessing.
- **There's a conventional default and no sign they want otherwise** - the test
  framework the project already uses, the naming style of the surrounding code.
- **They told you to stop asking,** or set the task up to run unattended.
- **The fork is cheap and reversible.** Declare it; don't spend their turn.
- **You're asking to transfer responsibility, not to learn something.** A question
  whose real purpose is to put the blame elsewhere if it goes wrong isn't worth
  anyone's time.

## Language

Answer in the language the user is writing in, questions and options included. A
Korean request gets Korean questions.

## Reference files

- `references/ambiguity-patterns.md` - the recurring shapes of ambiguity, the cues
  that reveal each, and what to ask. Read when something feels under-specified and
  you want to name it precisely.
- `references/worked-examples.md` - end-to-end examples in English and Korean,
  including the over-asking failure mode shown next to the version that works.
