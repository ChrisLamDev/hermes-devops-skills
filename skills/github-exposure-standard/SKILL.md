---
name: github-exposure-standard
description: Use when releasing a new open-source project or skill pack on GitHub. Follows the proven exposure playbook from Hermes Core Skills release — covering README, community health, topics, social media, and verification. Ensures every release meets a minimum exposure bar so the project is discoverable, trustworthy, and ready for adoption.
version: 1.0.0
author: Chris Lam / Hermes Agent (experiential learning from hermes-core-skills release, Jun 2026)
tags: [github, release, exposure, open-source, README, social-proof, community-health, topics, badges, SEO]
trigger_phrases:
  - "release this to GitHub"
  - "publish on GitHub"
  - "提高曝光"
  - "發佈去 GitHub"
  - "做好個 repo 先"
  - "GitHub exposure"
  - "準備 release"
  - "開源發佈"
---

# GitHub Release Exposure Standard

## Point of View

當你準備 release 一個新 open-source project 上 GitHub，唔可以就咁 push code 就算。你需要一個**系統性曝光流程**，確保每個層面嘅用家（全端工程師、AI 初學者、CTO、開源貢獻者、非英語用家等）都搵到佢哋想要嘅嘢，從而提高 star、fork、adoption 嘅機會。

呢個標準係從 hermes-core-skills 發佈嘅實戰經驗提煉出嚟，涵蓋 **3 個階段、15 項操作**。

---

## Phase 1: Repo 基礎建設（先於發佈）

呢啲係發佈前就要做嘅底層工作，唔可以後補。

### 1.1 README 結構標準

README 必須包含以下 sections（按順序）：

```
[1] Title + Badges row
[2] One-liner tagline（解決咩問題）
[3] Pain points / 場景描述（You've been there）
[4] Quick Start（git clone + 多平台 install，參考 Anthropic Cybersecurity Skills 嘅 npm + clone 雙選方式）
[5] Why This Project（背景動機 + 痛點 + 解決方案，參考佢哋嘅 workforce gap framing）
[6] Comparison vs alternatives（如適用，參考佢哋嘅 comparison table style）
[7] 使用範例（至少 3 個實際 scenarios，對話格式）
[8] Feature overview / Skill table + Skill Anatomy（解釋每個 skill 嘅內部結構）
[9] Compatible Platforms table（列出所有支援平台，參考佢哋嘅 category breakdown）
[10] Beginner guide（For Beginners section，4 個基本問題）
[11] Use Cases table（邊個角色用邊個功能）
[12] Roadmap（未來發展路線，checkbox 形式）
[13] What People Are Saying（testimonials / 早期用家 feedback）
[14] Star History（star-history.com 圖表 badge）
[15] Featured In（被邊啲 awesome list / 平台收錄）
[16] How to Contribute（Good First Issues，包括非 code 貢獻）
[17] Citation（學術引用格式 bibtex）
[18] License
```

**參考案例：** [mukul975/Anthropic-Cybersecurity-Skills](https://github.com/mukul975/Anthropic-Cybersecurity-Skills)（18.8k ⭐）嘅 README 結構係 gold standard：
- 佢有「Why This Project」section 用 workforce gap 帶出 project 必要性
- 佢有「Quick Start」用 npm install + git clone 雙選
- 佢有「What People Are Saying」testimonials 增加社交 proof
- 佢有「Featured In」section 顯示被收錄狀態
- 佢有「Star History」圖表展示增長趨勢
- 佢有「Compatible Platforms」詳細分類（AI Code Assistants / Autonomous Agents / Agent Frameworks）
- 佢有「Citation」bibtex 格式俾學術用家引用
- 佢有「Skill Anatomy」解釋每個 skill 嘅 YAML frontmatter + markdown body 結構
- 佢有「Roadmap」唔使用 checkbox 形式，用 bullet list 描述未來計劃

**Badges 要求：**

**Badges 要求：**
- GitHub stars（動態）
- GitHub forks（動態）
- License（MIT 推薦）
- 每個支援平台一個 badge（Claude Code、Cursor、Codex、Hermes Agent 等）

Badges 格式範例：
```
[![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-8A2BE2?style=for-the-badge&logo=anthropic)](https://claude.ai)
```

**Pain Points section 必須用「You've been there:」bullet list 形式**，直接打中 target audience 嘅日常痛點。

### 1.2 Community Health Files

Repo 必須有以下檔案先算 complete：

| 檔案 | 位置 | 用途 |
|------|------|------|
| `CODE_OF_CONDUCT.md` | root | Contributor Covenant |
| `CONTRIBUTING.md` | root | 貢獻指引 |
| `SECURITY.md` | root | 安全漏洞上報流程 |
| `LICENSE` | root | MIT 推薦 |
| `.github/CODEOWNERS` | .github/ | 預設 code owner |
| `.github/FUNDING.yml` | .github/ | Funding 鏈接 |
| `.github/PULL_REQUEST_TEMPLATE.md` | .github/ | PR template |
| `.github/ISSUE_TEMPLATE/bug_report.md` | .github/ISSUE_TEMPLATE/ | Bug report template |
| `.github/ISSUE_TEMPLATE/feature_request.md` | .github/ISSUE_TEMPLATE/ | Feature request template |
| `.github/ISSUE_TEMPLATE/config.yml` | .github/ISSUE_TEMPLATE/ | 指向 Discussions |

**GitHub 會自動檢測呢啲檔案**，喺 repo 頂部顯示 Community Standards 進度條。目標係 100%。

### 1.5 Infrastructure Files（參考 Anthropic Cybersecurity Skills）

以下 infrastructure 檔案令 repo 更專業、agent 更容易使用：

| 檔案 | 用途 | 建議 |
|------|------|------|
| `index.json` | 所有 skills 嘅 JSON 索引，agent 可快速 scan 唔使逐個 file 開 | **必加** — 用嚟俾 agent discovery |
| `SPECT-*.md` | 大型改動前嘅 spec document | 建議保留喺 repo 做記錄 |
| `mappings/` | Skills 對應標準框架（如 MITRE、NIST、用家場景） | 後續可加 |
| `.claude-plugin/` | Claude Code marketplace 插件設定 | 後續可加 |
| `docs/` | 補充文件 | 選擇性加 |

**`index.json` 格式範例：**
```json
{
  "version": "1.0.0",
  "generated_at": "2026-06-23T10:00:00Z",
  "repository": "https://github.com/user/repo",
  "domain": "your-domain",
  "total_skills": 25,
  "skills": [
    {
      "name": "skill-name",
      "description": "Brief description",
      "domain": "main-category",
      "subdomain": "sub-category",
      "path": "skills/skill-name",
      "tokens": {
        "scan": 150,
        "load": 2500,
        "category": "standard"
      }
    }
  ]
}
```

### 1.6 SKILL.md Frontmatter 標準

每個 SKILL.md 嘅 frontmatter 應該有：

```yaml
---
name: skill-name
description: ... 
domain: main-category      # 主分類
subdomain: sub-category    # 子分類
tokens:
  scan: 150                # Frontmatter scan token cost
  load: 2500               # Full load token cost  
  category: standard       # brief / standard / detailed
---
```

Domain 分類系統參考（4 domains, 9 subdomains）：
```
agent-core: quality-control, safety, efficiency, planning
ecosystem: agent-comparison, agent-skills
cross-session: continuity, setup
integration: architecture
```

去 repo Settings → About section → Topics input，加以下 tags（空格分隔，最多 20 個）：

**核心 tags（必加）：**
- `ai-agent` `open-source` `developer-tools`

**平台 tags（視乎支援）：**
- `claude-code` `codex` `mcp` `hermes-agent`

**功能 tags：**
- `automation` `workflow-automation` `skills` `agent-skills`

**搜尋 reach tags（擴展 reach）：**
- `langchain` `llamaindex` `cursor` `prompt-engineering`

**資源限制：**
- Fine-grained PAT **唔可以**透過 API 寫 topics（403）
- Topics 必須用 browser UI 手動 set
- 建議分批次 save（3-5 tags 一次），太多 tags 同時 save 會 fail

---

## Phase 2: README 深度內容（發佈時做）

### 2.1 Why This Project Section

參考 Anthropic Cybersecurity Skills 嘅 framing pattern：

```markdown
## 💡 Why These Skills?

### The problem

[Describe the industry-wide pain point with concrete numbers/statements]

### The solution

[Explain how this project solves that problem - what makes it different]
```

用「problem → solution」嘅 framing，唔好直接 sell features。先令讀者覺得「佢明我嘅痛」，再講點解你嘅方案得。

### 2.2 Skill Anatomy Section

解釋每個 skill 嘅內部結構，幫助用家/貢獻者理解格式：

```markdown
## 🧬 Skill Anatomy

Every skill follows a consistent directory structure:

skills/<skill-name>/
├── SKILL.md          # Main skill file (YAML frontmatter + markdown body)
├── references/       # External references, guides
└── scripts/          # Helper scripts (optional)
```

如果係 YAML frontmatter 格式，提供真實例子。

### 2.3 Compatible Platforms Table

列出所有支援平台，分 category：

```
| Category | Platform |
|----------|----------|
| **AI Code Assistants** | Claude Code, OpenAI Codex CLI, Cursor, Hermes Agent, GitHub Copilot |
| **Agent Frameworks** | LangChain, CrewAI, AutoGen, Any MCP-compatible agent |
| **MCP Clients** | Claude Desktop, VS Code via Continue/Cline, JetBrains, any MCP host |
```

### 2.4 Testimonials / What People Are Saying

Social proof section 放 early adopter quotes：

```
> *"[Concrete benefit quote]"*
> — **Role / Person, Date**

> *"[Specific feature saved me X]"*
> — **Role / Person, Date**
```

唔好作 fake quotes。如果未有 real feedback，用「Early adopter feedback」等 neutral phrasing。

### 2.5 Star History

用 star-history.com badge 顯示增長趨勢：

```
[![Star History Chart](https://api.star-history.com/svg?repos=user/repo&type=Date)](https://star-history.com/#user/repo&Date)
```

呢個 badge auto-update，展示 project 嘅 social proof。

### 2.6 Featured In Section

列出被邊啲 awesome list / platform 收錄：

```
## 📖 Featured In

- **awesome-list-name** — Description ([link](url))
- **platform-name** — Description ([link](url))
```

如果未有被收錄，用「Want to feature this project? [Let us know](discussion-link)」俾人聯絡。

### 2.7 Citation Section

學術引用格式（bibtex），幫 researcher 引用：

```bibtex
@software{project_name,
  author = {Your Name},
  title = {Project Title},
  year = {2026},
  url = {https://github.com/user/repo},
  note = {Brief description. License.}
}
```

### 2.8 Roadmap Section

未來發展路線，checkbox 形式或 bullet list：

```markdown
## 🗺️ Roadmap

- [x] v1.0.0 — Core features released
- [ ] v1.1.0 — Next milestone
- [ ] v2.0.0 — Major upgrade
```

### 2.9 Platform-Specific Quick Start

每個支援平台都要有自己嘅 install section，格式：

```
### Platform Name

\`\`\`bash
cp -r skills/* ~/.platform/skills/
\`\`\`

Platform auto-discovers skills in ~/.platform/skills/. After copying, just ask:

> *"Use the [skill-name] skill to [task]."*
```

唔可以只寫一條通用 command。每個平台嘅 behavior 唔同，要分開寫。

### 2.2 實際對話範例（Scenarios）

至少 3 個 scenarios，格式：

```
### Scenario N: [Pain point title]

\`\`\`
You: "[Prompt using the skill]"
Agent: [Loads workflow]
  - Step 1
  - Step 2
  - Step 3
Result: [Concrete outcome]
\`\`\`
```

每個 scenario 必須對應一個真實痛點，同埋一個具體 skill。

### 2.3 Comparison Table（如適用）

同 alternatives 比較，格式：

```
| Feature | This Project | Alternative A | Alternative B |
|---------|-------------|---------------|---------------|
| Feature 1 | ✅ ✓ | ❌ ✗ | ❌ ✗ |
| Feature 2 | ✅ ✓ | ❌ ✗ | ✅ ✓ |
```

必須誠實。只比較真正有優勢嘅維度。

### 2.4 Beginner Guide

For Beginners section 必須回答 4 個問題：
1. **What is [project type]?** — 用 simplest 嘅定義
2. **What's a [core concept]?** — 比喻解釋（e.g. skill = recipe for AI chef）
3. **How do I use it?** — 3 步流程
4. **Which [feature] should I start with?** — 推薦順序

### 2.5 Use Cases Table

依角色分類，格式：

```
| Who | Problem | Solution |
|-----|---------|----------|
| **Solo developer** | [pain point] | [specific features] |
| **Startup CTO** | [pain point] | [specific features] |
```

### 2.6 Contribution Guide

「Good First Issues」必須列出**唔使寫 code**嘅貢獻方式：
- 📝 Review a feature
- 🌐 Translate README
- 🐛 Report a bug
- ✨ Suggest new features

### 2.7 Translation Invites

加多語言 badges，鼓勵 community 貢獻翻譯：
```
[![中文](https://img.shields.io/badge/中文-翻译-8A2BE2?style=for-the-badge)](discussion-link)
[![日本語](https://img.shields.io/badge/日本語-翻訳-8A2BE2?style=for-the-badge)](discussion-link)
```

### 2.8 Visual Placeholder

如果未有 screenshot/demo GIF，至少加一個 placeholder section：
```
<div align="center">
## 📸 Demo
*Screenshot coming soon — [watch the repo](link) for updates*
</div>
```

---

## Phase 3: 曝光渠道（發佈後立即做）

### 3.1 GitHub 層面

| 操作 | 點做 | 效果 |
|------|------|------|
| Pin repo 去 profile | Profile → Popular repos → Customize pins | Profile 首頁顯示 |
| GitHub Pages 啟用 | Settings → Pages → Enable | 有 landing page |
| Release 建立 | Releases → Draft new release → Tag + 描述 | v1.0.0 ZIP 可下載 |
| Discussions 啟用 | Settings → General → Discussions ✅ | Community 交流區 |
| Social preview image | Settings → Social preview → upload 1280x640 PNG | 分享 link 時有 preview |
| Wiki / Projects 關閉 | Settings → 唔需要就關 | 保持 repo 整潔 |
| Profile README 更新 | 個人 profile repo 加 pinned repo 描述 | Profile 引導流量 |

### 3.2 社交媒體層面

| 平台 | 內容策略 | 頻率 |
|------|---------|------|
| **X/Twitter** | Short post + repo link + 1-2 個痛點 bullet + 1 screenshot/GIF | 發佈日 1 次 |
| **Reddit (r/programming, r/MachineLearning, r/ClaudeAI)** | Longer post → project介紹 → pain point → 同 alternatives 比較 → 點解唔同 | 發佈日 1 次 |
| **Hacker News** | Show HN post → 直接 link repo → comment 區回應問題 | 發佈日 1 次 |

**Social post 結構標準：**
1. Hook（一句捉住注意力，e.g. "Your AI coding agent keeps doing dumb things?"）
2. Pain point bullet（3 個 bullet max）
3. 解決方案（簡單一句）
4. Link to repo + 話明 MIT license
5. Call to action（Try it, star it）

---

## Top 100 爆星因素研究（2026-06-23 分析）

從 GitHub Top 100 starred repos 研究所得，排除社交媒體因素，純粹 GitHub 層面嘅爆星共通因素：

### 爆星因素排名（由最重要到次要）

1. **零摩擦價值主張（≤3秒明）** — 標題+第一句就要令人明呢個 repo 做咩。爆星 repo 全部符合
2. **問題導向定位** — 唔好 sell features，sell solutions。用 pain points 做章節標題（仿 mattpocock/skills 嘅 `#1`, `#2`, `#3` pattern）
3. **Social Proof Badges** — 自訂 badge 顯示採用率（仿 anthropics/skills 嘅「▲ Skills - 2.2M」）
4. **快速入門體驗** — Quick Start 要寫「30 seconds」，越低 friction 越好
5. **平台兼容性** — 支援越多平台，distribution channel 越闊。obra/superpowers 支援 5+ IDE plugins
6. **Topics SEO** — 越多 topics 越容易被發現。hermes-agent 有 16 個 topics 係策略性
7. **作者/機構品牌** — 呢個最難複製但最重要。anthropics, github, torvalds, sindresorhus 全部有品牌光環
8. **Bookmarkability** — Lists / tutorials / primers 呢類 repo 天然易爆星（star = bookmark）。Skills repo 都係 reference material，有同樣優勢

### 具體執行建議

- README 頭 20 行必須涵蓋：**badges row + tagline + numbered pain points + Quick Start**
- pain points 用 `#1`, `#2`, `#3` 形式，唔好用普通 bullet list
- 加 custom badge 顯示 skills count / users count
- Quick Start 寫「30 seconds」唔係「Quick Start」
- Topics 策略：16-20 個 tags，覆蓋平台 + 功能 + 搜尋 reach 三類

用以下 checklist 驗證每個項目完成咗先叫「發佈完成」：

### ✅ Phase 1: Repo 基礎建設
- [ ] README has badges row (stars, forks, license, platforms)
- [ ] Community health files: CODE_OF_CONDUCT, CONTRIBUTING, LICENSE, SECURITY
- [ ] .github/: CODEOWNERS, FUNDING.yml, PR template, issue templates, config.yml
- [ ] Topics tags set via browser UI (16-20 tags)
- [ ] Social preview image uploaded (1280x640px)

### ✅ Phase 2: README 深度內容
- [ ] Why This Project section (problem → solution framing)
- [ ] Quick Start with multiple install options (git clone + platform-specific)
- [ ] 3+ scenario examples showing how to use the project
- [ ] Comparison vs alternatives table (if applicable)
- [ ] Skill Anatomy section (explain file structure + format)
- [ ] Compatible Platforms table (categorized)
- [ ] For Beginners section (4 questions answered)
- [ ] Use Cases table (by role)
- [ ] Roadmap section (checkbox or bullet list)
- [ ] Testimonials / What People Are Saying (early feedback)
- [ ] Star History chart badge
- [ ] Featured In section (awesome lists, platforms)
- [ ] Citation section (bibtex format)
- [ ] How to Contribute section (with non-code good first issues)
- [ ] Translation badges (至少 3 種語言)
- [ ] Visual placeholder (screenshot/demo coming soon)
- [ ] License section

### ✅ Phase 3: 曝光渠道
- [ ] Repo pinned to profile
- [ ] Release created (v1.0.0 with tag + description)
- [ ] GitHub Pages enabled (optional)
- [ ] Discussions enabled
- [ ] Social media posts ready (X + Reddit + HN)
- [ ] Profile README updated with reference to this repo

---

## Pitfalls

- **Topics API 403**: Fine-grained PAT 唔可以寫 topics，必須 browser UI 手動。分批次 save 3-5 tags 一次，全部 save 會 fail
- **社交 proof chicken-and-egg**: 0 stars 會令新用家卻步。呢個只能靠社交媒體發文引發第一波。聽日發文後有人 star 先有 momentum
- **README 太長嚇人**: 用 emoji + 分隔線 --- + tables 增加掃讀性。關鍵信息放頭 30%
- **Translation badges 點去邊**: Badges 必須 link 去 Discussions／PR，唔好 link 去 dead page
- **Badge logo 參數**: shields.io badge logo 參數必須正確，否則 badge 顯示空白。測試每個 badge 先 push
- **GIF demo 困難**: Headless 環境錄 terminal GIF 需要 ttyd/VHS。如果環境唔支援，可叫 user 用 asciinema / QuickTime 自己錄
- **Community health 檢測 delay**: GitHub 需要時間 crawl 新檔案，community standards 可能延遲 1-24 小時先更新到 100%
- **Fine-grained PAT 嘅 Git push**：`gh auth login` 後 `git push` 可以 work，但 `gh repo create` 會 403。Create repo 需要 user 手動去 github.com/new

## 相關 Skill

- `github-repo-management` — Cloning, creating, configuring repos
- `github-finegrained-pat-limitations` — PAT edge cases
- `github-topics-react-form-workaround` — Browser UI topics workaround
- `github-skills-release-preparation` — Original release prep workflow
