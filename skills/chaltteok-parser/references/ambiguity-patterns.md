# Ambiguity patterns

The recurring shapes that under-specified requests take. Use this to name what's
bothering you about a request, then turn that name into a specific question -
or, more often, into a specific thing to go look up.

Each pattern lists the cue that reveals it, why a silent guess hurts, and the
shape of the question that closes it. Not every instance needs a question:
run it through Step 2 (close it yourself) and Step 3 (triage) first.

1. [Unresolved referent](#1-unresolved-referent)
2. [Scope boundary](#2-scope-boundary)
3. [Solution stated as the goal](#3-solution-stated-as-the-goal)
4. [Completeness quantifier](#4-completeness-quantifier)
5. [Undefined done-criteria](#5-undefined-done-criteria)
6. [Comparative without a baseline](#6-comparative-without-a-baseline)
7. [Unstated constraints](#7-unstated-constraints)
8. [Term collision](#8-term-collision)
9. [Audience and destination](#9-audience-and-destination)
10. [Appeal to context you don't have](#10-appeal-to-context-you-dont-have)
11. [Instructions that conflict](#11-instructions-that-conflict)

---

## 1. Unresolved referent

**Cue.** "this", "that", "it", "the file", "저거", "그 파일" - a pointer with more
than one thing it could point at, or nothing visible at all.

**Why guessing hurts.** The cost isn't evenly spread. Picking the wrong file to
read wastes a turn; picking the wrong file to overwrite destroys work.

**Close it yourself first.** Recently edited files, the file open in the editor,
the one discussed a few messages ago, the only file matching the description.
Often exactly one candidate survives - then say which one you picked rather than
asking.

**Ask when** two or more candidates survive and the action is destructive, or the
candidates live in different parts of the system. Offer the candidates as
options with enough detail to tell them apart (path, last modified, what's in
it) - not just filenames, which often look identical.

---

## 2. Scope boundary

**Cue.** A request naming a target but not an extent: "fix the login", "clean up
the parser", "add tests", "정리 좀 해줘".

**Why guessing hurts.** This is the most common source of wasted work. You do the
thorough version when they wanted the two-line fix, or the two-line fix when
they wanted the module rewritten. Both feel like failure to the user.

**Close it yourself first.** If there's a failing test, an open issue, or an
error in the transcript, that's the scope. A request that follows a specific
complaint inherits that complaint's boundaries.

**Ask when** the plausible extents differ by an order of magnitude in effort or
blast radius. Frame the options by size and consequence - "just the crash on
empty input", "the crash plus the three adjacent validation bugs", "restructure
the module" - so the answer is a budget decision they can actually make.

---

## 3. Solution stated as the goal

**Cue.** The request names a mechanism rather than an outcome: "clear the cache",
"add a retry", "switch to Postgres", "싱글톤으로 바꿔줘".

**Why guessing hurts.** You can implement exactly what was asked and still not
help, because the named solution addresses a symptom the user diagnosed in a
hurry. This is also the pattern where a single question has the highest
leverage in the whole catalogue.

**Close it yourself first.** Sometimes the reason is obvious from the code, and
the named solution is plainly right. Then just do it - interrogating someone
about a correct instruction is its own failure.

**Ask when** the named solution looks like it will not fix the underlying
problem, or will fix it at a cost the user may not have priced in. One question
about the problem - "what's going wrong that made you want this?" - usually
dissolves the other questions. Ask it without implying they were wrong; they may
well have context you don't.

---

## 4. Completeness quantifier

**Cue.** "all", "everything", "the rest", "the old ones", "전부", "나머지".

**Why guessing hurts.** The set boundary is doing all the work in the sentence
and it was never defined. "Delete all the old branches" is unbounded in exactly
the direction that hurts.

**Close it yourself first.** Enumerate the set. Often the act of listing it
resolves the question, and the list itself is the best thing to show the user.

**Ask when** the set is large, or the operation on it is destructive. The most
useful move here is usually not a question but a list: show the concrete members
you'd act on and ask for a yes, or for exclusions. People are much better at
editing a list than at specifying one.

---

## 5. Undefined done-criteria

**Cue.** No stated bar for what finished looks like: no tests mentioned, no
acceptance condition, no "it should do X when Y".

**Why guessing hurts.** You and the user each fill in a bar silently, and you
find out they differed at review time. A quick prototype and a production change
are different jobs with the same description.

**Close it yourself first.** The project usually declares its bar - a test suite
that's expected to pass, a CI config, a linter, a CLAUDE.md. Follow it.

**Ask when** nothing in the project settles it and the gap between bars is large -
specifically when a throwaway version would be acceptable and would take a tenth
of the effort. Frame it as speed versus durability, since that's the real trade.

---

## 6. Comparative without a baseline

**Cue.** "faster", "cleaner", "better", "more robust", "더 낫게" - with no number,
no current measurement, and no threshold.

**Why guessing hurts.** Without a target you can't tell whether you're done, and
optimization without a measurement tends to land on whatever was easiest to
change rather than whatever was slow.

**Close it yourself first.** Measure. A profile, a benchmark, a timing run: the
current number is usually cheap to get and immediately reframes the conversation.

**Ask when** you have the current number and need the target, or when "better"
could mean several incompatible things (faster vs. more readable vs. less
memory) and improving one will cost another. Bring the measurement to the
question - "it's 4.2s now, mostly in the JSON decode; what would count as fast
enough?" is a far better question than "how fast do you want it?"

---

## 7. Unstated constraints

**Cue.** Nothing in the request mentions the boundaries that obviously exist:
backwards compatibility, a dependency they won't accept, a deadline, a platform,
an API contract other people depend on.

**Why guessing hurts.** Constraint violations are usually discovered late and
invalidate everything built on top of them.

**Close it yourself first.** Most constraints are written down somewhere -
package manifests, lockfiles, CI targets, a CONTRIBUTING file, the versions the
existing code already supports. Read them instead of asking.

**Ask when** the work would add a dependency, change a public interface, or drop
support for something currently supported. Those three are worth a question
almost every time, because they're expensive to reverse and invisible in a diff
until someone downstream breaks.

---

## 8. Term collision

**Cue.** A word that means different things in different corners of this project:
"user" as a database table and as a domain object, "client" as a customer and as
an HTTP client, "session" as a login session and as a recorded run.

**Why guessing hurts.** You confidently do the right thing to the wrong subject,
and everything about the work looks correct until someone reads it closely.

**Close it yourself first.** Grep the term. The surrounding context usually
disambiguates within a couple of files, and the project's own vocabulary is more
authoritative than your guess.

**Ask when** both senses are live in the area you're about to change. Show the two
things concretely - file and symbol - rather than describing them, because
descriptions of near-synonyms are hard to tell apart in prose.

---

## 9. Audience and destination

**Cue.** A deliverable with no stated reader or resting place: "write up the
migration", "make a summary", "문서 하나 만들어줘".

**Why guessing hurts.** Audience determines nearly everything about the artifact -
length, assumed knowledge, tone, format - so this is one of the few cases where
guessing wrong means starting over rather than adjusting.

**Close it yourself first.** The request often names the reader indirectly ("for
the team", "so I can send it to them", "for the PR"), and that's enough.

**Ask when** it's genuinely unstated and the candidates differ sharply - a note to
self versus something a customer will read. Ask about the reader and the
destination together; they usually travel as a pair, and the destination implies
the format.

---

## 10. Appeal to context you don't have

**Cue.** "like last time", "the usual way", "same as the other one", "저번처럼".

**Why guessing hurts.** The user believes the specification is complete, because
for them it is. They won't discover the mismatch until they see the output.

**Close it yourself first.** Look for the precedent - git history, an existing
sibling implementation, an earlier file of the same kind. Finding it is usually
easy and completely resolves the request.

**Ask when** you can't find the precedent, or you find several that differ. Say
which one you found and ask whether that's the one they mean - a named guess is
much easier to confirm or correct than an open question, and it shows you looked.

---

## 11. Instructions that conflict

**Cue.** Two requirements that can't both hold: "don't change the public API" plus
"rename this exported function"; "keep it minimal" plus a list of twelve
features; a request that contradicts something in CLAUDE.md or an earlier turn.

**Why guessing hurts.** Silently honouring one and dropping the other is the worst
outcome, because the user doesn't learn that the conflict existed, and the
dropped requirement may have been the important one.

**Close it yourself first.** Check whether the conflict is real or only apparent -
there's often a third path that satisfies both, such as a deprecation shim, and
finding it is better than making anyone choose.

**Ask when** the conflict is real. Name both sides explicitly and ask which gives
way. This is the one pattern where surfacing the problem matters more than
resolving it quickly: the user usually didn't know the two things collided, and
that fact is itself the most useful thing you can tell them.
