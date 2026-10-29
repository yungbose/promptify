---
name: promptify
description: Use when the user has a rough prompt, task idea, or half-formed goal they want turned into the best possible prompt or handover for a Claude 5 session, with Opus 5.5 as the default target and Fable 5.1 as the escalation. Triggers on "/promptify <prompt>", "/fable-promptify <prompt>" (legacy name), "promptify this", "make this a fable prompt", "make this an opus prompt", "optimise this prompt", or pasting a rough prompt and asking for it to be improved before starting a session. NOT for executing the task itself; the deliverable is the refined prompt.
---

# promptify

Renamed from `fable-promptify` on 19 Oct 2026 and rescoped from Fable 5 to the whole Claude 5 family. The old name survives as a thin alias.

## Overview

Claude 5 models run for hours, test their own work and correct themselves, so the leverage sits in the prompt. A prompt earns that autonomy when it has four things: a crystal **aim**, **verification** the model can run without you, **context** behind every constraint (not bare rules), and explicit **scope**.

**Target model.** Opus 5.5 is the default: it is the orchestrator and judge, and Anthropic puts it at roughly Fable 5.1's level on most work, faster and cheaper. Fable 5.1 is the escalation, used when Opus 5.5 at `xhigh` or `max` has demonstrably fallen short, or when the operator says "plan with fable" for open-ended cross-domain planning. Lanes a handover delegates (Sonnet 5.5, Haiku 5.5, a coding-agent lane such as Codex) follow your routing policy. In the author's setup that is `~/.claude/skills/_shared/ROUTING.md`; without one, the lane table in Phase 4 is the default.

This skill converts a rough prompt into that shape by interviewing the user until the aim is crystal, then drafting a structured prompt. It is the "ask Claude to interview me before writing the final spec" pattern, made repeatable.

**Core loop**: ingest → reflect (four pillars, then the depth pass) → interview until crystal → draft → deliver + offer to run.

The skill never starts executing the task mid-flow. The deliverable is the prompt.

## Phase 0: Preflight

1. Read `references/learnings.md` (lessons from prior runs) and `<project-config>/learnings.md` if present. Entries marked `active` are binding.
2. Read `references/prompting-guide.md`, the Claude 5 family checklist used in Phases 2 and 4. Do not draft from memory of older Fable or Claude 4 guidance; the guide is the source.
3. For a handover that will use more than one lane, read your routing policy if you keep one (the author's is `~/.claude/skills/_shared/ROUTING.md`: the model table, effort defaults, parallelism and the spill rule). Without one, use the lane table in Phase 4.

## Phase 1: Ingest

1. Capture the user's rough prompt verbatim. If none was given with the invocation, ask for it.
2. **Investigate before asking.** If the task touches a repo, skim CLAUDE.md, the relevant files, and recent commits first. A question the repo can answer is a wasted interview round.
3. Classify the task shape; it changes which practices matter (see the guide's per-shape notes):
   - **Quick task** (bounded edit or fix): lean prompt, skip session mechanics
   - **Feature build**: full four-pillar treatment
   - **Long-horizon autonomous run** (hours, several context windows): add a checklist file, the unattended-run block, restart instructions
   - **Multi-lane build** (work split across parallel lanes or engines): add the lane-routing table (Phase 4)
   - **Research / analysis**: success criteria, source verification, structured notes
   - **Writing / creative**: voice, audience, format examples
4. Note whether the task is **goal-suited**: it has deterministic exit criteria that Claude's own output can demonstrate (tests pass, a score clears a threshold, a queue empties, a build exits 0). Goal-suited tasks get a `/goal` companion in Phase 4 and a delivery pattern in Phase 5. Taste-judged outcomes (design quality, prose voice) are not goal-suited, because a small evaluator model cannot judge them; the orchestrator's review handles those.
5. Pick the **target model and effort**: Opus 5.5 at `high` by default (`xhigh` for financial or regulated work and long runs with large budgets); Fable 5.1 at `high` only on the escalation conditions above. A prompt cannot set effort, so the delivery advice states it.

## Phase 2: Reflect

Score the rough prompt against the four pillars. For each, write down what is missing or ambiguous; this list drives the interview.

| Pillar | Crystal when... |
|---|---|
| **Aim** | "Done" is an observable outcome, not an activity ("users can export a CSV of X", not "work on exports") |
| **Verification** | The model can check its own work without the user: tests to write or run, a rubric file, checkable criteria, observable behaviour, and the output it must show |
| **Context** | Every constraint carries its motivation ("this is an experiment, real chance we delete it in a month", not "keep it simple") |
| **Scope** | Boundaries are explicit (what not to touch), and the ambition level is stated (minimal fix, or named extras beyond the basics) |

Also flag if unstated: output format, autonomy level (may it push, deploy, send, spend?), long-horizon needs (checklist file, restart instructions), which lanes and models the work splits across, and convention deviations: any way the user's setup differs from what the model would assume (branch-to-environment mapping, unusual auth, intentional quirks). Strong priors override unstated reality, so these must surface in the interview and land in the prompt's context section.

### Depth pass (every shape except quick tasks)

The pillars check that the prompt is clear. The depth pass asks whether it goes deep enough. Answer four questions about the task, concretely and with names, after investigating (the guide's "Depth, not bulk" section has per-shape examples):

1. **The professional's move.** What would a top professional in this field do here that the user is not asking for?
2. **The rawest material.** What would they start from instead of what has already been digested for the user? Primary sources over summaries, transcripts over notes, the system of record over a status file, a live probe over vendor docs, the dataset over the chart.
3. **Methods and tools.** Which specific methods and tools would they use? Name them (a technique, a library, a query, a protocol), not "analyse thoroughly".
4. **The outside verifier.** What, other than the model or the user, would show the result is right? A test against a real fixture, ground truth held back from the model, a primary source, a third party's data, a replication, a real reader's response.

**Deeper, not bigger.** A depth move changes the source, method or proof behind the aim the user already has. If an answer adds a new deliverable or workstream, it is an ambition question for the interview, not a depth move. Drop any answer that would only make the prompt longer. The depth pass sharpens how the aim is pursued; it does not replace challenging the aim itself, so keep pushing back on the aim where the evidence calls for it.

## Phase 3: Interview until crystal

Interview the user with AskUserQuestion, up to 4 questions per round, as many rounds as it takes. Answers often expose new ambiguity; iterate.

Rules:
- **The aim comes first.** If Phase 2 found the Aim pillar not crystal, round 1 is about the aim: what "done" looks like, who it is for, and any pushback on whether it is the right aim. The depth pass waits for the round after the aim settles, because depth on the wrong aim is wasted. It never takes a question slot the aim still needs.
- Once the aim is crystal (in round 1 if the rough prompt already had it), show the depth pass in four short lines (move, raw material, methods, outside verifier) and ask which of those moves to build into the prompt, with your recommendation. The user may know of a better source or a verifier you missed; that is the point of showing it.
- Ask only what Phase 2 flagged AND the repo cannot answer.
- Offer concrete options with a recommendation, not open-ended essay questions.
- When the user gives a bare constraint, ask for the why behind it. The model generalises from the motivation, not the bare rule.
- When the user does not know what they want yet, act as a thought partner: propose 2 or 3 directions and let them pick, rather than forcing an answer they don't have.
- Surface ambition explicitly at least once: "Is the minimal version the goal, or should the prompt ask for more, and if so which extras?" Opus 5.5 tends to grow scope on its own, so ambition lands as named extras in the aim, not as a blanket "go beyond the basics". On a Fable 5.1 escalation, start at the top of the difficulty range.
- Do not ask whether lanes should run one at a time. Parallelism follows ROUTING.md; a serial run is only ever the operator's own unprompted request.

**Exit test (the only exit condition):** could a colleague with zero context execute from your current understanding without asking a single question? If no, run another round. If yes, stop interviewing; do not pad with extra rounds.

## Phase 4: Draft the prompt

Build the prompt from `references/prompting-guide.md`. Order: context → aim → constraints-with-why → sources and methods (the accepted depth moves) → verification → output expectations → lane routing (multi-lane only) → session mechanics (long-horizon only).

Write the accepted depth moves into the prompt as instructions: the named raw sources to start from (with paths or URLs) and, where a digest exists, say not to rely on it, with the reason; the named methods and tools; and the outside verifier as the verification check, with the output it must show. A move the user declined stays out.

Drafting rules (rationale and sources in the guide):
- Action verbs ("Implement", "Build", "Fix"), never "Can you suggest". Claude 5 models follow instructions literally; suggestion language gets suggestions.
- Tell it what to do, not what to avoid, wherever a positive instruction exists.
- Calm prose. No blanket CAPS, MUST or CRITICAL; current models over-trigger on it. One emphasised line is acceptable only where the user reports that line being missed.
- XML tags only when the prompt mixes content types. Pasted customer text, logs or web content goes in a named tag marked as data.
- Give the check, not an instruction to check: name the command, test, rubric or observable behaviour that proves done, and ask for its output in the final report. Do not add "double-check", "include a final verification step" or "verify with a subagent" lines for Opus: they cause over-verification. Independent review is the orchestrator's step after the run. Lanes delegated to Sonnet 5.5 or Haiku 5.5 are the exception and get the guide's real-check paragraph.
- Ask for "a short explanation" or "a summary of the actions taken", never for the model's reasoning or thinking to be written out (the `reasoning_extraction` refusal hazard). No "think step by step", "think hard" or "think carefully" lines; effort is the lever.
- Use Anthropic's scope line for the target model (the guide quotes them) rather than inventing one.
- Long-horizon: the checklist file, the unattended-run block (named early-stop kinds and the legitimate stops), the grounding line and restart instructions, all in the first message. No token-budget or context-countdown lines.
- Carry the autonomy decision: if the user set boundaries (confirm before irreversible actions, never push, and so on), state them with their why.
- Every line must earn its place. A short crystal prompt beats a long padded one; do not import guide boilerplate the task does not need.

### The lane-routing table (multi-lane handovers)

A handover that splits work across lanes includes a table mapping this run's actual work items to lanes, following ROUTING.md. Include only the rows the run uses, and name the items in each:

```
| Lane | This run's work | Model (effort) | Launched via |
|---|---|---|---|
| Orchestrate, integrate, judge, visual sign-off | <items> | Opus 5.5 main session (high; xhigh for hard calls) | this session |
| UI, user-facing copy, design and API calls | <items> | Opus 5.5 (high) | session or `opus` subagent |
| Well-specced code, clear-repro fixes, tests, sweeps | <items> | Codex gpt-6.1-sol (high), parallel lanes | `delegated-implement` |
| Overflow, Claude-only tools, internal prose | <items> | Sonnet 5.5 (high) | Agent, `model: sonnet` |
| Search, inventory, extraction, Codex wrappers | <items> | Haiku 5.5 (medium; high for extraction) | Agent, `model: haiku` |
| Review | <cheap lane / judgement lenses / checkable lenses> | per ROUTING.md "Reviews" | `pr-review-cycle` |
```

Add, below the table: work that fits two rows takes the higher-judgement row; every spawn sets model and effort explicitly; each subagent or Codex brief restates the constraints it needs (lanes do not see the session's rules); read-only lenses waiting more than about 10 minutes for admission spill to Sonnet (high). A project's declared regulated surfaces run Opus 5.5 at `xhigh` with the Astra lens and the project's own gates.

The skill never writes a blanket "one lane at a time", "serial only" or "no parallel lanes" constraint into a prompt, handover or table. Concurrency is braked by the live admission rail (ROUTING.md), not by prompt text. Write a serial constraint only when the operator asks for one, scoped to that run, with their reason.

### The goal companion (goal-suited tasks only)

For goal-suited tasks, draft a second artifact alongside the prompt: the `/goal` condition. It is a completion contract for an evaluator, not a compressed copy of the prompt. After every turn a small fast model reads the condition plus the transcript and returns not yet met, met, or impossible; it cannot run tools or read files, and a not-yet-met reason steers the next turn. Draft the condition accordingly (mechanics and template in the guide's "Loops and /goal" section):

- Each criterion pairs a **measurable end state** with the **check that proves it**, so the evidence lands in the transcript ("all tests in `src/services/__tests__/split` pass: run `npx jest split` and show the summary", not "the split logic is correct").
- **Constraints that matter**: what must not change ("no other test file is modified", "no new dependencies").
- A **turn cap inside the text** ("stop after 20 turns"); there are no flags.
- Written for a small evaluator model: crisp yes/no statements, no vague quality language.
- Under **4,000 characters** (aim for 3,900 or fewer); report the character count on delivery.
- Context, motivation and instructions stay in the prompt or spec file; the goal carries only what the evaluator needs to answer yes or no.

## Phase 5: Deliver

1. Output the final prompt in a fenced code block, copy-paste ready. When it is also saved to a file or copied to the clipboard, still print the paste-ready text (the `/goal` for the spec-file pattern, or the prompt) in chat and say which file it matches.
2. For goal-suited tasks, deliver the goal companion with the pattern that fits the shape:
   - **Quick verifiable task → goal-only.** One `/goal` invocation carries the task, end state + check, and turn cap. Setting a goal starts a turn, so no separate prompt is needed.
   - **Feature build / long-horizon → spec file + compact goal (the default).** Save the compiled prompt as a handover or plan file; the goal opens with "Implement the spec in `<file>` fully" and then the completion contract. One paste, and every turn, including the first, is goal-evaluated.
   - **Already mid-session → prompt-then-goal.** Paste the compiled prompt, then set the compact goal once work is underway. In a fresh session the spec-file pattern is better: one paste and no unevaluated first turn.
3. Below it, when relevant, 2 to 4 lines of session setup advice: the target model and effort (Opus 5.5 at `high` by default; say when `xhigh` or a Fable 5.1 escalation is warranted and why; changing effort mid-session costs the prompt cache); the loop primitive (turn-based, `/goal`, `/loop` or `/schedule`); a checklist or state file for multi-context-window work; and the orchestrator's independent review after the run. Skip entirely for quick tasks.
4. Ask: run it in this session now, or take it to a fresh session on the target model? (A fresh session is better when this conversation has accumulated unrelated context.)
5. Append a run-journal entry to `references/run-journal.md`: task shape, target model, rounds of interview, what the user revised in the final prompt (if anything).

## Common mistakes

- **Drafting in the first response.** The whole point is the interview. If your first reply contains a refined prompt, you skipped Phases 2 and 3.
- **Asking what the repo can answer.** "What test framework do you use?" is a Read, not a question.
- **Translating constraints verbatim.** "Keep it simple" copied into the output is a failure; the interview should have extracted "this is a throwaway experiment" and the prompt should say that.
- **Aggressive emphasis.** Blanket CAPS and MUST were crutches for older models; current models over-trigger on them.
- **Boilerplate padding.** Importing every guide section into every prompt. A 10-line bug-fix prompt with state-file instructions is worse than the rough original.
- **Skipping the check.** A prompt with no runnable check wastes the model's main strength: self-correction in a loop.
- **Verify-again lines on Opus.** "Double-check", "add a final verification step" or "use a subagent to verify" make Opus over-verify. Give the check and the evidence requirement instead.
- **Asking for written-out reasoning.** "Show your reasoning" or "explain your thinking step by step" can be refused as reasoning extraction. Ask for a short explanation or a summary of actions.
- **Serial-lane constraints.** Writing "one lane at a time" or "serial only" into a handover the operator did not ask to serialise. Parallelism follows ROUTING.md.
- **A multi-lane handover with no routing table.** Lanes then inherit the session model or a guess. Every lane gets a row with model and effort.
- **Interviewing past crystal.** The exit test is the colleague test, not a round count.
- **Depth as bulk.** A depth pass that adds deliverables, sections or workstreams has made the prompt bigger, not deeper. Depth replaces a digested source with a raw one, a vague method with a named one, and a self-check with an outside one.
- **Generic depth answers.** "Use primary sources" and "validate the results" are not depth moves. Name the source, the tool and the verifier, or drop the line.
- **The model or the user as the only verifier.** "Review your work" and "I'll check it" are the two verifiers the depth pass rules out. If no outside check exists, say so in the prompt rather than inventing one.
- **Cramming the compiled prompt into the goal argument.** The 4,000-character cap forces lossy compression of exactly what makes the prompt good (context-with-why), and the evaluator re-reads the condition every turn. Prompt and goal are different artifacts for different readers.
- **Goal criteria the evaluator cannot see.** The stop-check model reads only the transcript and runs no tools. "The dashboard looks right" or "coverage is adequate" are unjudgeable; every criterion needs the check whose output the session will surface.
