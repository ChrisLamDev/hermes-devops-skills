---
name: social-media-account-registration
description: Social media account setup guide for AI/developer platforms. Agent prepares all content (bio, avatar, posts) for user to copy-paste, since automation is blocked by Cloudflare/CAPTCHA/React shadow DOM. Supports X/Twitter, Medium, Dev.to, Reddit, LinkedIn.
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [social-media, registration, signup, accounts, content-planning]
    related_skills: [github-demo-recording, reddit-post-launch-workflow]
    tokens:
      scan: 40
      load: 180
      category: tool
---

# Social Media Account Registration Skill

Agent prepares all content for social media account setup. User does the actual form filling because automation is universally blocked.

## Core Insight (Critical — Learned from hard experience)

**Most platforms block browser automation, but NOT all.** Hacker News can be fully automated with Cua Driver.

**Known exceptions that WORK with Cua:**
| Platform | What works | Caveat |
|----------|-----------|--------|
| **Hacker News** | ✅ Full registration + Show HN submission | No CAPTCHA, no email, just username+password. Title limited to 80 chars. |
| **Mastodon** (generic) | ✅ Form filling | Needs email verification link click (human step) |

**Known blocks (the rest):**
This was learned through ~30 minutes of failed attempts across X, Medium, GitHub, and Reddit — every single one blocked.

**Additional hard-earned lesson: user's bio needs to be future-proof.** 
When writing bios for GitHub/Medium, don't tie the bio to a single project (e.g. "Creator of Hermes Core Skills"). Use a broader, evergreen description like "Building useful tools for AI agents and developers. Open source." that won't become outdated when new projects launch. Each platform may need a slightly different version — X needs specificity (what are you doing NOW), GitHub needs generality (who ARE you), Medium needs writing-related angle.

| Platform | Block Type | What happens |
|----------|-----------|-------------|
| X/Twitter | React shadow DOM + bot detection | Programmatic clicks ignored, blank screen after Google OAuth |
| Medium | Cloudflare | "Performing security verification" — page never loads |
| Reddit | Cloudflare + CAPTCHA | "You've been blocked by network security" |
| Dev.to | Cloudflare | Same as Medium |

All platforms also require SMS/email verification codes or CAPTCHA — human-only steps that cannot be bypassed even with semi-automation.

## Correct Workflow

```
Agent prepares:                    User does:
┌──────────────────────┐          ┌──────────────────┐
│ • Pre-written bio    │   ──→   │ • Go to signup URL │
│ • Avatar file ready  │   ──→   │ • Upload avatar    │
│ • First post content │   ──→   │ • Copy-paste bio   │
│ • Thread templates   │   ──→   │ • Copy-paste posts │
│ • Content calendar   │   ──→   │ • Solve CAPTCHA    │
│ • Unified identity   │   ──→   │ • SMS verify       │
└──────────────────────┘          │ • Enter password   │
                                  └──────────────────┘
```

## What Agent Can Do (✅)

1. **Prepare unified identity** — same display name, username, avatar across all platforms
2. **Write bio** — tech tone, platform-optimized length
3. **Write first post/thread/article** — ready to copy-paste
4. **Create avatar** — generate geometric SVG/PNG logo (CL monogram with vibrant gradient)
5. **Create content calendar** — Week 1 daily posting plan with templates
6. **Provide registration links** — direct signup URLs for each platform
7. **Archive credentials** — never store passwords, tell user to remember them

## What Agent Cannot Do (❌)

- ❌ Fill login/signup forms (React shadow DOM ignores programmatic events)
- ❌ Solve CAPTCHA challenges
- ❌ Receive SMS verification codes
- ❌ Click Google/Apple OAuth consent buttons
- ❌ Complete email verification links
- ❌ Log in to existing accounts (all blocked)
- ❌ Post tweets/articles (needs API key or manual action)

## Platform-by-Platform Guide

### X/Twitter
- Signup: `x.com/signup`
- Email + phone (SMS verify required)
- Settings: `x.com/settings/profile`
- Character limit: 280 (free) / 4000 (Premium)
- Bio length: 160 chars max

### Medium
- Signup: `medium.com` → "Sign in with Google" (best path)
- Settings: `medium.com/me/settings`
- New story: `medium.com/new-story` or from profile menu → "Write a story"
- Cloudflare blocks all automation — user must navigate manually

### Dev.to
- Signup: `dev.to/enter` → "Sign in with GitHub"
- Profile: `dev.to/settings`
- New post: `dev.to/new`
- Cross-post Medium articles (same content works)

### Reddit
- Signup: `reddit.com/register`
- Email + CAPTCHA required
- Bot detection very aggressive — may need multiple attempts
- Recommended subreddits: r/cursor, r/ClaudeAI, r/machinelearning, r/artificial

### LinkedIn
- Signup: `linkedin.com/signup`
- Email or phone + CAPTCHA
- More professional bio tone
- Optional — lower priority

## Content Preparation Templates

### Bio Versions by Platform (Critical — learned from user feedback)
| Platform | Bio | Why |
|----------|-----|-----|
| GitHub | Building useful tools for AI agents and developers. Open source. | Evergreen, not tied to any single project |
| X/Twitter | Building tools for AI agents. Creator of Hermes Core Skills. Open source. | Needs specificity — people scan and need to know your thing |
| Medium | Building tools for AI agents & developers. Writing about open source, AI agents, and developer workflows. | Writing platform, signals you'll produce content |
| Dev.to | AI agent tooling & open source. Building Hermes Core Skills. | Developer community, direct is fine |
| Reddit | (leave blank) | Reddit doesn't prioritize bios |
| LinkedIn | Building AI agent tools at Nous Research. Creator of Hermes Core Skills. Open source contributor. | Professional tone, full context |

**⚠️ Never lock your bio to a single project.** User specifically corrected this: "如果我以後再發佈新 project 咪唔相符". Use general descriptions like "building tools for AI agents" that scale across multiple projects.

### Unified Identity

| Field | Value |
|-------|-------|
| Display Name | Chris Lam |
| Username | ChrisLamDev |
| Avatar | Geometric CL monogram (orange-red C + green-teal L on vibrant gradient bg) |
| Bio (full) | Building AI agent tools. Creator of Hermes Core Skills — 25 executable skills for debugging, planning & token efficiency. |
| Bio (short, X) | Building AI agent tools. Creator of Hermes Core Skills — 25 executable skills for debugging, planning & token efficiency. Open source. |

### First Post Template (X, 280 chars)
```
Just dropped Hermes Core Skills — 25 executable AI agent skills for debugging, planning, token efficiency & security.

🚀 `cp -r skills/* ~/.claude/skills/`
⭐ github.com/ChrisLamDev/hermes-core-skills

Built for Claude Code, Codex, Cursor & Hermes. #AIAgents #OpenSource
```

### Thread Template (X)
```
1/N Let's talk about AI agent reliability.

Most people think better prompts = better outputs.

But there's a better way: skills.

2/N A prompt is a one-shot instruction. A skill is a repeatable workflow with:
• Triggers (when to use it)
• Steps (how to execute)
• Checks (how to verify)

3/N Hermes Core Skills turns this into executable files:
skills/debugging/SKILL.md
skills/planning/SKILL.md
skills/review/SKILL.md

4/N One command to install all 25:
cp -r skills/* ~/.claude/skills/

Open source, MIT licensed.
github.com/ChrisLamDev/hermes-core-skills
```

## Avatar Generation (Geometric SVG)

Use Python PIL to generate a CL monogram avatar. **Expect 5-7 iterations** — user feedback loop is essential:

| Iteration | Feedback | Fix |
|-----------|----------|-----|
| v1-v2 | "C and L not visible" | Arc/path rendering issues, switch to solid blocks |
| v3 | "Too dark/somber" | Black bg → vibrant gradient (red-orange-purple) |
| v4 | "Not sharp enough" | Boost saturation, add high-contrast accent dots |
| v5 | "Not harmonious" | Reduce saturation, blend colors smoothly |
| v6 | "No tech feel" | Add grid overlay, outer glow ring, neon-style accents |
| v7 | ✅ "Good" | Solid block letters (orange C + green L), vibrant bg, subtle glow |

**Final approach that worked:**

1. Background: vibrant gradient (e.g., deep red → vibrant purple or warm orange → magenta)
2. Letter C: block or thick arc, orange/red gradient (contrasts with bg)
3. Letter L: block style, green/teal gradient
4. Accent dots around edge for tech feel
5. Circular mask with outer ring glow
6. Output: 1024x1024 PNG (24-100KB), 400x400 thumbnail

## Content Calendar Template

### Week 1 Schedule
| Day | Platform | Content Type |
|-----|----------|-------------|
| Day 1 | X + Medium | Launch post + full article |
| Day 2 | X | Value post (token efficiency) |
| Day 3 | X | Tutorial (debugging workflow) |
| Day 4 | Reddit | Community post (what I learned) |
| Day 5 | X | Show & tell (most popular skill) |
| Day 6 | X | GitHub star update |
| Day 7 | X | Weekly roundup |

## Pitfalls

1. Do NOT waste time attempting browser automation — it won't work. Skip directly to content preparation.
2. X compose page URL is `x.com/compose/post` — 404 if not logged in, user must be authenticated first.
3. Medium `medium.com/new` may 404 — use "Write a story" from profile menu instead.
4. Reddit may block even the homepage — navigating to `reddit.com` first then clicking "Sign Up" can work.
5. Always have 2-3 backup usernames ready in case primary is taken.
6. Never store passwords in skill files or memory — tell user to save them.
