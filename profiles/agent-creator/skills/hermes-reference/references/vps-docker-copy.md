# Company copy of the method on a Hostinger VPS (Hermes in Docker)

Use when Robi wants the Agent Creator method (scripts, rules, skills) on a second Hermes machine. The Mac (personal) and the VPS (company) stay two separate, unsynced copies: empty registry, no briefs, keys, chat history or memory carried over.

## 1. Where things are (seen on the Hostinger Hermes template, v0.22.5)

- Robi uses Hostinger's web terminal, which is already INSIDE the container: `docker ps` fails there with "Cannot connect to the Docker daemon". Do not ask for Docker commands.
- `HERMES_HOME=/opt/data`, a real mounted disk, so files there persist. Profiles: `/opt/data/profiles`. Dashboard Files page opens `/opt/data`.
- Other profiles and `/opt/data/agent-factory/registry.md` may already exist. They are not ours: read, never touch, and list them as "not yet following the requirements".
- The shell is root, but the gateway runs as user `hermes` (UID 10000). Anything root creates under `/opt/data` can be unreadable to it, so `chown -R hermes:hermes` what we add, and run the scripts as `hermes` (section 4).
- Hermes's own Python has `ruamel.yaml`, not PyYAML, and no pip, uv or yaml in plain `python3`.

## 2. Python with yaml (the scripts import `yaml`)

With Robi's OK (it downloads from PyPI and adds one folder):
`python3 -m venv /opt/data/agents-venv && /opt/data/agents-venv/bin/python -m pip install pyyaml && chown -R hermes:hermes /opt/data/agents-venv`. `python3 -m ensurepip` and `venv` work in Hermes's Python. Ignore the pip cache warning. A Hermes update may break the venv's link to the base Python: recreate with the same command.

## 3. Moving the files

- Beginner route: dashboard Files page, UPLOAD one `.tar.gz` into `/opt/data`, then unpack in the terminal. Warn that every row has a red delete icon.
- Build the archive on the Mac with `COPYFILE_DISABLE=1 tar --exclude='__pycache__' -czf ~/Desktop/agents-bundle.tar.gz Agents/...` run from `~/Team-Workspace`. Without `COPYFILE_DISABLE=1` the archive carries `._*` Mac metadata files; they unpack as junk (also inside `skills/`, which the build script would copy into the profile). Delete them only with Robi's OK: `find /opt/data/Team-Workspace -name '._*' -type f -delete`. The tar lines "Ignoring unknown extended header keyword LIBARCHIVE.xattr..." are harmless.
- Bundle: `bin/{check_agent,create_agent,registry}.py`, `CONVENTIONS.md`, `AGENT-REQUIREMENTS.md`, and the six method skills (agent-lifecycle, agent-requirements, hermes-reference, socratic-discovery, task-changelog, workflow-and-journey-mapping). Not `agentlib.py`, `hire_agent.py`, `new_client.py`, `enable_jira.py` (Mac-only Governor/Jira parts), not briefs, registry, `instances.yaml`, `Work/`. Run a secret scan on the bundle files first (`grep -rEi` for key patterns); the only hit in `check_agent.py` is its own detection regex.
- Unpack: `mkdir -p /opt/data/Team-Workspace && tar -xzf /opt/data/agents-bundle.tar.gz -C /opt/data/Team-Workspace && chown -R hermes:hermes /opt/data/Team-Workspace`.

## 4. Making the scripts find their folders

The scripts compute everything from `$HOME` (`$HOME/Team-Workspace/Agents`, `$HOME/.hermes/profiles`, `$HOME/.hermes/gateway_state.json`, `$HOME/.hermes/logs`) and the builder may not edit them. Use a small home of links, not a link inside the data folder pointing at itself (tools that walk `/opt/data` could loop):

`mkdir -p /opt/data/agents-home/.hermes && ln -s /opt/data/Team-Workspace /opt/data/agents-home/Team-Workspace && ln -s /opt/data/profiles /opt/data/agents-home/.hermes/profiles && ln -s /opt/data/gateway_state.json /opt/data/agents-home/.hermes/gateway_state.json && ln -s /opt/data/logs /opt/data/agents-home/.hermes/logs && chown -R -h hermes:hermes /opt/data/agents-home`

`chown -h` changes the links, not the real folders they point to. Then every script runs as:
`runuser -u hermes -- env HOME=/opt/data/agents-home /opt/data/agents-venv/bin/python /opt/data/Team-Workspace/Agents/bin/<script>.py ...` (the agent itself already runs as `hermes`, so it drops `runuser`).

`registry.py` fails if `registry.yaml` does not exist. Seed an empty one once (no helper rows), then only scripts touch it:
`printf '# Agent registry. Written by Agents/bin/create_agent.py and Agents/bin/registry.py.\n# Do not hand-edit agents[]. Readable copy: REGISTRY.md.\nversion: 1\nclients: {}\nagents: []\n' > registry.yaml`. Check: `registry.py status` printed "0 registered agent(s)" and the existing profiles as not following the requirements.

With `clients: {}` and no `new_client.py`, only team helpers (`placement: team:<Folder>`) can be built. Ask Robi how client folders should work before promising client helpers.

## 5. Model route (check BEFORE the build)

`create_agent.py` hard-codes `provider: anthropic`, `base_url: https://api.anthropic.com` for every new profile. Look at what the server really uses, printing no secrets:
`grep -A4 '^model:' /opt/data/config.yaml; grep -c '^ANTHROPIC_API_KEY=.\+' /opt/data/.env` (a count only).
The company VPS showed `claude-subscription-directsdk-experimental` (a Claude subscription login) and a count of 0. A new profile built by the script would then have no working key. Plan an explicit step after the build, with Robi's OK: compare `hermes -p <slug> config get model.provider` with the default, switch with `hermes -p <slug> config set model.<key> <value>` (never hand-edit `config.yaml`), find out whether the profile needs its own sign-in (Robi does it; you never see it), and prove it with a real answer before saying it works. Whether the subscription route offers the model named in the brief is a stated Unknown until tested.

What the failures looked like on the company VPS, in order (read the real error with `hermes -p <slug> --in <folder> -z "Say hello in five words."; echo $?`; the test scenarios only show "exit code 2"):
1. `Could not find the '<provider>' CLI command '(none configured)'`: the provider is a plugin, and a new profile does not inherit it. Cause shown by `ls $HERMES_HOME/plugins` (has it) versus `<profile>/plugins/` (empty) and `hermes -p <slug> plugins list --plain --no-bundled`. Fix with Robi's OK: `cp -r $HERMES_HOME/plugins/<plugin> <profile>/plugins/` as `hermes`, then `hermes -p <slug> plugins enable <plugin>`. Copying the default's `CLAUDE_*` lines into the profile `.env` did not help and was not needed.
2. Then `HTTP 401 ... Not logged in`: the plugin has no login of its own, it drives the `claude` program, which must be signed in as the user the gateway runs as. Pointing `plugins.entries.<plugin>.settings.claude_config_dir` at the default profile's folders did NOT help (`claude auth status` there said `loggedIn: false`), and `hermes -p <slug> auth add <provider> --type oauth`, which the 401 message suggests, is refused ("plugin ships no auth_handler"). What worked, each step with Robi's OK; he signs in and pastes any code into the server terminal only, never into chat: (a) `runuser -u hermes -- mkdir -p <profile>/claude-config`; (b) `runuser -u hermes -- env HOME=/opt/data CLAUDE_CONFIG_DIR=<profile>/claude-config /opt/data/.local/bin/claude auth login` (prints a browser link); (c) `hermes -p <slug> config set plugins.entries.claude-subscription-directsdk-experimental.settings.claude_config_dir <profile>/claude-config`; (d) prove it with `claude auth status | grep -i loggedIn` run the same way as (b) (expect `true`), then the hello test (exit 0), then `check_agent.py --scenarios`. One folder per profile leaves other profiles' logins untouched. When the 401 comes back, the saved login expired: repeat (b). Do not try to read the gateway's environment (`/proc/<pid>/environ` is denied, even to root) to learn how the default profile signs in.
3. The brief's Unknowns and the SOUL line about switching the route were worth having: ask Robi's OK before each profile-level setting change, and never mark the helper "works" while any scenario fails.
4. Every later `create_agent.py --update` rewrites `model` to the Anthropic route (`write_config` sets it unconditionally; the plugin copy and `claude_config_dir` stay). After each update re-run the three `config set model.provider / model.base_url / model.api_mode` lines from the default's values, then the hello test and the checks.
5. `check_agent.py` WARNs `missing change-log entry` until `Team-Workspace/CHANGELOG.md` exists (the bundle does not carry it). Create it with a title only, as `hermes`. The first check run after that still warns (it reads the logs before it writes its own "Checked" line); the second run passes.

## 6. Guiding a beginner through the server terminal

- One command per step, read-only first, with a plain-words line on what it changes and what output to expect, then ask him to paste the result. Ask his OK before anything that installs, deletes or writes outside what the plan already approved.
- Each pasted terminal log repeats the whole session from the start. Read only the output of the newest command.
- Put every command in one copy-paste block joined with `&&`, and end with a listing so the next step can be checked.
- Stop probing after two failed diagnostics on the same symptom: restate what is proven, what is not, and offer the documented route. Read the plugin's own README (GitHub, `hermes-official-plugins`) before suggesting a fix; a Hermes error message can name a command the plugin does not support.

## 7. Reaching a helper on the VPS (what the server offers)

- The server has the web dashboard only. Its Chat tab is the Hermes terminal screen inside the browser (no chat bubbles, by design); the profile box at top left chooses the helper, and an amber banner names the profile being managed. Palette icon in the header changes theme and font only. Robi finds this "not user friendly" next to his Mac Desktop app: say plainly it is a different front end of the same Hermes. Better front ends are Telegram or Open WebUI, each needing his separate OK. The Desktop app can register a remote server, but he wants the Mac and the company VPS kept separate, so do not propose connecting them.
- Telegram on the VPS: dashboard, Channels, Telegram card, choose the profile first, then "Create with QR" (it writes the token and the allowlist into THAT profile's `.env` and restarts the shared gateway, which affects every profile served; check nobody is mid-task). Allowlist = named numeric IDs only (Robi, then each junior); the helper has a terminal tool. `check_agent.py` verifies Telegram only when the brief's `reach` lists it, so with `reach: app` prove it with a real hello from an allowed phone; a later `--update` with `telegram` in `reach` resets the model route (section 5, item 4).
