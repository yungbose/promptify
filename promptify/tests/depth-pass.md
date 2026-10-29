# Pressure test: depth pass (added 29 Oct 2026)

**Failure mode:** promptify scores the four pillars and interviews well, but the final prompt still starts from digested inputs (prior briefs, summaries, a status file), names no specific methods or tools, and is verified only by the model's own review or the user's approval. The fix must make the prompt deeper without making it bigger, and without weakening aim clarification.

**Scenario A (research, depth expected):** in a startup's company-context repo that already holds a competitive landscape document, "/promptify research which companies are building benchmarks in our field, so I can position us against them".

**Scenario B (quick task, depth skipped):** "/promptify fix a spelling mistake in a script's usage message".

**Scenario C (aim regression):** two deliberately vague prompts in the same repo: "/promptify make our outbound better" and "/promptify write something about our benchmark for linkedin".

**To re-run:** spawn a subagent (Opus 5.5, high) in a real repo with `SKILL.md`, `references/prompting-guide.md` and each scenario, read-only, told to output its first response verbatim (interview round 1 as text, since a subagent cannot open the question dialog), then `NEXT ACTION: <interview round 1 | final prompt>`. Run the skill version before the change on the same inputs. Judge blind with a fresh-context reviewer that did not write the change.

**Pass iff:**
- A: round 1 names (1) a professional's move the user did not ask for, (2) at least one specific raw source more primary than a summary, (3) at least one named method or tool, (4) an outside verifier that is neither the model nor the user; and it adds no new deliverable as a "depth" move (new deliverables appear only as an ambition question).
- B: no depth pass block; the response stays lean.
- C: round 1 is spent on the aim, the depth pass is deferred until the aim is crystal, and a blind aim-only score (diagnosis, observable candidate aims, challenge or hidden decision, share of the round on the aim, a colleague would know "done") is no worse than the old skill's.
- None contains "double-check", "verify with a subagent", "show your reasoning" or "think step by step".

**RED (old skill, 29 Oct 2026):** A: strong repo investigation and a sharp pushback on who the positioning was really for, but no named raw sources ("re-check each row against its primary source"), no named methods and no outside verifier ("coverage and sourcing can still be checked"). Failed raw source, methods and verifier.

**GREEN (new skill, 29 Oct 2026):** A opened with the depth pass: adoption over existence as the professional's move (who submits to and cites each benchmark); papers, repos, dataset cards and leaderboard submitter lists as raw material; a feature matrix plus Semantic Scholar forward citations and Hugging Face download counts as methods; a held-back recall check (hide the entrants already known, measure the share the research rediscovers) plus primary URLs as the verifier. Blind judge: all six criteria passed, preferred over RED. B: classed as a quick task, no depth pass, and it caught that the typo did not exist rather than writing a prompt for it.

**Aim regression (C), found and fixed the same day:** the first version opened round 1 with the depth pass and so took one of the four question slots; aim questions fell from two to one. Fix: while the aim is not crystal, round 1 is about the aim and the depth pass waits. Re-run blind on aim only: better on the outbound prompt (21 against 17 of 25; it tied "better" to the campaign's own kill rule and deadline), level on the LinkedIn prompt (18 against 17). PASSED.

**Judge's caveats worth keeping:** the depth pass sharpens the method; it does not replace challenging the aim. For writing tasks both versions offered post types rather than outcomes with a success signal, a gap that predates the depth pass.
