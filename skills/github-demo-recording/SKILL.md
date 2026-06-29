---
name: github-demo-recording
description: Auto-detect environment and record terminal/GUI demos for GitHub repos. Supports VHS, asciinema+agg, svg-term, and cua-driver modes. Single-command orchestration layer.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [demo, recording, VHS, GIF, asciinema, terminal, cua-driver, github]
    related_skills: [github-repo-management, buddha-heart-boot-sequence]
    tokens:
      scan: 65
      load: 280
      category: tool
---

# GitHub Demo Recording Skill

Record terminal/GUI demos for GitHub repos. Single command, auto-detect environment, multiple output formats (GIF/SVG/MP4).

## When to use

- User asks "點樣錄個 demo" or "整段 demo"
- User needs a GIF for README
- User asks "錄個安裝步驟" or "show me how to use"
- You need to create demo media for repo promotion
- The README has a placeholder like "![Demo]()"

## Core principle

**Don't reinvent the wheel.** Use existing tools as engines, orchestrate via AI.

## Architecture

```
Scenario Classifier → Script Generator → Executor → Quality Check → Delivery
     │                     │               │              │             │
     │                     ▼               ▼              ▼             ▼
  tape/VHS             demo.tape      vhs run        size/fps     README update
  asciicast            asciinema      agg render     clarity      git commit
  svg                  svg-term       svg output     correctness  release attach
  cua-screen           cua script     cua record     resolution   asset upload
```

## Available recording modes

| Mode | Tool | Output | Requirements | Best for |
|------|------|--------|-------------|----------|
| `tape` | VHS (charmbracelet/vhs) | GIF | Go runtime | Terminal commands |
| `asciicast` | asciinema + agg | GIF | Python + Rust | Terminal, cross-platform |
| `svg` | svg-term-cli | SVG | Node.js | Headless/CI, inline README |
| `cua-screen` | cua-driver + ffmpeg | MP4 | macOS + cua-daemon | GUI apps, full-screen |

## Environment detection

Run these checks in order to determine available tools:

```bash
# VHS
which vhs && vhs --version

# asciinema
which asciinema && asciinema --version

# agg (asciinema/agg)
which agg && agg --version

# svg-term
which svg-term && svg-term --version

# cua-driver (MCP server)
test -f ~/.cua-driver/config.json && echo "cua-driver configured"

# ffmpeg (for GIF composition)
which ffmpeg && ffmpeg -version | head -1

# Native terminal capture
which script && which scriptreplay
```

## Demo types & auto-selection

| User says they want to demo... | Recommended mode | Priority |
|---|---|---|
| "裝 skills 嘅步驟" | `tape` (VHS) | High |
| "run 一個 command 嘅效果" | `asciicast` | High |
| "展示 GUI 操作" | `cua-screen` | Low (macOS only) |
| "睇 code 結構 / 目錄" | `svg` (svg-term) | Medium |
| "頭先講嘅 workflow" | Best available | Auto |

## Step-by-step workflow

### Step 1: Understand the demo goal
Ask or infer: what should the demo show?
- Installation process?
- Feature walkthrough?
- CLI usage flow?
- GUI interaction?

### Step 2: Generate demo script
If using VHS/tape mode:
Create a `.tape` file with:
```vhs
Output demo.gif
Set Width 800
Set Height 400
Set FontSize 14
Type "git clone https://github.com/user/repo.git" 1s
Enter
Sleep 3s
Type "cd repo" 1s
Enter
Sleep 1s
Type "ls" 500ms
Enter
Sleep 2s
# ... more steps
```

If using asciicast mode:
Use `script` to capture real session, or write commands for `asciinema rec`.

If using svg mode (best for headless/CI):
Two approaches:

**Approach A: svg-term-cli (if available)**
```bash
# Generate asciicast v2 JSON → SVG
svg-term --cast demo.cast --out demo.svg --window
```

**Approach B: Pure Python SVG generator (recommended for headless)**
When svg-term is not available or fails, use the Python-based SVG generator from `scripts/gen-svg.py`.
It generates an animated SVG with CSS `@keyframes + <animate>` — no external dependencies.
Output is a self-contained SVG that works in all browsers, on GitHub README, and scales perfectly.

The generator:
1. Reads an asciicast v2 file (`.cast`)
2. Reconstructs screen state over time from output data
3. Strips ANSI escape codes, keeps printable characters
4. Produces an animated SVG with per-frame opacity transitions (68 frames, ~28s duration)
5. Output is typically 150-200KB (vs. 3-5MB for equivalent GIF)

Key features:
- Pure Python, no external deps
- Works in headless/CI environments
- Terminal color scheme (dark theme with traffic-light title bar)
- Color-codes output: green (headings/success), blue (prompts), gray (dim/comments), purple (box art)
- Final blinking cursor
- SVG can be embedded directly inline in Markdown, or linked as an external file

If using cua-screen mode:
1. Launch Terminal.app via cua-driver
2. Type commands via `type_text`
3. Capture frames via `get_window_state`
4. Composite via ffmpeg

### Step 3: Execute recording

**VHS mode:**
```bash
vhs demo.tape
# Output: demo.gif
```

**asciinema mode:**
```bash
# Record
asciinema rec demo.cast --command "bash demo-script.sh"
# Convert to GIF
agg demo.cast demo.gif
```

**svg mode:**
```bash
# Convert asciicast to SVG
svg-term --cast demo.cast --out demo.svg --window
```

### Step 4: Quality check

```bash
# Check GIF size (should be < 5MB for GitHub)
ls -lh demo.gif

# Check dimensions
identify -verbose demo.gif | grep "Geometry"  # requires ImageMagick

# Check frame count and FPS
identify -format "%n %s\n" demo.gif | head -5
```

Minimum quality thresholds:
- File size < 5MB
- Width ≥ 600px
- Font readable
- No flickering (stable frame rate)
- Complete demo flow (no missing steps)

### Step 5: Delivery

**Update README:**
```markdown
## Demo

![Install Demo](./screenshots/demo.gif)
```

**Create screenshots directory:**
```bash
mkdir -p screenshots
mv demo.gif screenshots/
```

**Commit and push:**
```bash
git add screenshots/demo.gif README.md
git commit -m "docs: add installation demo GIF"
git push
```

## Pitfalls

1. **Headless environment can't run VHS** — VHS needs a real PTY. Fallback to svg-term which generates SVG from text.
2. **GIF > 5MB breaks GitHub rendering** — Always quality-check. Use `gifsicle --optimize=3` to compress.
3. **asciinema needs a real terminal** — In CI/headless, use `script` command instead (always available on Linux).
4. **VHS may not be installed** — Auto-detect and suggest fallback. Never assume.
5. **Cua Driver needs macOS** — Skip cua-screen mode on Linux/headless.
6. **`script` command output includes ANSI escape codes** — Use `agg` or `svg-term` which handle ANSI natively.
7. **Type speed matters** — In VHS tape scripts, add `Sleep` between commands for natural pacing.

## Installation of tools

### VHS (Go)
```bash
go install github.com/charmbracelet/vhs@latest
```
Or on macOS:
```bash
brew install vhs
```

### asciinema (Python)
```bash
pip install asciinema
```

### agg (Rust)
```bash
cargo install agg
```
Or download from [GitHub releases](https://github.com/asciinema/agg).

### svg-term (Node.js)
```bash
npm install -g svg-term-cli
```

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| VHS "no PTY available" | Headless environment | Use svg-term fallback |
| GIF too large | Too many frames | Reduce frame count, use gifsicle |
| asciinema "not a terminal" | Script mode | Use `script` + `agg` pipeline |
| svg-term empty output | Invalid asciicast JSON | Check JSON validity |
| Cua Driver "permission denied" | TCC not granted | `cua-driver check-permissions` |

## Verification checklist

- [ ] Environment detection completed
- [ ] Demo script generated
- [ ] Recording executed successfully
- [ ] Output file exists and is non-empty
- [ ] File size < 5MB (GIF) or < 1MB (SVG)
- [ ] Visual quality acceptable (readable, complete flow)
- [ ] README or target location updated
- [ ] Git commit made (if applicable)
