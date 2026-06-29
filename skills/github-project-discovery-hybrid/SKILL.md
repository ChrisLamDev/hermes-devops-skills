---
name: github-project-discovery-hybrid
description: Multi-angle GitHub project discovery using browser + GitHub API + trending, with practical clone/install/absorption workflow
version: 2.1.0
author: Hermes Agent
tags: [github, discovery, research, tools, devops, git-clone]
  triggers:
    - discover project
    - find project
    - github explore
    - trending

---

# GitHub Project Discovery & Absorption Hybrid Workflow

## Overview

No AI halluncination about whether a project is good or not — use the browser to actually visit GitHub to check trending, topics, API search, then clone + verify via README.

## Step 1: Multi-Angle Search

Run three search vectors simultaneously:

### Vector A: GitHub Trending (This Week)
```bash
browser_navigate(url="https://github.com/trending?since=weekly")
browser_snapshot()
```
Review this week's hottest repos, filter for ones relevant to our stack.

### Vector B: GitHub Topics
Browse topic pages one by one:
```
https://github.com/topics/<topic-name>
```
Common topics: `wechat-miniprogram`, `cocos-creator`, `ollama`, `macos-automation`, `local-llm`

### Vector C: GitHub API Search (curl)
Use the API for precise searches in specific domains:
```bash
curl -s "https://api.github.com/search/repositories?q=<keyword>&sort=stars&order=desc&per_page=10"
```

## Step 2: Filter + Verify

Don't trust descriptions blindly — read each project's README:

1. **Active?** — Recent commits? (within 6 months)
2. **Practical?** — Can it be installed and used? Any dependency issues?
3. **Relevant?** — Suitable for our macOS M1 / 16GB RAM / WeChat Mini Program / Cocos Creator environment?

## Step 3: Take Action

For each project, decide one of the following:
- **Clone + Absorb** (e.g. agent-skills) — clone and read source code, patch into existing skills
- **Install + Test** (e.g. SkillSpector) — install and immediately test run to verify it works
- **Clone but not immediately usable** (e.g. ccc-devtools) — acknowledge limitations, document reasons

## Step 4: Absorption (Most Important)

Don't stop at cloning — do the following:
1. Read each skill/component's README
2. Compare against existing Hermes skills to identify gaps
3. Use `skill_manage(action='patch')` / `skill_manage(action='create')` to integrate

## Pitfalls

- **Don't delegate_task for searching** — subagents tend to be lazy and recommend mainstream projects (PyTorch, LangChain). Do your own browser research.
- **Check project structure before installing tools** — ccc-devtools is Cocos Creator-specific, won't work with WeChat Mini Program structure
- **Token consumption management** — browser navigation and curl API search are cheap, but cloning large repos and reading long READMEs burn context. Stop if no progress in 5 minutes.
- **Don't re-report installed projects** — use memory to track what's already installed
- **3 failures / 5 min no progress → use brake system to ask Chris**
- **Absorption is the heaviest work** — cloning is just the start. Real time is spent on: reading each README → comparing with existing skills → deciding patch/create → writing content. Reserve sufficient token budget.
- **Don't over-absorb at once** — a project typically has 20+ components. Pick the 3-5 highest-value ones to absorb, record the rest for future reference.
