# Pressure test: interview-first (creation gate, 2026-06-11; rescoped 19 Oct 2026)

RED and GREEN below were run against the Fable 5 version of the skill; the re-run instructions are current.

**Failure mode:** given a vague prompt ("make the expenses dashboard better"), the agent drafts a refined prompt immediately, inventing the aim instead of interviewing.

**RED (no skill):** subagent asked to "turn this rough prompt into the best possible Fable 5 prompt" produced a full polished prompt in its first response — guessed that "better" meant summary cards + charts + filtering. Never asked a question. FAILED as expected.

**GREEN (with skill):** subagent given SKILL.md + guide produced interview round 1 (4 questions: biggest problem, observable success, ambition level, constraints-with-why) and declared `NEXT ACTION: interview round 1`. Continued with simulated answers, it correctly applied the exit test, declared crystal, and produced a final prompt containing: 3 observable done-criteria, constraint-whys woven in, calm prose (no CAPS/MUST), self-check instruction, session setup advice, run-here-or-fresh offer. PASSED.

**To re-run:** spawn a subagent with `SKILL.md` + `references/prompting-guide.md` and the vague prompt above, framed as "/promptify make the expenses dashboard better", with the instruction "output your first response verbatim, then NEXT ACTION: <interview round 1 | final prompt>". Pass iff NEXT ACTION is interview round 1 and the round probes aim/verification/ambition/constraint-why. If the run continues to a final prompt, it also fails if that prompt contains "double-check", "verify with a subagent", "show your reasoning" or "think step by step".
