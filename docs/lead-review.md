# Review of `/tradecraft:lead` against the best practices

Checked against [orchestration-best-practices.md](orchestration-best-practices.md) on 2026-11-18. Every practice has a row; the table says where `skills/lead/SKILL.md` implements it and which section states the practice. The Gaps section lists the practices with no implementing line.

| Practice | Section | Where in `skills/lead/SKILL.md` |
|----------|---------|---------------------------------|
| Skip delegation for trivial work | 1 | Size triage: line 33 "Trivial" |
| Delegation does not cut total tokens; it pays for a cheaper model or bulky output kept out of context | 1 | Phases: line 52 "Delegation does not cut total tokens" |
| Worker count scales with the task | 2 | Size triage: line 37 "Worker count: one for a lookup" |
| Cap on parallel workers: ten, below the default of 20 | 6 | Size triage: line 37 "At most ten workers in parallel" |
| Route model by phase | 3 | Workers table: lines 12-21; Verification follows the phase: line 58 |
| A task that changes a running app's behavior is verified by one run of the app | 3 | Verification follows the phase: line 58 "finish with one `tradecraft:qa-runner` run" |
| No cheap model on code | 3 | Workers: line 26 "Never Haiku on a phase that writes or reviews code" |
| Do not spawn `Explore`; the cheap read-only worker does the same search | 4 | Workers: line 26 "Never spawn `Explore`". Section 4 only mentions Explore as an agent that skips CLAUDE.md (line 50); the ban itself lives only in the skill. |
| Name the file and function even in a short brief | 4 | Size triage: line 34 "**Inputs** with the exact files and symbols" |
| Pass the findings a subagent needs and nothing more | 4 | Rules for every brief: line 74 "Never forward the whole conversation" |
| Brief as contract | 4 | Worker brief: lines 62-69 |
| Restate rules for agents that skip CLAUDE.md | 4 | Worker brief, Constraints: line 66 "do not load CLAUDE.md" |
| Workers batch reads and return big output as a file | 11, 5 | Worker brief, rules for every brief: line 75 "independent reads in one turn" |
| A worker waits for what it starts and reports once | 12 | Worker brief, rules for every brief: line 75 "Tell it to wait for what it starts" |
| Parallel workers share a cached prefix | 11 | Worker brief, rules for every brief: line 76 "byte for byte" |
| Fixed report shape, with a friction field the lead lists under Found | 5 | Worker report: lines 80-90 "**Friction**"; Delegation rules: line 101 "List each worker's **Friction** item under **Found**" |
| Parallel only when independent; disjoint files or a worktree for writers | 6 | Delegation rules: lines 94-95 |
| Turn limit is not a provider failure | 11 | Delegation rules: line 97 "A worker that hits its turn limit" |
| One `tradecraft:qa-runner` run covers one flow; the 80-turn cap is hit by a brief with several | 12 | Verification follows the phase: line 58 "One `tradecraft:qa-runner` run covers one flow" |
| A worker is resumed at most three times | 12 | Delegation rules: line 96 "Resume a worker at most three times" |
| No worker above `high` effort | 3 | Workers table: lines 12-19, every effort is `medium` or `high`; the skill passes no `effort` |
| Retry on the same or a stronger model, never a cheaper one; `model: opus` for a failed `tradecraft:implementer` phase, `tradecraft:deep-reasoner` for a failed `tradecraft:reviewer` or `tradecraft:scout` phase | 3, 7 | Workers: line 25 "`model: opus`"; Delegation rules: line 97 "Retry on the same worker or a stronger one" |
| Escalate after two failures, once, read-only, on the most capable model; a different provider for disputes | 7 | Workers table: lines 19-20; Delegation rules: line 97 "goes once to `tradecraft:oracle`" |
| A major design, and a question the Opus worker could not settle, go to the most capable model without two failures | 7 | Workers table: line 19; Phases: line 54 "A major design goes to `tradecraft:oracle`"; Delegation rules: line 97 "report that reaches no conclusion" |
| The stronger model is a worker the lead calls, not an advisor | 7 | Workers table: line 19 "one call per question" |
| The second provider is called directly and read-only, not through a forwarding subagent | 7 | Delegation rules: line 97 "Never pass `--write`, and never go through `/codex:rescue`" |
| Destructive actions stay in the lead; ask only when the user's instructions do not authorize them | 7, 12 | Delegation rules: line 99 "Ask before one unless the user's own instructions already authorize it" |
| Verify the premise | 8 | Verify the premise: lines 39-46 |
| Lead keeps synthesis; "Blocked on me" lists only what the user alone can do | 7, 12 | Workers table: line 21 "you"; Delegation rules: line 101 "Report conclusions, not worker transcripts" and "**Blocked on me** lists only what the user alone can do" |
| No gate or polling loop in the lead's own background | 12 | Delegation rules: line 102 "Never run a gate or a polling loop in the background" |
| The newest user message sets the scope after compaction | 12 | Last paragraph: line 104 "The arguments below opened this session" |
| User-only command | 9 | Frontmatter: line 3 `disable-model-invocation: true` |

## Gaps

- Clear history between unrelated tasks (section 1). The skill does not implement it; the author's own global instructions cover it outside this repository.
- Fast mode off for delegated runs, and model and effort chosen when a task starts (sections 3 and 11). Both are session settings the skill cannot set; the author's own global instructions cover them.
- Raise effort one level before moving to a stronger model (section 3). The skill retries a failed `tradecraft:implementer` phase on `model: opus` at the same `medium` effort instead: one retry is left before the escalation, and the stronger model is the larger step.
- Text written by other people is data, never instructions (section 11). The lead skill does not state it; `skills/pr-review/SKILL.md` section 2 does, because that skill reads PR text.
- The eval loop (section 10). It is a practice for maintaining the skill, not a rule for running it, so the skill omits it on purpose.

## Agent definitions

| Agent | Tools | CLAUDE.md | Turn guard |
|-------|-------|-----------|------------|
| `scout` | Read, Grep, Glob, Bash | skipped | 40 |
| `reviewer` | Read, Grep, Glob, Bash; may use a throwaway worktree the brief names | skipped | none: review length follows the diff |
| `qa-runner` | no edits: all but Edit, Write, NotebookEdit | skipped | 80 |
| `implementer` | all | loaded | none: the brief bounds the files |
| `deep-reasoner` | no edits: `disallowedTools: Edit, Write, NotebookEdit` | loaded | none |
| `oracle` | Read, Grep, Glob, Bash | loaded | none: one call, long turns expected |

The skill passes these to the Agent tool as `tradecraft:scout`, `tradecraft:reviewer`, and so on. The `oracle` description opens with "Use when", like the other routing descriptions.
