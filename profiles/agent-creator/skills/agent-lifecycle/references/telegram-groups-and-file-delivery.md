# Telegram: sending files for approval, group chats, adding a second person

Use when a helper must hand Robi a file (an invoice PDF, a report) inside Telegram, when its Telegram chat is a group, or when a second person is to talk to it or approve for him.

## 1. "It cannot send the file" is usually the helper's own rule

- First read what the helper actually said: `sqlite3 ~/.hermes/profiles/<slug>/state.db "select role, substr(replace(content,char(10),' '),1,600) from messages m join sessions s on s.id=m.session_id where s.source like '%telegram%' and role in ('user','assistant') order by m.timestamp"`. Then read the `[Telegram]` lines of `~/.hermes/profiles/<slug>/logs/gateway.log` (they show inbound and sent messages and any send error).
- A rule such as "never paste bank details in chat" is read by the helper as "never attach the PDF" and it answers with a summary and a file path. Reword the rule: never TYPE the private values as text; attach the file only where Robi has said who may see it. Change it through an updated, re-approved brief.
- The gateway sends a local file when the reply carries `MEDIA:<absolute path>` on its own line. Put that in the helper's instructions for the approval step. Scenario tests run one-shot with no Telegram, so they cannot prove delivery; only a real message in the chat does. Ask Robi to send `/new` first (a running chat keeps its old instructions), then a real request, and ask whether the file arrived and opens.
- The log line `[Local] send_document fallback: native file send unavailable` comes from your own one-shot test runs, not from Telegram. Do not read it as a Telegram failure.

## 2. Group chats

- A negative chat id in the log (for example `-5388...`) is a group. Everything the bot posts, including an attached file, is visible to every member. Before letting a helper attach private data (bank details, pay), ask who is in the group and whether each person may see it; record the answer as Robi's decision in the brief, and say plainly that the helper cannot tell if someone new joins.
- The allowlist controls who can give orders, not who can read replies. `Blocked unauthorized user <id>` in the log means that sender is not on the allowlist.
- By default each sender in a group has a separate conversation (`group_sessions_per_user: true`), so a second person's "approved" cannot see a draft the first person started. For approval by either of two people set `group_sessions_per_user: false` in the helper's own `config.yaml` (docs: one shared conversation per room; the setting is re-read within about 30 seconds). The cost is one shared context and one invoice at a time. Offer the choice in plain words and get a yes. Robi can set it himself with `hermes -p <slug> config set group_sessions_per_user false`; verify with `config get` and by grepping the helper's `config.yaml`. The key survives `create_agent.py --update`.
- With one shared conversation, `/new` wipes the pending draft from the helper's memory. It can still find the draft by listing its `Work/` folder, so tell people to say "approved" together with the invoice number and person, and have the instructions say that a message which does not clearly approve a named item ("I am approving Alvin's report") is not an approval.
- Put "approval comes from Robi or <named person>; nobody else's counts" in the helper's instructions.

## 3. Letting a second person talk to the helper

- The person sends any message to @userinfobot in Telegram and passes Robi the number it shows (an id is not secret). Robi appends it to the allowlist line of the helper's `.env` himself (never print the token or the whole file): `sed -i '' 's/^TELEGRAM_ALLOWED_USERS=.*/&,<her id>/' ~/.hermes/profiles/<slug>/.env`, then checks `ls -l` still shows 600. You may not edit that file.
- The new id is only picked up after a gateway restart (`gateway-restart-and-discord.md`): ask Robi first and make sure `active_agents` in `~/.hermes/gateway_state.json` is 0, because the restart affects every helper.
- Run the append once: running it twice adds the id twice (harmless, but untidy). Check the allowlist by script, printing only the count, whether the new id is present and any duplicates, never the ids.
- The helper sees only a sender's display name, not who they are. When the second person first writes, it may refuse to treat her as the approver ("I cannot confirm you are the executive assistant"), especially when her display name equals a name in its data. Fix in the instructions: Telegram lets only the named people through, so treat every message that reaches the helper as coming from one of them, whatever the display name, and never ask who someone is.
- If the second person is also a subject of the work (the assistant who approves invoices is herself invoiced), say so plainly before changing anything and offer the safeguard: her own item needs the owner's approval. It is a rule the helper follows, not a lock, because it only sees display names. Record whichever choice Robi makes as his decision in the brief.
- Afterwards ask both people to send `/whoami`, then one real request each. Update the brief's reach, approval and privacy rows so the registry and the hand-over card say who may use it.
