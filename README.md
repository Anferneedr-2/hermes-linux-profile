# Portable export — Agent Creator + Multi-Agent Engineer + generic helpers

Built on <this Mac>, for use on another Hermes device. Contains NO session
history, memory, logs, cache, credentials, or client data from the source
device. You still need to do provider/credential setup on the new device.

## What's inside

- `profiles/` — 7 sanitized Hermes profile folders, each with only
  `SOUL.md`, `profile.yaml`, `config.yaml` (model/toolset settings, no
  secrets), and `skills/`:
  - `agent-creator` — builds new single-purpose helpers from an interview.
  - `multi-agent-engineer` — assembles teams of helpers for a client project.
  - `team-lead`, `short-text-writer`, `reviewer`, `memory-manager`,
    `document-engineer` — the generic helper templates the two builders
    copy from / recommend as a starting team.
- `team-workspace-agents/` — the non-Hermes scaffolding both builders need
  to actually work: build/check/registry scripts (`bin/`), governance docs
  (`CONVENTIONS.md`, `AGENT-REQUIREMENTS.md`, `SCHEMA.md`), method skills
  used while interviewing/building, and the 7 agents' own briefs (each
  independently marked "no client data" in its own text). `registry.yaml`
  and `instances.yaml` are fresh/minimal — no client list, no other agents,
  no test-run history from this device.

## What was deliberately left out

- All session history, memories, logs, cron state, cache, attachments
  (`state.db`, `memories/`, `logs/`, `cache/`, `attachments/`, etc.)
- All `.env` files / API keys / tokens — none copied. The new device must
  run its own provider setup (`hermes setup` / `hermes model`) for each
  profile.
- The full `Team-Workspace/Agents` registry (other clients' agents, the
  client list, Work/, teams/*.yaml, backups/) — only the 7 relevant agents
  are listed.
- Any other profile's briefs/data (sales-agent, guardare-invoice-maker,
  client-specific copies, etc.) — out of scope for this export.

## Deploying on the new device

1. Copy `team-workspace-agents/` to `~/Team-Workspace/Agents/` on the new
   machine (or wherever you want the repo to live — then update
   `terminal.cwd` in each profile's `config.yaml` and the `folder:` /
   `brief:` paths in `registry.yaml` to match).
2. For each folder under `profiles/`, create the profile and copy files in:
   ```
   hermes profile create <name> --description "<paste from profile.yaml>"
   cp profiles/<name>/SOUL.md      ~/.hermes/profiles/<name>/SOUL.md
   cp profiles/<name>/config.yaml  ~/.hermes/profiles/<name>/config.yaml
   cp -R profiles/<name>/skills/.  ~/.hermes/profiles/<name>/skills/
   ```
3. Run `hermes setup` / `hermes model` on the new device to wire up
   providers and credentials per profile (none were carried over).
4. Verify with `python3 team-workspace-agents/bin/registry.py status`.
