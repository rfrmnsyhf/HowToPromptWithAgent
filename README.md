# OpenCode Skill Agent Config

Portable configuration guide for setting up **OpenCode CLI agents** with reusable skills, tools, and project-aware model selection.

The goal is simple:

> Drop `CONFIGURE_SKILL_AGENT.md` into a project, let the agent audit the environment, and configure only what is actually needed.

No hardcoded API keys.  
No machine-specific paths.  
No hardcoded model IDs.  
No OpenCode-level fallback chains.

---

## What is this?

`CONFIGURE_SKILL_AGENT.md` is a portable setup guide for OpenCode-based AI coding agents.

It tells an agent how to:

- Inspect the current environment before making changes
- Preserve existing OpenCode configuration
- Install reusable agent skills
- Configure optional development capabilities
- Select models based on the models actually available on the current machine
- Keep PLAN, BUILD, and EXPLORER roles separated
- Avoid unnecessary dependencies and configuration
- Protect credentials from being committed to Git
- Validate the resulting setup before declaring it complete

The configuration is designed to work across different machines instead of assuming that every computer has the same providers, models, paths, or credentials.

---

## Design Principles

### Inspect first, modify second

The agent must understand the existing environment and configuration before changing anything.

### Smallest diff that works

Avoid unnecessary files, dependencies, abstractions, and configuration.

If the existing setup already works, don't rebuild it just because humans enjoy touching things that aren't broken.

### No hardcoded models

The guide does **not** prescribe a specific model.

The agent checks:

```powershell
opencode models
