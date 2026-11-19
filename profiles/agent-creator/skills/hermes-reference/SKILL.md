---
name: hermes-reference
description: "Use when designing, connecting, checking or fixing a Hermes agent. Quick reference from the official Hermes docs, checked on this Mac: where things live, limits, tools, connections, security, messaging, verify commands, and fixes."
version: 1.0.0
---

# Hermes reference (for building and checking agents)

Official docs: https://hermes-agent.nousresearch.com/docs (Skills, Tools & Toolsets, Memory, Personality & SOUL.md, Context Files, MCP, Security, Configuration, Messaging, Tips, CLI Commands). Commands below were checked on this Mac's install. If something here disagrees with the live docs or with `hermes <command> --help`, trust those and tell Robi.

## 1. An agent is a profile

A "Bot" in the Hermes app is a profile: `~/.hermes/profiles/<name>/` with its own `SOUL.md`, `config.yaml`, `.env`, `skills/`, `memories/`, sessions and logs. `hermes -p <name> <command>` runs anything as that agent. One gateway serves all profiles (`hermes gateway list`).

## 2. How instructions load

| File | Holds | Loaded from |
|---|---|---|
| `SOUL.md` | who it is: voice, tone, how it handles doubt | the profile home, always |
| `AGENTS.md` | rules for the work in one folder | the **working folder** (outside a git repo, only that folder) |
| `.hermes.md` | same, highest priority | working folder |

- Only one project file type loads (`.hermes.md`, then `AGENTS.override.md`, then `AGENTS.md`, then `CLAUDE.md`).
- One-shot runs (`hermes -p x -z`) use the folder you pass with `--in`. Always pass it.
- Context files are scanned for prompt injection and blocked if flagged. They count against the budget on **every message**, so keep them short.
- Rule of thumb: if it should follow the agent everywhere, `SOUL.md`. If it belongs to one folder or project, `AGENTS.md`.

## 3. Memory (limits matter)

- `MEMORY.md` (agent notes) 2,200 characters. `USER.md` (about the person) 1,375 characters. Both load at session start as a snapshot; changes show next session.
- Memory is per profile. Do not point two agents at one profile.
- Past chats are searchable (`session_search`, `hermes sessions list`). External memory providers are optional (`hermes memory setup`). Check with `hermes -p <name> memory status`.
- Save durable facts (preferences, locations, conventions), not trivia or temporary context. `memory.write_approval: true` makes it ask first.

## 4. Skills

- A skill is `skills/<category>/<name>/SKILL.md` with frontmatter `name`, `description`, `version`, optional `references/`, `templates/`, `scripts/`.
- Loaded on demand: the list of names and descriptions is always visible, the body loads only when needed. Write the **description** so it says *when to use it*. Keep the body under about 24,000 characters.
- Check: `hermes -p <name> skills list`. Audit hub skills: `hermes skills audit`. Invoke by name with `/skill-name`.
- Lean agent: create the profile with `--no-skills` so it carries only the skills it needs.
- `skills.write_approval: true` makes agent-written skills wait for review.

## 5. Tools and toolsets

- Toolsets on this install (`hermes -p <name> tools list` shows each as enabled or disabled): `web`, `browser`, `terminal`, `file`, `code_execution`, `vision`, `video`, `image_gen`, `video_gen`, `x_search`, `tts`, `stt`, `skills`, `todo`, `kanban`, `memory`, `context_engine`, `session_search`, `connections`, `clarify`, `delegation`, `cronjob`, `computer_use`, and plugin toolsets such as `a2a`. MCP servers add `mcp-<server>`.
- Turn a toolset off for one agent with `agent.disabled_toolsets` in its `config.yaml` (`create_agent.py` does this from `toolsets_off`). `hermes tools` with no subcommand is interactive and cannot run from a script; use `hermes -p <name> tools list`.
- Least privilege: give an agent only the toolsets its job needs. Plain questions: browse the web? read and write files? run commands on the computer? create images? speak aloud? run on a timer? call other helpers?
- Some tools need keys (for example web search). A missing key means the tool is not really connected.
- **The `connections` toolset is a side door.** It is on by default and gives a new helper every hosted connector already connected to Robi's account (for example ClickUp, Jira), read AND write, with no include list. Put `connections` in `toolsets_off` unless the brief deliberately uses hosted connectors. Then connect each outside tool on purpose (MCP with an include list, read-only first). After every build confirm with `hermes -p <slug> tools list` and `hermes -p <slug> mcp list`, and look in the helper's own history for tool calls it should not have (`sqlite3 ~/.hermes/profiles/<slug>/state.db "select distinct substr(content, instr(content,'CLICKUP_'), 32) from messages where content like '%CLICKUP_%'"`).

## 6. Connecting outside tools (MCP)

- Servers live under `mcp_servers:` in the **agent's** `config.yaml`. Stdio: `command`, `args`, `env`. HTTP: `url`, `headers`. OAuth servers: `auth: oauth`, then `hermes -p <name> mcp login <server>`.
- Tools appear as `mcp_<server>_<tool>`. Limit them with `tools.include` or `tools.exclude` (include wins). Prefer an include list of only what the job needs, and read-only tools first.
- Secrets: write `${VAR}` in config and keep the value in the agent's `.env` (permissions 600). Never put a key in chat, `SOUL.md`, `AGENTS.md` or config.
- **Placing a key without seeing it (token for a header or `${VAR}`).** Robi creates the key in the vendor's console and runs ONE command in his own Terminal app (not through you): `bash -c 'read -s -p "Paste the key, then press Enter: " T; echo; printf "NAME=%s\n" "$T" >> ~/.hermes/profiles/<slug>/.env; echo saved'` (use `bash -c`: macOS zsh has a different `read -s -p`). Nothing shows while he pastes, and the key stays out of chat and shell history. Tell him to press paste once, then Enter. Check without printing it: `ls -l .env | cut -c1-10` (`-rw-------`), `grep -c '^NAME=.\+' .env`, then the SHAPE (measure the length with `cut -d= -f2-`, not `awk -F=`, which stops at an `=` inside the value) with a small Python that prints only booleans (expected prefix, contains spaces, only allowed characters, length) and one read-only vendor call that prints only the HTTP status (`curl -s -o /dev/null -w '%{http_code}'` with the value read from the file inside the same command, then `unset`). A count of 0 means an empty paste (nothing was on the clipboard); spaces or a wrong prefix mean he copied other text; 401 means rejected. After any failed attempt delete the bad line with `sed -i '' '/^NAME=/d' .env` (never print it) so a retry does not leave two lines, then re-check the permissions and ask him to create or copy the key again. Never read the `.env` contents to diagnose.
- **Catalog install of an OAuth server** (`hermes -p <slug> mcp install <name>`; always pass `-p`, otherwise it lands on the builder's own profile): the install cannot probe the server before sign-in, so it writes the entry with NO tool filter and `mcp list` shows Tools `all`, which includes write tools. The helper cannot reach the tool until a token exists, so fix it in this order: (1) Robi runs `hermes -p <slug> mcp login <server>` himself in a terminal (browser sign-in; interactive, you cannot do it), (2) before anyone uses the helper, list the real tool names with `hermes -p <slug> mcp test <server>` and cut the list to read-only tools with `hermes -p <slug> mcp configure <server>` or `tools.include` in its `config.yaml`, (3) confirm `mcp list` no longer says `all`. Take the names from that output; the native server's tool names differ from the hosted connector's, so do not guess an include list.
- **Public server that needs no login** (for example a catalog lookup): append an `mcp_servers:` block to the helper's `config.yaml` with `url` carrying the tool list in the URL (`?tools=a,b`) AND `tools.include` with the same names (belt and braces, `mcp list` then shows `2 selected`). Back up the config first. `mcp test` must say `Auth: none` and list exactly those tools; prove the no-login claim there instead of trusting the vendor docs. Check with `mcp list` again after any later `create_agent.py --update`.
- **Switching that server to a token (adding run or write tools).** Back up `config.yaml`, then change the one `mcp_servers.<name>` entry: `url` with the longer `?tools=` list, `headers: {Authorization: 'Bearer ${NAME}'}` and `tools.include` with the same names (edit with a short PyYAML load and `safe_dump(sort_keys=False)`, which drops any comments, so diff against the backup). Prove the config holds only the placeholder: `grep -c 'Bearer \${NAME}' config.yaml` is 1 and `grep -c -F "$(grep '^NAME=' .env | cut -d= -f2-)" config.yaml` is 0, and both files stay `-rw-------`. `mcp test` shows `Authorization: ***` and more tools discovered than allowed; `mcp list` must still say `N selected` for exactly the include list. Then restart the gateway (below) before any helper test.
- Verify: `hermes -p <name> mcp list`, then `hermes -p <name> mcp test <server>`, then ask the agent for one harmless read. After config changes run `/reload-mcp`. Startup problems show in `hermes logs`. To prove the helper really used the tool, grep its `state.db` for the server name (`tool_calls like '%apify%'`): history records tools as `mcp__<server>__<tool_with_underscores>`, so a grep for the hyphenated tool name returns 0 even when it ran. Also count calls to any run/write tool it must not have (should be 0). For a paid run tool, set per-call spending limits in the helper's instruction (the server accepts them as call options; check the vendor's minimum first, Apify refuses a cost cap under $0.50) and read the account-level usage limit, which the vendor enforces and is the real wall, back from the vendor's API with the helper's own key instead of trusting what Robi says he set. A `tool_describe` row for a run tool in the helper's history is a lookup, not a call; count actual runs from the vendor's record. After adding run tools `mcp list` must show exactly the include list (`6 selected`) even though `mcp test` discovers extra auto-added tools: the include filter is what hides them.
- Typical failures: 401/403 wrong token or login; 400/405 the endpoint is SSE-only (`transport: sse`); HTML back means the URL is a web page; tools missing means a filter, `enabled: false` or a failed connection; `npx` missing means Node is not found.

## 7. Safety model

- Approvals (`approvals.mode`): `smart` (default), `manual`, `off`. Keep `smart` or `manual`. Never `off`, and avoid `--yolo`.
- One-shot runs, cron jobs and API sessions cannot ask anyone, so risky commands are **denied** there by default. That is correct; do not loosen it without a reason.
- `approvals.deny` takes patterns that are always blocked. Use it for "never" rules that map to commands.
- File tools refuse protected paths (SSH keys, `.env`, OAuth files). But the terminal runs as the same Mac user, so on this Mac a folder rule is a **rule the agent follows, not a wall**. Say that plainly. A real wall needs a sandbox (Docker or similar), which is not installed here.
- Secrets live in `.env` or the Keychain, never copied between agents.

## 8. Reaching the agent

- **Hermes app:** it appears under Bots. **Terminal:** `hermes -p <name>`. **Another agent:** `hermes -p <name> --in <folder> -z "<task>"`.
- **Telegram (one bot per agent).** A token can only be used by one agent, so every agent that should be on Telegram gets its **own** bot. Easiest: Hermes app, Messaging, Telegram, pick the agent, "Create with QR". It creates the bot, finds the numeric user ID, writes `TELEGRAM_BOT_TOKEN` and `TELEGRAM_ALLOWED_USERS` into that agent's `.env` and restarts the gateway. Manual: create the bot with @BotFather (`/newbot`), find the numeric ID with @userinfobot, then run `hermes -p <name> gateway setup` or put the two lines in that agent's `.env`. **Robi handles the token. You never see, type or store it.**
- **Access is locked by default.** Only numeric IDs in `TELEGRAM_ALLOWED_USERS` may talk to it (not usernames). Never allow everyone on an agent that has terminal access. `hermes pairing list|approve|revoke` also exists.
- Telegram groups follow the same session rules as Discord channels: each sender gets a separate conversation unless `group_sessions_per_user: false` is set in that helper's `config.yaml` (`hermes -p <name> config set group_sessions_per_user false`; the docs say the setting is re-read within about 30 seconds). Allowlist changes in `.env` are only read after a gateway restart. A negative chat id is a group. Details and the approval, file-attachment and second-person recipes: `agent-lifecycle` `references/telegram-groups-and-file-delivery.md`.
- After any config change run `hermes gateway restart`, but first make sure nobody's task is running (`active_agents` is 0 in `~/.hermes/gateway_state.json`). `/sethome` in a chat sets where timer results go. In groups: turn off Group Privacy in BotFather and re-add the bot, or make it admin; `telegram.require_mention: true` limits replies.
- **Check:** `hermes gateway status`, `~/.hermes/logs/gateway.log`, `/whoami` and `/platform list` inside the chat, and `~/.hermes/gateway_state.json` showing `<name>:telegram` as `connected`. Then send it a hello from the allowed account and see it answer.
- **Failures:** no reply means a wrong token or a stopped gateway; "Unauthorized" means your ID is not on the allowlist; "concurrent polling" means two gateways use one token; adapter paused means `/platform resume telegram`; silent despite a token means `platforms.telegram.enabled: false`.
- Chat is short by nature: an agent reached by Telegram should keep answers brief and avoid long tables.

### Discord (per Hermes docs, read 2026-11-10)

- Discord works as a bot through the same gateway. Each helper on Discord needs its own bot; Robi creates it in the Discord Developer Portal and keeps the token himself (never in chat). `hermes -p <name> gateway setup` can walk through it.
- Access: `DISCORD_ALLOWED_USERS` lists the numeric user IDs allowed to talk to it. Keep it to the named people only.
- In server channels it answers only when @mentioned, unless the channel is in `DISCORD_FREE_RESPONSE_CHANNELS` (or `DISCORD_REQUIRE_MENTION=false`). In DMs it answers everything. Use a free-response channel for a dedicated command channel, and the default for a team-chat channel where the helper should stay quiet until called.
- Each person gets a separate session inside a shared channel by default (`group_sessions_per_user: true`). One shared conversation needs `false`; explain the trade-off (shared context cost, interrupted runs) plainly.
- Only ONE home channel (`/sethome` or `DISCORD_HOME_CHANNEL`) receives timer output by default. Several output channels (brief, alerts, log) need per-job delivery targets in `platform:chat_id` form; confirm in the docs before promising it.
- A second person cannot see a request pending in another person's session. For any rule that needs a second person's approval, have the helper keep pending approvals in one shared file (for example `Work/pending-approvals.md`) and test it with two users.
- `create_agent.py` accepts `reach` values `app`, `telegram`, `terminal`, `agents` only, and `check_agent.py` does not check Discord. Build with `reach: app`, connect Discord by hand at stage 6 (Robi creates the bot and keeps the key), and prove it with a real hello from an allowed user.
- **Newbie walk-through order (worked on this Mac):** (1) Robi creates the server and channels. (2) Developer Portal: New Application, Bot page, Public Bot OFF, Server Members and Message Content intents ON, Save. If Save says "Private application cannot have a default authorization link", set Installation > Install Link to None, then save Bot again. (3) Ask for the Application ID (not secret) and build the invite link yourself: `https://discord.com/oauth2/authorize?client_id=<APP_ID>&scope=bot+applications.commands&permissions=309237763136` (least privilege). (4) Robi resets and copies the token and runs `hermes -p <slug> gateway setup` himself; never ask for the token in chat.
- **Setup wizard trap:** at "Other allowed users (comma-separated, Enter to skip)" a typed `y` is saved as a user named "y". Warn him before he gets there, and tell him to press Enter at the home-channel prompt. Fix by editing only that line: `sed -i '' 's/^DISCORD_ALLOWED_USERS=.*/DISCORD_ALLOWED_USERS=<ids>/' ~/.hermes/profiles/<slug>/.env`. Verify with `grep -E '^DISCORD_ALLOWED_USERS='`, `ls -l .env` (600), and `grep -c '^DISCORD_BOT_TOKEN=.\+' .env`; never print the token.
- **A new MCP or Discord setup is not seen until the shared gateway restarts.** One-shot runs go through the running gateway, so the helper says "I have no ClickUp tool" until `hermes -p default gateway restart`. It refuses while any chat turn is live, including the builder's own Telegram chat. Ask Robi first, run `sleep 25; hermes -p default gateway restart` as a background terminal job with notify, finish the reply, then verify with `hermes gateway list` (new PID) and a fresh read test. The restart job itself ends with exit 143 because the restart kills its own shell; that is normal, so judge success by the new PID. If the output file instead says 'Refusing to restart the gateway: 1 live session(s)' and lists the builder's own background job, nothing restarted (same PID, and the helper still answers 'cannot run in this configuration'): re-run `sleep 20; hermes -p default gateway restart --yes` as a background job and end the turn, but only when that is the sole live session listed.
- Docs page: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/discord

### Timers (cron), per the Hermes docs (not yet run on this Mac, verify with `hermes cron --help`)

- Two kinds of job. An agent job runs the helper with a prompt and delivers its final answer. A script-only job (`no_agent`) runs a script on the schedule and delivers its stdout as-is, with no model: empty output posts nothing (the watchdog pattern), a non-zero exit or timeout posts an error alert. Scripts must sit in the profile's scripts folder, so Robi copies them there.
- Create with `hermes -p <slug> cron create "every 30m" --no-agent --script <file> --deliver discord:<channel id> --name <name>` or `hermes -p <slug> cron create "<schedule>" "<prompt>" --deliver discord:<channel id>`. Delivery targets are `origin`, `local`, `discord` (home channel) or `discord:<channel>`; an explicit channel id avoids setting a home channel. Manage with `hermes cron list|pause|resume|run|remove`; `list` shows `delivery_failed` and unverified deliveries separately from run failures.
- Cron-run sessions cannot create more cron jobs, and a job whose provider key, skills or delivery target is not configured is marked `blocked_config` with one alert and no model call. A job follows the main model unless pinned.
- Timers need Robi's separate OK, and the helper's own `cronjob` toolset stays off: agent-creator creates the job.

### Hosted connectors (manage_connections)

- A hosted connector such as ClickUp connects to the builder session through a sign-in link that Robi approves in his browser. The builder cannot limit it to read-only, so say plainly that it will only read until told otherwise. The built helper needs its own connection later, with its own approval.
- Find the read tools with `tool_search` ("clickup workspace hierarchy", "clickup custom fields", "clickup get tasks"), load them with `tool_describe`, call them with `tool_call`. They were `CLICKUP_GET_WORKSPACE_HIERARCHY` (max_depth 2 gives spaces, folders and lists), `CLICKUP_GET_CUSTOM_FIELDS` (list_id) and `CLICKUP_FILTER_TASKS` (list_ids plus include_closed to count every task). `CLICKUP_GET_LIST` (list_id) returns the list's configured statuses: read it before trusting a status name the person gives, because the real name can differ in spelling or plural (for example "backlogs" for "Backlog").
- Read the structure first (workspace hierarchy, custom fields, tasks in one list) to see how the tool is really set up. Treat a pasted policy document as a possibly outdated draft and show Robi any mismatch with the live data.

### Voice notes (speech to text)

- Hermes transcribes incoming voice notes automatically (`stt.enabled: true`). With no engine available the agent only gets an audio file; its own log (`~/.hermes/profiles/<name>/logs/gateway.log`) says "Voice transcription failed" and the agent should say so and ask for typing, never guess.
- Engines (`stt.provider`): `local` (faster-whisper, no key, voice stays on the Mac), `groq` (`GROQ_API_KEY`), `openai` (`VOICE_TOOLS_OPENAI_KEY`). Auto order: local, groq, openai.
- **Local setup on this Mac (done 2026-11-10):** install into the *active* Hermes runtime with Hermes's own uv: `~/.hermes/tools/uv-*/uv pip install --python <runtime venv>/bin/python faster-whisper "av==16.1.0"`. The runtime venv is the one `hermes doctor` calls "Runtime venv staged". **Pin `av`:** the newest `av` (19) breaks faster-whisper 1.2.1 with "open() got an unexpected keyword argument 'metadata_errors'"; 16.1.0 and 15.1.0 work. The runtime has no pip. The model downloads from Hugging Face on first use (`small` is about 460 MB in `~/.cache/huggingface`).
- Settings per agent: `hermes -p <name> config set stt.provider local`, `stt.local.model small`, `stt.language ""` (auto-detect; the default `en` forces English and ruins Tagalog or Taglish). Then restart the gateway with `hermes -p default gateway restart` when no task is running.
- The docs do not say whether the install survives `hermes update`; after an update, re-check with a spoken test and reinstall if needed.
- Test without a person: `say -o sample.aiff "..."`, then transcribe it with the runtime's Python and `faster_whisper.WhisperModel('small', device='cpu', compute_type='int8')`. Then ask Robi to send a real voice note and check the agent's log.

## 9. Verify toolbox

| Question | Command |
|---|---|
| Is Hermes healthy? | `hermes doctor` |
| Is the agent configured? | `hermes -p <n> config check`, `hermes -p <n> config get <key>` |
| Is it served? | `hermes gateway list` |
| What does every message cost? | `hermes -p <n> prompt-size --json` |
| Which tools and skills? | `hermes -p <n> tools list`, `hermes -p <n> skills list` |
| Are connections alive? | `hermes -p <n> mcp list`, `hermes -p <n> mcp test <server>` |
| What happened? | `hermes logs errors -n 50 --since 1h`, `hermes sessions list` |
| What is it remembering? | `hermes -p <n> memory status`, read `memories/MEMORY.md` |
| Which model logins exist, in what order? | `hermes -p <n> auth list` |
| Does it answer? | `hermes -p <n> --in <folder> -z "<prompt>"` (exit 0 ok, 2 failed or partial, 130 interrupted, 1 no text) |

## 10. Cost and quality

- Routine work: a fast model. Judgment and careful writing: a stronger one. Avoid switching models mid-conversation.
- Keep the instructions stable and short so caching works; check with `prompt-size`. A stable system prompt is cached after the first message, so its length costs little per turn; the large costs are workers (subagents), a long chat history and large tool schemas.
- Real cost per session: `sqlite3 ~/.hermes/profiles/<n>/state.db` table `sessions` (input, output, cache read and write tokens, `estimated_cost_usd`, `parent_session_id` set for workers). Recipe and levers (`delegation.model`, `max_concurrent_children`, `max_iterations`): `agent-requirements` `references/helpers-with-workers.md` section 5.
- Be specific in requests. Put recurring rules in `AGENTS.md`, recurring procedures in a skill.

## 11. Symptom to fix

| Symptom | Likely cause and fix |
|---|---|
| Ignores messages on Telegram | not on the allowlist or not paired; adapter paused (`/platform resume`) |
| Works in the app, not in a one-shot | forgot `--in`, or the action needs approval (denied in headless runs) |
| "I can't do that" for a tool it should have | toolset disabled, MCP filter, failed MCP connection, or missing key |
| Says it cannot access an attached PDF/document | first check the helper's `state.db` for the user message; app attachments include a saved file path. If the file tool is on, reproduce with `hermes -p <name> --in <folder> -z "read this path ..."`. If that works, the fix is an instruction/test update: the helper must try the saved path before asking Robi to retype. |
| Never uses its skill | skill description does not say when to use it; skill not in its profile |
| Forgets things | memory is full (2,200 characters); consolidate, or use session search |
| Works in the wrong folder | working folder not set, or `AGENTS.md` missing there |
| Slow and expensive | system prompt or skill list too large; use `prompt-size`; lean skills; a faster model |
| Asks approval all the time | approvals mode `manual`; consider `smart`, or allow a specific safe pattern |

## 11b. Model logins (credential pool)

- `hermes -p <n> auth list [provider]` shows each login's label, type and short id only (no secret values), so it is safe to run and show. `auth status <provider>` says logged in or not.
- Set which login is tried first: `hermes -p <n> auth priority <provider> <label|index|id> 0` (0 = first under `fill_first`; the others are renumbered). Undo by setting the old one back to 0. Other subcommands: `refresh` (renew an OAuth login and clear its cooldown), `reset` (clear exhausted marks), `remove`, `logout`. There is no `reorder`.
- The pool is shared: the list read with `-p <helper>` and with `-p default` is identical, so one change reaches every helper (about 15 on this Mac). Say that before changing it, and say it is a first-choice setting, not a lock: if the first login hits a limit or expires, Hermes falls back to the next one.
- The `←` marker stays on the last-used login and did not move after a priority change plus a test request. Do not tell Robi the new login is "in use" from the marker; say the order is set and that real use is not confirmed.

## 12. A second Hermes machine (company VPS in Docker)

The Agent Creator method can be copied to another Hermes install (Robi's company Hostinger VPS) as a separate, empty copy. Read `references/vps-docker-copy.md` first: container and `/opt/data` layout, the Python-with-yaml setup, what to put in the bundle, the link-based home folder that lets the unmodified scripts find the profiles, running scripts as the `hermes` user, and the model-route check (the build script hard-codes the Anthropic key route; check what the server really uses before building).
