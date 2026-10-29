# promptify: learnings

Per-skill process learnings from prior runs. Read at the start of every run.
Entry format: see upskill `references/learnings-format.md`. New entries land `proposed`; only `operator-correction` + `active` enforce.

<!-- No entries yet. First retrospective will populate this. -->

### print-goal-in-chat · status: proposed · source: operator-correction · seen: 1 · updated: 2026-09-30 · tags: #delivery
**What happened:** A long-horizon handover was delivered as a file plus a clipboard copy only; the owner was on their phone with no clipboard and had to ask for the `/goal` text in chat. Phase 5 step 1 (output in a fenced code block) had been skipped.
**Rule:** Always print the paste-ready artefact (the `/goal` for the spec-file pattern, or the prompt) in a fenced code block in chat, in addition to saving it to a file and copying it to the clipboard; state which file it matches.
**Check:** Did the final message contain the exact paste-ready text in a fenced code block?

### skill-native-handover · status: proposed · source: self-observed · seen: 1 · updated: 2026-10-05 · tags: #delivery #handover
**What happened:** The build session would use HyperFrames, whose entry skill runs its own intake interview unless a `BRIEF.md` already exists in the project root. Writing the handover in HyperFrames' own artifact formats (BRIEF.md, STORYBOARD.md, frame.md) lets the build skip a second interview. A setup-order bug nearly defeated it: invoking the skill before `init` and the copy would have triggered the interview anyway.
**Rule:** When the build session will drive a skill with its own intake artifacts, write the handover in that skill's formats too, and make the spec's setup order put the artifacts in place before the skill is invoked.
**Check:** Does the handover produce the downstream skill's no-repeat artifact before that skill's first invocation?

### writing-aims-need-a-success-signal · status: proposed · source: skill-inferred · seen: 1 · updated: 2026-10-29 · tags: #aim #writing
**What happened:** In an aim test, for "write something about X for LinkedIn", both the old and the new skill offered post types as candidate aims ("thought piece", "recruitment") without saying what a reader should do or what would count as success. A blind judge scored both 3 of 5 on observable aims.
**Rule:** For writing and content tasks, each candidate aim names the reader's action or the signal that shows it worked (comments from data holders, DMs, sign-ups), not only the form of the piece.
**Check:** Does every candidate aim in round 1 for a writing task name a success signal?
