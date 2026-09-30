# How I work

Some of this is process, some of it is temperament. All of it is visible in the repositories, because I write it down where the next person (or the next AI session, or me in six months) will find it.

[← Portfolio home](../README.md)

---

## How I think

**Diagnosis before prescription.** Before proposing a fix, name exactly what is broken and why. Nearly every serious defect I have shipped lived somewhere other than its symptom: a parse error that was a token budget, a 404 that was a missing route registration, an empty audit log that was a wrong secret. The right question is never "what do I change?" It is "what exactly is wrong, and how do I know?"

**Inspect before explaining.** If code, logs or a live system can answer the question, read them first. A plausible explanation is not evidence. I label my own claims the way I make my systems label theirs: *observed*, *derived*, *inferred* or *unknown*, and I treat "unknown" as a respectable answer.

**Aesthetics and engineering are the same kind of decision.** The colour of a toast and the schema of an error log are both choices about how a system communicates. I do not split a product into "the real work" and "the polish".

**Change the minimum, and know what it touches.** One shared stylesheet edit can break a dozen pages. I contain the scope of every change, then verify the surfaces I did not mean to touch.

**Decide, then record why.** I prefer one recommendation with evidence over a survey of options. Once decided, the reasoning goes in a decision record *with the alternatives that lost*, so I never have to win the same argument twice.

**Stop when the evidence says stop.** I closed an expensive experiment once the data was in. Sunk cost does not get a vote. (See [ZION Music](../case-studies/zion-music-retired.md).)

---

## How I work

| Habit | What it looks like in practice |
|---|---|
| **Git is the only road to production** | Edit, commit, push, CI deploys. Never a manual deploy. Rollback is a revert. This is written into my user-level rules and applied to every project, after one manual deploy caused a real regression. |
| **Small, well-described commits** | Nothing counts as done while it exists only in a working tree. |
| **Confirm before anything outward-facing or irreversible** | Publishing, deleting, spending. Approval for one action does not extend to the next. |
| **Verify live, separate from "tests pass"** | Status pages have one column for "passing tests" and another for "observed working in production". I test production read-only when I can. |
| **Every bug gets a post-mortem and a regression test** | Root cause, blast radius, what changed. The blast-radius check matters: *did anyone actually get hurt?* |
| **Cap the downside before you start** | Hard spend limits, local-first experiments, a rollback that costs nothing. |
| **Documentation is a discipline, not a deliverable** | Every repo has an operating manual for AI collaborators, an architecture document, a changelog and, where it matters, decision records. Systems outlive the context in which they were built. |
| **One source of truth per fact** | Where two documents disagree, the repo says which one wins. Stale docs are bugs, and I file them as bugs. |
| **Own the correction** | When guidance (mine or an assistant's) turns out wrong, the correction is written into the record with what was verified, not quietly patched over. |

---

## Working with AI

I build with AI coding assistants every day, and I treat that as an engineering problem rather than a convenience.

- **A written protocol per repository** tells an assistant the scope boundary, what it may not touch, what to read first and how deploys work. A fresh session should be safe and useful within a minute.
- **Scope boundaries are explicit.** Each project names the siblings it must not inspect and the cloud resources it may not touch.
- **I direct; the assistant executes.** The valuable part is the brief and the judgement applied to the result, which is also what I teach my students: *the direction is the work.*
- **Disclosure is part of the product.** My public site states exactly where AI tools were and were not used in my music, writing, teaching and code.
- **Authority is clear.** I can override any written rule by explicit instruction, and then the documents are updated to match, not argued with.

---

## How I like to have results

This is also how I deliver them.

1. **Lead with the answer, then the evidence.** A recommendation, what it rests on, and what would change my mind.
2. **Say plainly what is done, what is not, and what was not verified.** "Deployed and live-verified" and "committed but untested" are different sentences.
3. **Numbers with their assumptions.** Costs, latencies and failure rates come with how they were measured.
4. **Honest about uncertainty, without hedging everything.** Flag what matters; do not bury the answer in caveats.
5. **No flattery, no manufactured agreement.** If a claim outruns its evidence, say which part and why.
6. **Finished means committed, documented and reproducible.** Not "it worked on my machine".
7. **Show the trade-off you accepted.** Every design has a cost. Name it.

---

## What I care about beyond the code

I came to engineering from music, theology and teaching, and it shows. I want systems that are **honest** (they do not claim more than they know), **legible** (someone else can operate them), and **humane** (they respect the person on the other end, including their data). The tools I build are for problems I understood from the inside before I wrote a line of code.

[← Portfolio home](../README.md) · [Engineering decisions →](engineering-decisions.md)
