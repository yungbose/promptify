# promptify: retrospective checklist

Skill-specific yes/no checks for the end-of-session retrospective. Each catches a known failure mode.

1. **Interview-first:** Did the skill ask clarifying questions before producing any draft prompt? (Failure: refined prompt in the first response.)
2. **Repo-answerable questions:** Was every interview question something the repo/files could NOT answer? (Failure: asking "what test framework?" instead of reading config.)
3. **Constraint motivation:** Does every constraint in the final prompt carry its why? (Failure: "keep it simple" copied verbatim from the user.)
4. **Verification baked in:** Does the final prompt name checks the model can run without the user (tests, rubric, observable behaviour) and ask for their output, without "double-check" or "verify with a subagent" lines on Opus?
5. **Calm prose:** Is the final prompt free of CAPS/MUST/CRITICAL emphasis?
6. **Right-sized:** Did the prompt include only the sections the task shape needed (no long-horizon boilerplate on a quick fix)? Was it shorter or equal in ceremony to what the task warranted?
7. **First-version acceptance:** Did the user accept the prompt without major revision? If revised, what specifically changed — that delta is a candidate learning.
8. **Lane routing:** For a multi-lane handover, is there a routing table that follows ROUTING.md (model and effort per lane), and is the handover free of any one-lane-at-a-time or serial-only constraint the operator did not ask for?
9. **No reasoning extraction:** Is the prompt free of "show your reasoning" or "write out your thinking" style instructions?
10. **Depth, not bulk:** For non-quick tasks, did round 1 show the depth pass with named raw sources, methods and an outside verifier (neither model nor user), and did the final prompt add no deliverable the user had not chosen?
