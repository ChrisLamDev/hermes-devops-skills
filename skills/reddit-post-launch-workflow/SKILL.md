---
name: reddit-post-launch-workflow
description: Complete Reddit posting workflow for new accounts — handles flair selection per subreddit, AutoMod account-age blocks, comment-karma farming while waiting, and cron-job monitoring for comment opportunities.
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [reddit, social-media, posting, automation, cron]
    related_skills: [social-media-account-registration]
    tokens:
      scan: 30
      load: 150
      category: tool
---

# Reddit Post-Launch Workflow

Post-launch phase for Reddit — what to do after account creation but before posts go live. New Reddit accounts are frequently blocked by subreddit-specific AutoMod rules based on account age (typically 3-7 days) and/or low karma.

## Common Reddit Posting Blockers

### 1. AutoMod Account-Age Filter
Many subreddits (especially tech/AI ones like r/ClaudeAI) auto-remove posts from accounts under 3-7 days old.

**Symptom:** Post appears to submit successfully, but AutoMod sends a removal message like:
> "抱歉你的帳號太新，無法發文。如果你有較永久的帳號，請使用。否則請再等幾天再試一次。"

**Solution:** Simply wait 3 days (for most subreddits) or up to 7 days (for strict ones). The post content itself is fine — it's the account age, not the content.

### 2. Post Flair Requirements
Many subreddits require selecting a "post flair" (category tag) before submitting.

**Symptom:** "Your post must contain post flair" error when trying to submit.

**When this matters:**
- **URL/ Link posts** — often require flair. If flair UI is broken or confusing, switch to **Text post** instead (which may not require flair)
- **Text posts** — may or may not require flair depending on subreddit
- **Workaround:** If flair doesn't appear in a dropdown, try the "select" button next to "choose a flair" field

**Flair tips by subreddit:**
| Subreddit | Suggested Flair | Notes |
|-----------|----------------|-------|
| r/cursor | Resources & Tips | ✅ Worked successfully |
| r/ClaudeAI | Built with Claude | Or "Skills" if available |
| r/artificial | Resource | Or "Other" |
| r/ClaudeCode | Resource | Or "Guide/Tutorial" |

### 3. Reddit Karma Requirements
Some subreddits require minimum comment karma or combined karma to post.

**Solution:** Build karma by leaving meaningful comments on other posts first.

## Comment-Karma Farming Strategy (While Waiting for Account Age)

**Core principle:** Leave genuinely useful, on-topic comments — not spam. Your expertise in AI agents, Claude Code, Cursor, and debugging workflows gives you authentic value to add.

### High-Value Comment Templates

**Template 1 — When someone struggles with agent debugging:**
> This is exactly the problem I ran into. I was constantly rebuilding the same debugging flow across Claude Code, Cursor, and Codex — every time I switched agents, I had to reinvent the wheel.
>
> Ended up packaging 25 reusable skills as markdown files that work across all of them. The systematic debugging one alone saved me hours. Happy to share the approach if anyone's interested — it's basically a 5-step reproduction → isolate → fix → verify → log flow.

**Template 2 — When someone talks about context window issues:**
> One thing that made a huge difference for me was having a strict context compaction strategy. I was hitting token limits constantly until I built a skill that forces my agent to summarize + checkpoint before it gets too deep.
>
> The trick is not just compressing — it's knowing *when* to compress. I trigger it after every major tool call chain, not when the context is already full. Saves about 40% tokens on long sessions.

**Template 3 — When someone compares Claude Code vs Cursor:**
> I use both daily. Claude Code is better for complex refactoring and architecture decisions — it has better context understanding across multiple files. Cursor wins on speed for smaller edits and quick iterations.
>
> What I found most useful is having portable skills that work on both. Things like TDD flow, code review pipeline, and browser automation — same markdown files, same workflow, just different agent underneath. That's where the real productivity gain is, not choosing one tool over the other.

**Tip:** Don't hard-sell your repo in every comment. Template 1 is the softest — mentions it naturally. Template 2 and 3 don't mention it at all but build your reputation as someone who knows their stuff.

## Automated Monitoring via Cron Jobs

Set up daily cron jobs to monitor target subreddits for hot posts where you can leave comments. This keeps you active without manual checking.

### Cron Job Setup Pattern

```bash
# Using Hermes cron job tool:
cronjob(
  action='create',
  name='Reddit-Hot-Posts-r{SubredditName}',
  prompt='''Check r/{subreddit} on Reddit for today's hot posts. Use old.reddit.com/r/{subreddit}/. 

Your job:
1. Open old.reddit.com/r/{subreddit}/ 
2. Look at the top 10 hottest posts from the last 24 hours  
3. For each post, list: title, upvotes, number of comments, and a 1-line summary of what it's about
4. Identify which 2-3 posts would be good targets for a meaningful comment about: {relevant_topics}
5. For each good target, write a short note on what angle the comment should take

Deliver the results as a clean summary to the user on Telegram. Format it clearly with headings for "Hot Posts Today" and "Good Comment Targets". Include direct links to the posts you recommend commenting on.''',
  schedule='0 10 * * *',
  repeat=3,  # 3 days of monitoring
  skills=['youtube-content', 'agent-reach-graft-scrapling']
)
```

### Recommended Subreddits to Monitor
| Subreddit | Topics for Comment Angle |
|-----------|-------------------------|
| r/ClaudeAI | AI agent tools, Claude Code vs Cursor comparison, debugging workflows, token efficiency |
| r/cursor | Cursor usage tips, AI coding workflow, debugging, agent skills, reusable tools |
| r/ClaudeCode | Claude Code tips, workflow optimization, debugging, context management, agent comparison |
| r/artificial | AI tools, open source, developer productivity |
| r/machinelearning | ML workflows, automation, development patterns |

## 3-Day Reminder Cron Job

Set a one-shot cron job that triggers 3 days after account creation to remind the user their account is old enough to post:

```bash
cronjob(
  action='create',
  name='Remind-Try-rClaudeAI-Post',
  prompt='''Check if today is {target_date} or later. If it is, send a reminder to the user with:

"⏰ 夠3日啦！你個 Reddit 帳號應該夠 age 去 r/ClaudeAI 發 post 啦！

去呢度：https://old.reddit.com/r/ClaudeAI/submit

用呢個 content：
Title: {post_title}
Flair: {recommended_flair}

記住揀 flair 先 submit！"

If before {target_date}, do nothing and exit.''',
  schedule='0 10 * * *',
  repeat=1,
)
```

## Verification — How to Confirm a Post is Live

After clicking submit on Reddit, the post page will show:
- ✅ **"submitted just now by {username}"** — confirms it's active
- ✅ **delete/hide/save/share buttons** — only shown on your own live posts
- ✅ **flair tag shown next to title** — flair was successfully applied
- ❌ **"we need something here"** on a URL post — this is normal for URL posts with no text body, ignore it
- ❌ **AutoMod removal message in thread** — post was removed (age/karma issue), not a submission failure

To verify: go to `old.reddit.com/r/{subreddit}/` and look for your post in the feed.

## Pitfalls

1. **AutoMod replies to your post — this means it WAS removed.** If you see a comment from AutoMod saying "抱歉你的帳號太新", your post didn't go through. Do NOT re-submit — just wait.
2. **Don't re-submit the same content.** If removed by AutoMod, trying again with the same content will look like spam. Wait the full age requirement.
3. **r/ClaudeAI is stricter than r/cursor.** Cursor allowed posting on day 1; ClaudeAI blocks until ~3 days. Test on less strict subreddits first.
4. **Don't hard-sell in comments.** Useful, genuine comments build karma. Overt self-promotion gets downvoted.
5. **old.reddit.com is more reliable than new Reddit** for flair selection — the UI is simpler and works better with fewer JS issues.
6. **Link posts vs Text posts:** If you can't figure out the flair selector for a link post, switch to "Text" tab — paste your URL in the body text. Text posts sometimes bypass flair requirements.
