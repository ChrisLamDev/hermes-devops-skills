# hermes-devops-skills

<div align="center">

[![GitHub stars](https://img.shields.io/github/stars/ChrisLamDev/hermes-devops-skills?style=for-the-badge&logo=github)](https://github.com/ChrisLamDev/hermes-devops-skills/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/ChrisLamDev/hermes-devops-skills?style=for-the-badge&logo=github)](https://github.com/ChrisLamDev/hermes-devops-skills/forks)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![Skills](https://img.shields.io/badge/Skills-25-8A2BE2?style=for-the-badge)](skills/)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-8A2BE2?style=for-the-badge&logo=anthropic)](https://claude.ai)
[![OpenAI Codex](https://img.shields.io/badge/OpenAI%20Codex-ready-8A2BE2?style=for-the-badge&logo=openai)](https://openai.com)
[![Cursor](https://img.shields.io/badge/Cursor-ready-8A2BE2?style=for-the-badge&logo=cursor)](https://cursor.sh)
[![Hermes Agent](https://img.shields.io/badge/Hermes%20Agent-ready-8A2BE2?style=for-the-badge&logo=python)](https://github.com/NousResearch/hermes-agent)

</div>

**14 battle-tested macOS & GitHub DevOps skills for AI coding agents.**

Your AI agent spends 30% of its time reinventing infrastructure. These skills encode hard-won DevOps patterns so your agent ships faster, debugs smarter, and stops making the same mistakes twice.

## You've been here before:

- 🔄 Brew upgrade broke your Node tooling again? `fix-brew-node-dylib-mismatch`
- 🎥 Need a demo GIF but hate manual recording? `github-demo-recording`
- 🚀 Releasing open-source without a playbook? `github-exposure-standard`
- 🔍 Finding the right GitHub project among 10,000 clones? `github-project-discovery-hybrid`
- 📦 Installing a new tool and forgetting to verify it works? `install-tool-with-verification`
- 📱 Building iOS apps from CLI without opening Xcode? `ios-app-build-automation`
- 💾 Backup overwriting instead of accumulating cruft? `macos-backup-overwrite-only`
- 🐍 Deploying a Python tool from GitHub on Apple Silicon? `macos-deploy-python-tool-from-github`
- 🧠 macOS RAM full — swap thrashing, what's eating your memory? `macos-ram-swap-analysis`
- 🐳 Docker + local LLM setup without 20 tabs open? `open-notebook-docker-ollama-setup`
- 📣 First Reddit post gets AutoMod-blocked? `reddit-post-launch-workflow`
- 🔁 Agent stuck in a loop burning tokens? `self-regulation-brake-system`
- 📝 Setting up social accounts across 5 platforms? `social-media-account-registration`
- 🔔 Need event-driven agent activation? `webhook-subscriptions`

## Quick Start

```bash
# Clone the repo
git clone https://github.com/chrislamlayer1-gif/hermes-devops-skills.git

# Copy skills to your agent's skill directory
# For Hermes Agent:
cp -r skills/* ~/.hermes/skills/

# For Claude Code:
cp -r skills/* ~/.claude/skills/

# For Cursor:
# Add the skills/ directory in Cursor settings → Custom AI Agent → Skills Directory
```

## Highlight Skills

| Skill | Pain Point | One-liner |
|-------|-----------|-----------|
| `self-regulation-brake-system` | Agent loops endlessly, burning tokens | Auto-stops after 3 failures, prevents infinite retry |
| `github-exposure-standard` | 0-star repos nobody discovers | Release playbook with proven social media + README templates |
| `ios-app-build-automation` | Xcode GUI slow for CI | Full XcodeGen + xcodebuild pipeline, zero GUI |
| `macos-ram-swap-analysis` | "System application memory insufficient" | Identifies RAM shortage vs swap congestion in one command |
| `reddit-post-launch-workflow` | New accounts get shadowbanned | AutoMod-aware posting with karma-farming strategy |
| `github-demo-recording` | Manually recording demos is tedious | Auto-detect + record terminal/GUI demos for any repo |

## What's Inside

### 🐙 GitHub Automation (5)
- `github-demo-recording` — Auto-detect environment, record terminal/GUI demos
- `github-exposure-standard` — Release playbook for discoverable, trustworthy repos
- `github-project-discovery-hybrid` — Multi-angle GitHub project search & evaluation
- `reddit-post-launch-workflow` — Full Reddit launch with flair, karma, and AutoMod handling
- `webhook-subscriptions` — Event-driven agent activation via webhooks

### ⚙️ macOS Operations (5)
- `macos-backup-overwrite-only` — Daily backup that overwrites instead of accumulating
- `macos-deploy-python-tool-from-github` — Deploy Python tools on Apple Silicon
- `macos-ram-swap-analysis` — Diagnose memory pressure and swap congestion
- `install-tool-with-verification` — Install + verify + health-check in one command
- `fix-brew-node-dylib-mismatch` — Fix brew-upgrade broken Node browser tooling

### 🛠️ Developer Tooling (4)
- `ios-app-build-automation` — XcodeGen + xcodebuild CI pipeline (zero GUI)
- `open-notebook-docker-ollama-setup` — Open Notebook + local Ollama via Docker
- `self-regulation-brake-system` — Agent self-regulation with pressure escalation
- `social-media-account-registration` — Social account setup with content pre-generation

## Platform Compatibility

These skills work with any AI agent that supports skill/memory injection:

- **Hermes Agent** (native) — full support
- **Claude Code** — drop `skills/*` into `~/.claude/skills/`
- **Cursor** — add via Custom AI Agent settings
- **Codex CLI** — reference via `-p` flag

## How to Contribute

1. Fork the repo
2. Add your skill as `skills/your-skill-name/SKILL.md`
3. Open a PR with description of the DevOps pain point it solves

## License

MIT — use freely, improve openly.
