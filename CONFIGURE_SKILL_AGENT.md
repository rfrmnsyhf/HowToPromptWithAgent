# CONFIGURE_SKILL_AGENT.md

> **Portable setup instructions for OpenCode CLI agents.**
> Drop this file in your project root, run OpenCode, and the agent will:
> 1. audit the environment and existing config
> 2. preserve valid settings
> 3. install required capabilities (skills, tools, models)
> 4. validate before reporting success.
>
> **No secrets are stored in this file. No API keys are ever printed, copied into configs, or committed to Git.**

---

## 1. Rules the agent must obey

1. **Inspect first, modify second.** Read existing `opencode.json` and any `.opencode/` tree before changing anything.
2. **Preserve valid config.** Never delete fields you didn't add. Never replace a working provider with an unconfigured one.
3. **No duplicate skills.** If a skill folder already exists, do not re-clone. If a plugin is already registered, do not re-add.
4. **Token efficiency wins.** `installed` does NOT mean `loaded into every prompt`. Default behavior: install the capability, then only activate it when the task actually needs it.
5. **No secrets in Git.** If a credential file exists nearby, ensure it is in `.gitignore` before any `git add`.
6. **Validate before reporting done.** `opencode models` must resolve, PLAN/BUILD/EXPLORER models must be reachable, and skill discovery must complete without error.
7. **Stop on contradiction.** If existing config contradicts this guide (and you can't tell which is correct), stop and ask the human.
8. **No machine-specific assumptions.** Never hardcode a model ID, provider name, API key, or local file path. Always audit the current machine first.

---

## 2. Two modes

| Mode | Purpose | Behavior |
|------|---------|----------|
| `PERSONAL` | The author's primary machine. Installs the full arsenal of optional tools. | Clone every repo listed in §6. Most stay in `.cache/`, not in context. |
| `PORTABLE` | A colleague's fresh device. Installs only what this project needs. | Clone only Core skills (§4) and the Required capabilities (§5) actually used by this project. |

**Default = `PORTABLE` if you are reading this for the first time.** Switch to `PERSONAL` only if the human tells you to.

Detect the mode by asking once at the top of the run. Do not guess.

---

## 3. Environment prerequisites

Before doing anything, the agent must verify these tools exist. On Windows PowerShell:

```powershell
# All checks are non-fatal; report missing ones to the human instead of failing.
git --version
node --version
python --version
uv --version                 # optional but recommended
gh --version                 # only if you intend to use the gh CLI flow

# OpenCode CLI
opencode --version
```

If `opencode` is missing, stop. The human must install it from <https://opencode.ai>.

Authentication for providers is handled via:

```powershell
opencode auth login
```

Follow the interactive prompt. **Never** paste an API key into `opencode.json`, into a project file, or into a chat transcript.

---

## 4. Core behavioral skills (always active)

These three skills form the **behavioral layer** of every session. They are loaded at the start of every run. The agent should be able to handle an entire project with just these three plus a small set of selective skills.

| Skill | Repo | Install |
|-------|------|---------|
| **Ponytail** | `DietrichGebert/ponytail` | npm: `npm i -g @dietrichgebert/ponytail`. Then in `opencode.json` add `"plugin": ["@dietrichgebert/ponytail"]`. |
| **I Have ADHD** | `ayghri/i-have-adhd` | `git clone --depth 1 https://github.com/ayghri/i-have-adhd.git <global-cache>/i-have-adhd`. Copy `skills/i-have-adhd/` to `<global-cache>/skills/i-have-adhd/`. Also register the plugin from `.opencode/plugins/i-have-adhd.mjs` in `opencode.json`. |
| **No AI Slop** | `petergyang/no-ai-slop` | `git clone --depth 1 https://github.com/petergyang/no-ai-slop.git <global-cache>/no-ai-slop`. Copy `skills/no-ai-slop/` to `<global-cache>/skills/no-ai-slop/`. |

`<global-cache>` on Windows is `~/.config/opencode/.cache/` (use `$HOME` in PowerShell, not a hardcoded path). The skills directory the agent must populate is `~/.config/opencode/skills/`.

After installing, in the global OpenCode config add:

```json
"skills": {
  "paths": [
    "<global-cache>/skills"
  ]
}
```

Replace `<global-cache>` with the actual path resolved from `$HOME` at runtime.

Verification: `Get-ChildItem "<global-cache>/skills" -Directory` returns the three skill folders.

---

## 5. Required capabilities

**In `PORTABLE` mode**, install only the required capabilities that the current project actually uses. **In `PERSONAL` mode**, install both as the default baseline.

### 5.1 Strix (security testing)

- Repo: `usestrix/strix`
- Interface: Python CLI + 9 native agent skills under `strix/skills/`
- Install:

  ```powershell
  pip install strix-agent
  # or, preferred for isolation:
  uv tool install strix-agent
  ```

- Verify: `strix --help` prints the help block (the binary is `strix.exe` under Python's `Scripts\`; ensure that directory is on `PATH` for the current user).
- Skills: the 9 skills in `strix/skills/` are **NOT** copied to global skills. They are documented here so the agent can pull them on demand if the project explicitly invokes security testing.

### 5.2 OpenSEO

- Repo: `every-app/open-seo`
- Interface: full self-hosted app (Docker) + a small `.opencode/` integration (one command file, one Context7 MCP config)
- **Status in this guide:** `source cached / runtime not enabled`. The OpenSEO **app** is not a single-process CLI; the only honest way to "install" it is either (a) clone the source for reference, or (b) run the full Docker self-host stack. The agent **must not** claim OpenSEO is "ready to use" after a bare `git clone`.
- Minimum install (`PORTABLE`): just clone the repo to `<global-cache>/open-seo/`. Do not register any of its components as OpenCode skills or plugins.
- Full install (`PERSONAL`, only if a real SEO audit is in scope):
  ```powershell
  # Self-host the OpenSEO app
  docker compose -f <global-cache>/open-seo/compose.yaml up -d
  ```
  The agent must wait for the stack to report healthy before claiming OpenSEO is operational.
- Verify cached source: `Test-Path "<global-cache>/open-seo/.opencode/opencode.jsonc"` returns `True`.
- Verify runtime: `(Invoke-WebRequest -UseBasicParsing http://localhost:<port>/health).StatusCode -eq 200` (replace port with whatever `compose.yaml` exposes; do not guess).

---

## 6. Selective agent skills

**In `PORTABLE` mode**, install only what maps to the current project stack. **In `PERSONAL` mode**, install the canonical nine below.

Source: `addyosmani/agent-skills`. All skills live under `skills/<name>/SKILL.md` and follow the same native format that OpenCode reads.

Canonical nine (copy each to `<global-cache>/skills/<name>/`):

1. `planning-and-task-breakdown`
2. `incremental-implementation`
3. `test-driven-development`
4. `debugging-and-error-recovery`
5. `code-review-and-quality`
6. `security-and-hardening`
7. `code-simplification`
8. `documentation-and-adrs`
9. `git-workflow-and-versioning`

Agentic Awesome Skills (`sickn33/agentic-awesome-skills`) is **OPTIONAL** and capped at 5 skills. Always run `audit` and `--dry-run` first. If a skill duplicates one from `agent-skills`, prefer the `agent-skills` version.

---

## 7. Optional on-demand arsenal (PERSONAL mode only)

Each row is an installed capability, not a context slot. Clone to `<global-cache>/<name>/` and stop there. Do not register as a skill or plugin unless the project actually needs it at runtime.

| Capability | Repo | Use it when |
|------------|------|-------------|
| Browser Use | `browser-use/browser-use` | a session needs a real browser |
| Firecrawl | `firecrawl/firecrawl` | a research task needs clean scraped content |
| Crawl4AI | `unclecode/crawl4ai` | same, but Python-first |
| Maxun | `getmaxun/maxun` | no-code scraping is required |
| Book To Skill | `virgiliojr94/book-to-skill` | converting a PDF into a new skill |
| Strix skills | `usestrix/strix/skills/*` | the project triggers a security scan |
| CrewAI | `crewAIInc/crewAI` | orchestrating multiple roles |
| OpenHands | `All-Hands-AI/OpenHands` | reference architecture (do not install side-by-side) |
| Coolify | `coollabsio/coolify` | self-host deploy for any service |
| Open WebUI | `open-webui/open-webui` | you want a local chat UI |
| AnythingLLM | `Mintplex-Labs/anything-llm` | a knowledge-base front end |
| Open Notebook | `lfnovo/open-notebook` | research notebook |
| Pipecat | `pipecat-ai/pipecat` | real-time voice pipelines |
| Dify | `langgenius/dify` | a full LLM app platform |
| OmniRoute | `diegosouzapw/OmniRoute` | LLM routing layer |
| Kimi K3 in C | `FareedKhan-dev/kimi-k3-in-c` | reference C port, not a tool |
| AI Job Search | `MadsLorentzen/ai-job-search` | reference only |
| Open Generative AI | `Anil-matcha/Open-Generative-AI` | reference only |

---

## 8. Model configuration (provider/model-agnostic)

### Conceptual roles

OpenCode distinguishes three agent roles. What each role **does** is fixed; which **model** powers it is up to the current machine.

| Role | Purpose |
|------|---------|
| `plan` | Reads the task, drafts a plan, asks clarifying questions. Reasoning-heavy. |
| `build` | Writes code, runs tools, applies patches. Action-heavy. |
| `explorer` | (Subagent) Searches the codebase, gathers context, summarizes. Read-only. |

**No concrete model ID is prescribed here.** The agent must select a model for each role from whatever the current machine actually exposes.

### Selection procedure

The agent must perform these steps in order, and must not skip any:

1. **Audit the live catalog:**
   ```powershell
   opencode models
   ```
   This is the source of truth for what is available right now. Ignore cached state files, recent lists, or assumptions from the machine that produced this guide.

2. **List what is actually there.** Group the output by provider prefix (`<provider>/<model>`). Note which providers have stored auth (check `opencode auth list` or equivalent).

3. **Choose one model per role.** Rules of thumb:
   - Prefer providers the human has already authenticated for.
   - For `plan`: a reasoning-capable model.
   - For `build`: a code-capable model.
   - For `explorer`: a model good at reading and summarizing, or reuse `plan` if no dedicated model fits.
   - Do not invent a model ID. If `opencode models` does not list it, it is not available.
   - Do not assume a model exists because it was used on some other machine.

4. **Verify each chosen ID resolves:**
   ```powershell
   opencode models | Select-String "<chosen-id-substring>"
   ```
   Each line must be non-empty.

5. **If the project already has `opencode.json` with working agent models, preserve them.** Only patch what is missing or broken.

### Fallback policy

**This guide does not define a fallback layer in OpenCode.** Reasons:

- OpenCode's `agent.<role>.model` is a single string; there is no native fallback field.
- Any routing/fallback should be handled by a provider-side layer (e.g. a router that exposes multiple upstream models under one provider prefix). The agent's job is to point at that provider; the provider's job is to handle fallbacks.
- Hardcoding a second model as a "fallback" inside `opencode.json` creates a silent double-routing layer that is hard to debug.

If the human is using a router-style provider, configure one model per role and let the router handle the rest. If the human wants explicit per-model fallback inside OpenCode, ask them which model to use and patch it in as a second `agent.<role>.model` entry only after they confirm.

### Patch the global config

Append to the global OpenCode config (do not replace the file):

```json
"agent": {
  "plan":     { "model": "<verified-id-1>" },
  "build":    { "model": "<verified-id-2>" },
  "explorer": { "mode": "subagent", "model": "<verified-id-3>" }
}
```

Replace `<verified-id-n>` with model IDs the agent actually saw in `opencode models` on the current machine. If a project has its own `opencode.json`, repeat the same patch there.

### Authentication

Authentication is per-provider and is the human's responsibility. Use:

```powershell
opencode auth login
```

Follow the interactive prompt for each provider the agent intends to use. **Never** paste an API key into `opencode.json`, into a project file, or into a chat transcript. The agent must only reference the existence of stored auth, not its contents.

---

## 9. Git safety

If the project root is a Git repository, ensure any obvious secret file is gitignored **before** any commit:

```powershell
# Check if the project root is a Git repo
git -C <project-root> rev-parse --is-inside-work-tree

# Add common credential filename patterns to .gitignore (do not hardcode one name)
$patterns = @("Apikey *.txt", "*credential*.txt", "*.env", "secrets.*")
foreach ($p in $patterns) {
  Add-Content -Path "<project-root>/.gitignore" -Value $p
}

# Verify the patterns are actually ignored
git -C <project-root> check-ignore -v "Apikey *.txt"
```

The last command must report the file as ignored. If it does not, stop and tell the human.

---

## 10. Validation checklist

The agent must complete every item before reporting `DONE`:

- [ ] Mode chosen (`PERSONAL` or `PORTABLE`); confirmed with the human.
- [ ] OpenCode CLI installed and on PATH.
- [ ] Three Core skills present under `<global-cache>/skills/`.
- [ ] Required capabilities installed (or explicitly skipped) per the chosen mode.
- [ ] Selective skills installed (or explicitly skipped) per the chosen mode.
- [ ] `opencode models` was run on the **current** machine, and the output is the basis for model selection.
- [ ] Each chosen `plan` / `build` / `explorer` model ID was verified to appear in `opencode models` output.
- [ ] Global `opencode.json` contains `agent.plan.model`, `agent.build.model`, and (if used) `agent.explorer.model` set to the verified IDs.
- [ ] Any project-level `opencode.json` with outdated or broken agent models is patched with the same verified IDs.
- [ ] No fallback layer was invented inside `opencode.json`. If a router provider is in use, fallback is its responsibility.
- [ ] Any working custom provider (e.g. a router) is preserved if it was present.
- [ ] Any working MCP server (e.g. OpenDesign) is preserved if it was present.
- [ ] Obvious secret file patterns are gitignored in every Git repo that can see them.
- [ ] No secret value appears in any output the agent produced.

If any item is unchecked, report exactly which item failed and what you observed. Do not report `DONE` until every box is filled.

---

## 11. What this guide deliberately does not do

- It does not delete or rewrite `opencode.json` from scratch. Preservation is the default.
- It does not install every skill from the catalogs. Selective install only.
- It does not bake a working provider out of the box. Auth is the human's responsibility.
- It does not invent model IDs. If a model ID is not in `opencode models` on the current machine, the agent stops and asks.
- It does not hardcode a specific provider, model, or API key. Every machine picks its own.
- It does not define a fallback layer inside OpenCode. Routing/fallback belongs to the provider or router, not to OpenCode's `agent.<role>.model` field.

---

## 12. Quick command summary (PowerShell)

```powershell
# 1. Verify environment
git --version; node --version; python --version; opencode --version

# 2. Clone sources into cache
$cache = "$HOME/.config/opencode/.cache"
New-Item -ItemType Directory -Path $cache -Force | Out-Null

git clone --depth 1 https://github.com/ayghri/i-have-adhd.git  "$cache/i-have-adhd"
git clone --depth 1 https://github.com/petergyang/no-ai-slop.git "$cache/no-ai-slop"
git clone --depth 1 https://github.com/addyosmani/agent-skills.git "$cache/agent-skills"
git clone --depth 1 https://github.com/usestrix/strix.git "$cache/strix"
git clone --depth 1 https://github.com/every-app/open-seo.git "$cache/open-seo"

# 3. Populate global skills
$skills = "$HOME/.config/opencode/skills"
New-Item -ItemType Directory -Path $skills -Force | Out-Null

Copy-Item "$cache/i-have-adhd/skills/i-have-adhd" $skills/i-have-adhd -Recurse -Force
Copy-Item "$cache/no-ai-slop/skills/no-ai-slop" $skills/no-ai-slop -Recurse -Force

$sel = @(
  "planning-and-task-breakdown","incremental-implementation","test-driven-development",
  "debugging-and-error-recovery","code-review-and-quality","security-and-hardening",
  "code-simplification","documentation-and-adrs","git-workflow-and-versioning"
)
foreach ($s in $sel) {
  Copy-Item "$cache/agent-skills/skills/$s" "$skills/$s" -Recurse -Force
}

# 4. Install Strix CLI
pip install strix-agent

# 5. Audit the live model catalog (this is the source of truth)
opencode models

# 6. Pick one model per role from the output above, verify each, then patch opencode.json
opencode models | Select-String "<your-plan-id>"
opencode models | Select-String "<your-build-id>"
opencode models | Select-String "<your-explorer-id>"

# 7. Authenticate (interactive, per provider)
opencode auth login

# 8. Validate
opencode --model <your-plan-id>    # confirm plan agent can be reached
opencode --model <your-build-id>   # confirm build agent can be reached
```

End of guide.
