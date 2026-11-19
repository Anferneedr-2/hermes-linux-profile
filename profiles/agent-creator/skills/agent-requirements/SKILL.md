---
name: agent-requirements
description: "Use when deciding whether and what helper (agent) to build, checking it meets every requirement, writing the Agent Brief with tests, and building it only after approval."
version: 2.0.0
---

# Agent requirements, readiness gate, brief and build

Companions: `agent-lifecycle` (the 10 stages), `hermes-reference` (Hermes facts), `~/Team-Workspace/Agents/CONVENTIONS.md` (how agents are organized), `~/Team-Workspace/Agents/AGENT-REQUIREMENTS.md` (the full checklist). Talk about all of it in plain words (see `socratic-discovery`); never paste it at the person.

## 1. Is a new helper the right answer?

1. **No helper needed?** Would a checklist, a template or a better habit solve it?
2. **An existing helper?** Run `hermes profile list` and `python3 ~/Team-Workspace/Agents/bin/registry.py status` (listing skills does not show helpers, so it is not this check). Could a live helper take this on? When a role Robi names sounds like a live helper, read that helper's brief ("In plain words") before saying whether they overlap; never judge from the name (a "chief marketing officer" may really be a document-writing strategist, not a day-to-day manager).
3. **One job?** If the need has two unrelated jobs, plan two helpers, or build the first job now. If the parts are facets of one role on one system (for example a project manager for one ClickUp: add tasks, check status, daily digest, reports), keep ONE helper and build it in stages, each stage tested before the next is added. Do not force a split the person pushes back on; the one-job rule is about one role, not one command. Ask which stage matters most only to set the build order.

4. **First piece of a bigger platform?** When Robi describes several helpers that will later work together (for example an onboarding helper that will call a research helper), build and brief only the one he names as first. Record the others in the brief's "Not known yet" as later, separate helpers, and keep their duties, permissions and costs out of this brief. The hand-off between them is settled when the second helper is designed, through an updated brief and a new approval. Confirm the split with him as one assumption before the readiness gate. If the platform will control who may reach the helper (role-based access), record that as a later platform concern: the helper is built and tested here first, and the platform's access rules are not part of the build.

   If the helper's output is a shared record that later helpers will rely on (a per-client folder, a "source of truth"), settle these early because every later helper inherits its mistakes: the exact list of items it must hold before it may say "complete" (ask "is that the full list?" and name likely gaps such as goals, audience, contacts), where the folder lives, which summary document it produces and who sends it on (the helper never contacts the outside person), what file types it must read (PDF, Word, images, links), and that it asks instead of guessing when an item is missing or conflicts.

For a named client, also search `Team-Workspace/CHANGELOG.md` and the registry for helpers built and later retired for that client (`grep -i <client>`): their brief files may be gone. Tell Robi what was retired and say you treat the new request as a fresh start unless he asks to reuse it.

Say the result plainly ("I think this is a new helper / an existing helper / not a helper job, because ..."). Let Robi disagree.

If Robi talks about a team of helpers for a client, a reusable library of helpers, which helpers are generic or project-specific, reorganizing the helper files, a creator for project teams, or pastes another workspace's structure to compare, read `references/helper-library-and-project-teams.md` first.

If the helper's job is turning one document into another (a milestone report into an invoice, notes into a summary), read `references/document-in-document-out-helper.md` before the interview goes far: get a real input and a finished output early, read and render both, and compare them with what Robi described.

If the helper collects what a client already has into a per-client record that later helpers trust (a "201 file"), read `references/intake-helper-client-record.md` before the interview goes far.

If the request is really for software (a plugin, app or service) with a Hermes helper as its brain, read `references/helper-with-companion-software.md` before the interview goes far: it separates the helper from the companion code, and lists the design questions (who calls whom, where safety rules live, backups, secrets). When the design is approved for a stage, build and test the code here with `references/wordpress-plugin-build-and-test.md`.

If Robi supplies a finished prompt or spec file and says "just build it", read `references/build-from-supplied-spec.md` for the short path (draft brief, dry run, one approval question). If the supplied file is a tool kit (scripts, templates, an install briefing), also read `references/helper-around-a-tool-kit.md`.

If Robi wants a helper that finds leads with Apify, scrapes LinkedIn or a members directory such as BNI, or builds a many-leads funnel, read `references/apify-prospector-and-lead-funnel.md` first (connection, token handling, the $0.50 cap, terms and personal-data facts).

If Robi asks for or reviews sales-team helpers (researcher, conversation agent, trainer, client knowledge interviewer), a whole AI SDR, or mentions RIBO, read `references/sales-team-helpers-and-ribo.md` first.

If Robi wants a helper that talks with employees, assesses them and reports to an admin (onboarding, coaching, NTE or incident interviews), read `references/assessment-and-coaching-helper.md` before the interview goes far: interview order, what "cycle finished" means, the honest limit of what an AI can detect, and the scanned-books step.

If Robi wants a helper that watches or manages websites or systems (WordPress plugin, updates, security, logs, alerts to Discord), read `references/website-watchdog-helper.md` first: staged shape, where files must live, how to test without a live site, and the traps already met.

## 2. Readiness gate (do not write the brief until every line is true and confirmed)

- [ ] **The problem** and who has it
- [ ] **The workflow today**, step by step, and **where the helper enters** it
- [ ] **Who talks to the helper** and how it reaches them (user journey)
- [ ] **Who its work reaches** (customer journey) and which moments it must **not** touch
- [ ] **The tools and systems** the work uses, and whether the helper must read or change them
- [ ] **Which clients or projects it covers.** Ask it early, and never assume one from the first mention: the person's "client" can mean one of several clients served through the same tool. Read the live client options (for example the Client dropdown) and let the person pick. If it covers more than one client, it is a team helper
- [ ] **Where it lives**: a client's folder or its own team folder
- [ ] **What "done well" looks like** and how we will know it worked
- [ ] **What it must never do**, **what needs approval first (and from whom)**, and **what must be logged**. If an approver is also a subject of the work (for example the assistant who approves invoices is one of the people invoiced), say so plainly and offer a safeguard; record his choice
- [ ] **What the helper does when a detail is missing** (ask, never guess) for anything it writes to a shared system
- [ ] **A first test task** and at least one task it must refuse or ask about

If a line is not true, name the gap and ask the next question. Never fill a gap yourself.

## 3. Requirements to settle (plain question, a recommendation, Robi decides)

| # | Settle | Plain question | Default if unsure |
|---|---|---|---|
| 1 | **Job** | "In one sentence, what is its one job?" | must be answered |
| 2 | **Where it lives** | "Which clients should it cover?" then "Is this for one client, or a helper for the whole team?" | client folder only if it serves exactly one client; several clients means a team helper with its own team folder (house rule: client helpers never cross into another client's folder) |
| 3 | **Name** | "What should we call it?" (the role only; the client name is added) | job-based, short |
| 4 | **How you reach it** | "Hermes app, Telegram, or another helper calling it?" | Hermes app |
| 5 | **AI brain** | "Routine work, or judgment and careful writing?" | routine: `claude-haiku-5-5`; judgment: `claude-sonnet-5-5` |
| 6 | **Built-in tools** | "Does it need to browse the web? Read and write files? Run commands? Create images? Speak?" | only what the job needs; the rest go in `toolsets_off`. Always include `connections` in `toolsets_off` (it exposes every hosted connector on Robi's account with write access) unless the brief deliberately uses it. A staged build turns off the toolsets of later stages (`cronjob`, `code_execution`) until that stage is approved |
| 7 | **Outside tools** | "Does it need to use Jira, Notion, Drive or similar? Only read, or also change things?" | none; read-only first |
| 8 | **Skills** | "Does it need a special method, or just its instructions?" | none, lean |
| 9 | **Memory** | "Remember lots of facts over time, or only the chat?" | built-in notes |
| 10 | **Timer** | "Should it run on its own at set times?" Settle the time, the time zone, and which days (weekdays only or every day). | no (needs a separate OK) |
| 11 | **Passwords** | "Does it log in to anything?" | no (Robi stores any key himself, never in chat) |
| 12 | **Hand-offs** | "Does it answer only to you, or hand work to another helper?" Temporary workers inside the helper, and how to keep them cheap: read `references/helpers-with-workers.md` | Robi |
| 13 | **Rules** | "What must it never do? What must it ask first? Any command that must always be blocked?" For risky-but-allowed actions (deleting, changing due dates) settle three things: who approves, that the helper waits for that approval, and that an activity log records who asked and what changed. Treat "allowed with approval and logged" as a rule, not a never. | ask first before sending, publishing, spending, deleting |
| 14 | **Tests** | "What is a first real task? What should it refuse?" | intro + normal + boundary |
| 15 | **Review** | "When should we look at how it is doing?" | after 3 tasks or 7 days |

Fixed by the house rules (do not ask): every agent is registered, starts as `testing`, gets a home folder with README, AGENTS.md and CHANGELOG, and is only promoted to `production` after Robi approves a correct real result.

For a helper that lives in a team chat over a work tool (ClickUp, Jira, Notion), also read `references/chat-helper-design.md`: channel layout, alert types, timer cadence, approvals and reports.

## 4. The Agent Brief

Save to `~/Team-Workspace/Agents/briefs/<name>.md`. The top is plain words for Robi. The build spec at the bottom is for the scripts.

````markdown
# Agent brief: <plain name>

Status: DRAFT   (change to: APPROVED by Robi on YYYY-MM-DD, only after he says yes)
Date: YYYY-MM-DD

## In plain words
<5-8 short lines: the problem, who has it, what the helper will do, what it will not do, how we'll know it works>

## The problem
## The workflow today
<numbered steps, who does each, and where the helper enters>
## Who talks to it (user journey)
## Who its work reaches (customer journey)
## Tools and systems it touches
## What it does and what it must never do
- Does:
- Never:
- Asks Robi first before:
## How we will know it works
## Tests
<the intro, normal, edge, boundary, scope and tool tests, in words>
## Not known yet
## Why a helper (and not something simpler)

## Requirements check
| Requirement | Answer |
|---|---|
<the 15 rows above, answered>

## Build spec
(Simple format: `key: text` lines, and `- item` lines under list keys. No quoting rules. The block must sit inside a ```build-spec fence or the dry run stops with "no build-spec block found"; copy the template instead of retyping it. Give each test distinct prompts and `expect:` words that only a correct answer holds; generic words copied from the job description make a weak test.)
```build-spec
# client:<ClientFolder> works in that client's folder; team:<TeamFolder> gets its own team folder
placement: team:Example-Team
# the role only; for a client agent the client prefix is added automatically
name: example-helper
description: One paragraph saying what it does and for whom.
# claude-haiku-5-5 for routine work, claude-sonnet-5-5 for judgment and careful writing
model: claude-haiku-5-5
# lean = only the skills it needs (recommended); standard = all built-in skills
skills_mode: lean
# names from ~/Team-Workspace/Agents/skills, comma separated, or leave empty
skills:
# builtin or holographic
memory: builtin
# built-in toolsets it must NOT have: web, browser, image_gen, tts, code_execution, terminal, cronjob, delegation, connections ...
toolsets_off:
  - image_gen
  - tts
# command patterns that are always blocked
block_commands:
intent: The goal in one or two sentences.
role: Who it is and what it is responsible for.
instructions:
  - one plain, direct rule per line
never:
  - ...
ask_first:
  - ...
context:
  - ...
examples:
  - Robi: '...' -> It replies: '...'
# outside tools: name | method (mcp, builtin or skill) | what it is for | read-only or can change | secret name only
connections:
# tests: kind | prompt | expect: words the answer should contain
tests:
  - normal | the first real task, in words | expect: key words
  - boundary | a request that must make it ask first | expect: ask, approval
```
````

Test scoring in `check_agent.py` (write tests to match it): a `boundary` test passes only if the answer holds back or asks (words like ask, approval, first, cannot), whatever `expect` says; every other kind passes if the answer contains ANY `expect` word, otherwise it shows REVIEW. So give boundary prompts a request that must be held back, and choose short `expect` words that the helper's own refusal or plan will really contain.

## 5. Approval, build, verify

1. Show Robi the **"In plain words"** part, the limits and the readiness result. Ask: "Shall I build it as written?" Do not build on silence or a maybe. Robi answers in short words ("Ok", "Oki", "Go"): a short yes given directly to that exact question, after the plain-words plan was shown, is the clear yes; quote his exact reply in the Status line so the record shows it. A short "Ok" to anything else (a status update, a side question) is not approval.
2. `create_agent.py <brief> --dry-run` needs no approval: run it (with `/usr/bin/python3`) on the DRAFT brief and check the name is free BEFORE asking "Shall I build it as written?", so the plan Robi sees is already checked. After a clear yes set `Status: APPROVED by Robi on <date>` and run (from `~/Team-Workspace/Agents/bin/`):
   - `python3 create_agent.py ../briefs/<name>.md --dry-run`
   - `python3 create_agent.py ../briefs/<name>.md --approved --smoke`
   - `python3 check_agent.py <slug> --scenarios`
3. Change an existing helper: update the brief, get a new approval, dry-run with `create_agent.py ... --dry-run --update` (without `--update` a dry run of an existing helper stops with "already exists"), then `create_agent.py ... --update --approved` (backs up first). Retire: `create_agent.py --retire <slug> --approved`.
4. Report in plain words after each command. Follow `agent-lifecycle` for connecting tools, hand-over, first real use and review.
5. Never create timers, store passwords, connect outside tools, or message anyone without a separate, explicit OK.
6. A staged helper is built stage 1 only. The brief says so, and later stages come through an updated brief, a new approval and `--update`.
