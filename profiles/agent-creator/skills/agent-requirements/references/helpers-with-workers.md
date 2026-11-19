# Giving a helper temporary workers ("agentic", "multiple agents", "subagents")

Use when Robi asks for a helper to be "agentic" or to have "multiple agents in it".

## 1. Find out what he means (one question, with options)

"Agentic" can mean: (a) does multi-step work itself (runs calculations, saves drafts); (b) reads his real system (needs a connection and his OK); (c) runs on a schedule (needs a timer and his OK); (d) has several workers. Ask once which he means. If he says "subagents", that is Hermes delegation: temporary workers the lead starts for one job. They are NOT separate registered helpers. State that as an "I assumed" line and offer the other reading (separate registered helpers plus a lead) so he can correct it.

## 2. How Hermes delegation works (read the live docs page "Subagent Delegation" if unsure)

- The lead calls `delegate_task`. Workers start with an empty conversation: the lead must put the goal, the exact data, the layout and every limit into each task.
- Workers inherit the lead's enabled toolsets. They cannot be given more, and the lead cannot narrow them per call. So the helper's `toolsets_off` fences the workers too. Decide that fence before turning delegation on.
- Workers cannot ask the user, use memory, message anyone or schedule timers. They return a summary only.
- Delegation is a toolset: take `delegation` OUT of `toolsets_off` to enable it. Leave `code_execution`, `cronjob` and `connections` off unless separately approved.
- Limits on a folder or file rule are rules the workers follow, not walls (they keep the lead's file tool). Say so; the lead should be the only one that saves files.

## 3. What goes in the brief and the helper's instructions

- Worker roles taken from the job's real sections (for example receivables, payables, reconciliation, reports) plus one independent **Checker** that re-does the key totals.
- When to use workers: only big or multi-part jobs, never a small question (each worker is a separate model run, so cost and time multiply).
- Rules the lead copies into every worker task: read and analyze only, save nothing, contact no one, every authority limit of the lead, never invent figures, say "not provided", return facts, calculations with arithmetic, observations, assumptions, missing information.
- Write "return text only, never create, save or edit a file" literally in the worker rule and in `never`. "Drafts only" is not enough: workers then saved draft files in Work/ unasked in a live test. After a live worker test, list the Work folder to catch stray files.
- **Document-writing and self-scoring helpers** (marketing, reports, plans) get roles by kind of work, not by document: a Researcher (real facts, each with source link and date, unverified items marked), a Numbers Analyst (only figures given or sourced, every step of the maths shown), a section Specialist (drafts one section from that document's reference ideas), an Independent Evaluator (scores a draft against the criteria without being told the author's reasoning, so scores are not softened; the lead shows its scores honestly), and a Panel (simulated focus group or expert opinions, always labelled simulated, never research). Cap simultaneous workers (for example 4) and none for small requests.
- The lead rechecks what workers return, reports disagreements instead of choosing one, uses the Checker before the high-stakes reports, and ends with a "Workers used:" line.
- Tell Robi plainly: more cost, slower, workers are not registered or tested separately, and the instructions get longer (the long-instructions warning grows).

## 4. Applying and testing

- Existing helper: update the brief, new approval, then `--dry-run --update` (a dry run without `--update` stops with "already exists"), then `--update --approved --smoke`. Move the old brief to the scratch folder first if you want a copy.
- Add two tests: a small full review that must name its workers (`expect: workers, checker`), and a boundary asking the workers to approve a payment or change bank details (must still refuse, start no workers).
- One-shot test runs (`check_agent.py`, `-z`) allow only 2 workers in total (`delegation.oneshot_max_children`); the app and gateway sessions are not limited. A test that asks for the whole team of workers is refused, the helper falls back to doing it alone and says so, and the word-match test still PASSES. So write the worker test to ask for at most 2 workers, and also run it by hand with `hermes -p <slug> --in <folder> -z "... use at most 2 workers ..."` and read the full answer for a real "Workers used" line.
- **Prove real use from the helper's history, not from its "Used:" line** (that line is a self-report). The one-shot run's session id starts with `local-`, and the newest session is often a different check run, so find the session by `sqlite3 ~/.hermes/profiles/<slug>/state.db "select session_id, count(*) from messages where tool_calls like '%delegate_task%' group by session_id"`. Then read each call's `goal` and `context` (the `tool_calls` JSON) to confirm the fence rules were passed, and the `role='tool'` rows to see what came back. Check that sources and calculations in the answer trace to those results.
- Confirm in the real app on the first real job; one-shot runs cannot show a full-size worker team.
- If check_agent.py FAILs with "toolset delegation is NOT turned off" right after the update while config.yaml is correct, registry.yaml kept the old `toolsets_off`. Never hand-edit it: re-run the same `--update --approved` (it backs up first), confirm the registry row, re-run the check.

## 5. Keeping a team helper cheap (when Robi asks for "token efficient" or "low usage")

Measure first, then change the biggest cost. Workers are where the tokens go: each one is a full model run, so a lead plus two workers cost several times a plain answer, while the lead's long instructions are cached after the first message and cost little to repeat. Do not trim the user's command layouts for caching reasons; cut worker count, worker model and worker length instead.

- **Measure from the helper's history:** `sqlite3 -header -column ~/.hermes/profiles/<slug>/state.db "select substr(id,1,12) id, model, message_count, tool_call_count, input_tokens, output_tokens, cache_read_tokens, cache_write_tokens, round(coalesce(actual_cost_usd,estimated_cost_usd),3) usd, (parent_session_id is not null) child from sessions order by started_at desc limit 14"`. Rows with `child=1` are workers. The cost is Hermes's own estimate; say so. `hermes -p <slug> prompt-size --json` shows the per-message size (system prompt plus tool schemas).
- **Cheaper worker brain:** `delegation.model` pins every worker to one model (no per-task choice) while the lead keeps its own. Set it with `hermes -p <slug> config set delegation.model <model>`; `create_agent.py` has no spec key for it, but `--update` keeps other config keys, so confirm with `config get delegation` after the next update. Re-run the team tests, because a cheaper worker can write weaker reports; the lead rechecks them anyway.
- **Caps (config keys under `delegation`):** `max_concurrent_children` (default 10), `max_iterations` per worker (default 250), `child_timeout_seconds` (0 = none). Set lower values and tell Robi the numbers.
- **Lead rules (in `instructions`):** answer small questions alone; default to 2 workers and use the full team only for portfolio-wide jobs; hand each worker only the documents it needs; ask each worker for a short report capped in length.
- **Smaller messages:** turn off toolsets the job never uses (for example `vision`); each enabled toolset adds tool schema bytes to every message.
- Workers load the working folder's `AGENTS.md` but not `SOUL.md`, so the fence rules must travel in each task text, and a long folder `AGENTS.md` is paid for by every worker.
- Apply through the normal path: updated brief, new approval, `--update`, re-check. State the measured before and after in the brief.
