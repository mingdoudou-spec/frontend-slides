---
name: frontend-slides
description: Create stunning, animation-rich HTML presentations from scratch or by converting PowerPoint files. Also supports VERTICAL 9:16 mode for short video backgrounds (iPhone screen). Use when the user wants to build a presentation, convert a PPT/PPTX to web, create slides for a talk/pitch, or generate vertical video slide backgrounds. Helps non-designers discover their aesthetic through visual exploration rather than abstract choices.
---

# Frontend Slides

Create zero-dependency, animation-rich HTML presentations that run entirely in the browser.

## Core Principles

1. **Zero Dependencies** — Single HTML files with inline CSS/JS. No npm, no build tools.
2. **Show, Don't Tell** — Generate visual previews, not abstract choices. People discover what they want by seeing it.
3. **Distinctive Design** — No generic "AI slop." Every presentation must feel custom-crafted.
4. **Viewport Fitting (NON-NEGOTIABLE)** — Every slide MUST fit exactly within 100vh. No scrolling within slides, ever. Content overflows? Split into multiple slides.

## Design Aesthetics

You tend to converge toward generic, "on distribution" outputs. In frontend design, this creates what users call the "AI slop" aesthetic. Avoid this: make creative, distinctive frontends that surprise and delight.

Focus on:
- Typography: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics.
- Color & Theme: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes. Draw from IDE themes and cultural aesthetics for inspiration.
- Motion: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions.
- Backgrounds: Create atmosphere and depth rather than defaulting to solid colors. Layer CSS gradients, use geometric patterns, or add contextual effects that match the overall aesthetic.

Avoid generic AI-generated aesthetics:
- Overused font families (Inter, Roboto, Arial, system fonts)
- Cliched color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and component patterns
- Cookie-cutter design that lacks context-specific character

Interpret creatively and make unexpected choices that feel genuinely designed for the context. Vary between light and dark themes, different fonts, different aesthetics. You still tend to converge on common choices (Space Grotesk, for example) across generations. Avoid this: it is critical that you think outside the box!

## Viewport Fitting Rules

These invariants apply to EVERY slide in EVERY presentation:

- Every `.slide` must have `height: 100vh; height: 100dvh; overflow: hidden;`
- ALL font sizes and spacing must use `clamp(min, preferred, max)` — never fixed px/rem
- Content containers need `max-height` constraints
- Images: `max-height: min(50vh, 400px)`
- Breakpoints required for heights: 700px, 600px, 500px
- Include `prefers-reduced-motion` support
- Never negate CSS functions directly (`-clamp()`, `-min()`, `-max()` are silently ignored) — use `calc(-1 * clamp(...))` instead

**When generating, read `viewport-base.css` and include its full contents in every presentation.**

### Content Density Limits Per Slide

| Slide Type | Maximum Content |
|------------|-----------------|
| Title slide | 1 heading + 1 subtitle + optional tagline |
| Content slide | 1 heading + 4-6 bullet points OR 1 heading + 2 paragraphs |
| Feature grid | 1 heading + 6 cards maximum (2x3 or 3x2) |
| Code slide | 1 heading + 8-10 lines of code |
| Quote slide | 1 quote (max 3 lines) + attribution |
| Image slide | 1 heading + 1 image (max 60vh height) |

**Content exceeds limits? Split into multiple slides. Never cram, never scroll.**

---

## Phase 0: Detect Mode

Determine what the user wants:

- **Mode A: New Presentation** — Create from scratch (landscape 16:9). Go to Phase 1.
- **Mode B: PPT Conversion** — Convert a .pptx file. Go to Phase 4.
- **Mode C: Enhancement** — Improve an existing HTML presentation. Read it, understand it, enhance. **Follow Mode C modification rules below.**
- **Mode D: Vertical Video (竖版短视频)** — Create 9:16 vertical slides for short video backgrounds. Go to Phase V.

### Mode C: Modification Rules

When enhancing existing presentations, viewport fitting is the biggest risk:

1. **Before adding content:** Count existing elements, check against density limits
2. **Adding images:** Must have `max-height: min(50vh, 400px)`. If slide already has max content, split into two slides
3. **Adding text:** Max 4-6 bullets per slide. Exceeds limits? Split into continuation slides
4. **After ANY modification, verify:** `.slide` has `overflow: hidden`, new elements use `clamp()`, images have viewport-relative max-height, content fits at 1280x720
5. **Proactively reorganize:** If modifications will cause overflow, automatically split content and inform the user. Don't wait to be asked

**When adding images to existing slides:** Move image to new slide or reduce other content first. Never add images without checking if existing content already fills the viewport.

---

## Phase 1: Content Discovery (New Presentations)

**Ask ALL questions in a single AskUserQuestion call** so the user fills everything out at once:

**Question 1 — Purpose** (header: "Purpose"):
What is this presentation for? Options: Pitch deck / Teaching-Tutorial / Conference talk / Internal presentation

**Question 2 — Length** (header: "Length"):
Approximately how many slides? Options: Short 5-10 / Medium 10-20 / Long 20+

**Question 3 — Content** (header: "Content"):
Do you have content ready? Options: All content ready / Rough notes / Topic only

**Question 4 — Inline Editing** (header: "Editing"):
Do you need to edit text directly in the browser after generation? Options:
- "Yes (Recommended)" — Can edit text in-browser, auto-save to localStorage, export file
- "No" — Presentation only, keeps file smaller

**Remember the user's editing choice — it determines whether edit-related code is included in Phase 3.**

If user has content, ask them to share it.

### Step 1.2: Image Evaluation (if images provided)

If user selected "No images" → skip to Phase 2.

If user provides an image folder:
1. **Scan** — List all image files (.png, .jpg, .svg, .webp, etc.)
2. **View each image** — Use the Read tool (Claude is multimodal)
3. **Evaluate** — For each: what it shows, USABLE or NOT USABLE (with reason), what concept it represents, dominant colors
4. **Co-design the outline** — Curated images inform slide structure alongside text. This is NOT "plan slides then add images" — design around both from the start (e.g., 3 screenshots → 3 feature slides, 1 logo → title/closing slide)
5. **Confirm via AskUserQuestion** (header: "Outline"): "Does this slide outline and image selection look right?" Options: Looks good / Adjust images / Adjust outline

**Logo in previews:** If a usable logo was identified, embed it (base64) into each style preview in Phase 2 — the user sees their brand styled three different ways.

---

## Phase 2: Style Discovery

**This is the "show, don't tell" phase.** Most people can't articulate design preferences in words.

### Step 2.0: Style Path

Ask how they want to choose (header: "Style"):
- "Show me options" (recommended) — Generate 3 previews based on mood
- "I know what I want" — Pick from preset list directly

**If direct selection:** Show preset picker and skip to Phase 3. Available presets are defined in [STYLE_PRESETS.md](STYLE_PRESETS.md).

### Step 2.1: Mood Selection (Guided Discovery)

Ask (header: "Vibe", multiSelect: true, max 2):
What feeling should the audience have? Options:
- Impressed/Confident — Professional, trustworthy
- Excited/Energized — Innovative, bold
- Calm/Focused — Clear, thoughtful
- Inspired/Moved — Emotional, memorable

### Step 2.2: Generate 3 Style Previews

Based on mood, generate 3 distinct single-slide HTML previews showing typography, colors, animation, and overall aesthetic. Read [STYLE_PRESETS.md](STYLE_PRESETS.md) for available presets and their specifications.

| Mood | Suggested Presets |
|------|-------------------|
| Impressed/Confident | Bold Signal, Electric Studio, Dark Botanical |
| Excited/Energized | Creative Voltage, Neon Cyber, Split Pastel |
| Calm/Focused | Notebook Tabs, Paper & Ink, Swiss Modern |
| Inspired/Moved | Dark Botanical, Vintage Editorial, Pastel Geometry |

Save previews to `.claude-design/slide-previews/` (style-a.html, style-b.html, style-c.html). Each should be self-contained, ~50-100 lines, showing one animated title slide.

Open each preview automatically for the user.

### Step 2.3: User Picks

Ask (header: "Style"):
Which style preview do you prefer? Options: Style A: [Name] / Style B: [Name] / Style C: [Name] / Mix elements

If "Mix elements", ask for specifics.

---

## Phase 3: Generate Presentation

Generate the full presentation using content from Phase 1 (text, or text + curated images) and style from Phase 2.

If images were provided, the slide outline already incorporates them from Step 1.2. If not, CSS-generated visuals (gradients, shapes, patterns) provide visual interest — this is a fully supported first-class path.

**Before generating, read these supporting files:**
- [html-template.md](html-template.md) — HTML architecture and JS features
- [viewport-base.css](viewport-base.css) — Mandatory CSS (include in full)
- [animation-patterns.md](animation-patterns.md) — Animation reference for the chosen feeling

**Key requirements:**
- Single self-contained HTML file, all CSS/JS inline
- Include the FULL contents of viewport-base.css in the `<style>` block
- Use fonts from Fontshare or Google Fonts — never system fonts
- Add detailed comments explaining each section
- Every section needs a clear `/* === SECTION NAME === */` comment block

---

## Phase V: Vertical Video Mode (竖版短视频)

For short video backgrounds — fixed 1080×1920 (9:16 iPhone ratio).

### Workflow (Simplified)

1. **User provides script/text** — oral script, bullet points, or topic
2. **Auto-analyze & split** — run Script Analyzer (Step V.1) to extract slide structure
3. **Show split plan for confirmation** — present the slide-by-slide breakdown to user
4. **Style selection** — pick from presets (same as Phase 2, but rendered vertically)
5. **Generate HTML** — using [viewport-vertical.css](viewport-vertical.css) instead of viewport-base.css
6. **Open in browser** — user screenshots or screen-records for video

### Step V.1: Script Analyzer (自动拆稿)

When user provides a continuous script, **automatically** analyze and classify each segment into slide types. This is the core intelligence of vertical mode — the user should NOT need to manually split.

**Classification rules — scan the script and tag each segment:**

| 检测信号 | 归类为 | 处理方式 |
|---------|--------|---------|
| 开头的问句、反常识陈述、冲突性判断 | **封面页** | 提炼为 ≤8 字标题 + ≤15 字副标题 |
| "我认为"、"核心是"、"关键在于"、因果论述、立场表达 | **观点页** | 提炼为 ≤10 字标题 + ≤50 字正文 |
| 并列结构（"第一…第二…"、"A、B、C"）、列举、对比 | **要点页** | 提炼为 ≤10 字标题 + 2-3 条要点 |
| 短句、节奏感强、情绪浓缩、可独立引用的判断句 | **金句页** | 原文保留或微调，≤25 字居中 |
| 具体数字、百分比、统计、排名 | **数据页** | 提取核心数字 + ≤15 字说明 |
| 两组事物的正反对照 | **对比页** | 2 列各 2-3 条，每条 ≤12 字 |
| "所以"、"总结"、行动建议、关注引导 | **结尾页** | 提炼为 ≤15 字 CTA |

**分析输出格式（给用户确认）：**

```
📋 拆稿结果（共 X 页）

第 1 页 [封面页]
  标题：XXXXX
  副标题：XXXXXXXXXX

第 2 页 [观点页]
  标题：XXXXX
  正文：XXXXXXXXXXXXXXXXXX

第 3 页 [金句页]
  "XXXXXXXXXXXXXXX"

第 4 页 [要点页]
  标题：XXXXX
  · XXXXXXXXXX
  · XXXXXXXXXX
  · XXXXXXXXXX

...

第 X 页 [结尾页]
  CTA：XXXXXXXXXX
```

**分析原则：**
- **提炼不是摘抄**。口播稿是口语化的，竖版文字需要更凝练。去掉口头禅、过渡词、重复表达
- **保留原文精华**。金句不改动原句的节奏和力度，除非超字数
- **一个观点一页**。如果原文一段话包含 2 个观点，拆成 2 页
- **数据必须突出**。原文中嵌在句子里的数字，单独拎出来做数据页
- **不要生造内容**。所有文字必须来源于用户的原稿，不添加用户没说过的观点
- **总页数控制**：1 分钟脚本 → 5-8 页，2 分钟 → 8-12 页，3 分钟 → 12-18 页

**确认后进入风格选择（Phase 2），然后生成 HTML。**

### Vertical Content Density Limits (中文)

**CRITICAL: Vertical slides have MUCH LESS space per slide than landscape. Follow these limits strictly.**

| 页面类型 | 最大内容 | 字数上限 |
|----------|---------|---------|
| 封面页 | 1 标题 + 1 副标题 | 标题 4-8 字，副标题 8-15 字 |
| 观点页 | 1 标题 + 1 段正文 | 标题 ≤10 字，正文 ≤50 字 |
| 要点页 | 1 标题 + 2-3 条要点 | 标题 ≤10 字，每条 ≤20 字 |
| 金句页 | 1 句话 + 出处 | 金句 ≤25 字 |
| 数据页 | 1 大数字 + 1 行说明 | 说明 ≤15 字 |
| 对比页 | 2 列对比（各 2-3 条） | 每条 ≤12 字 |
| 结尾页 | 1 行动号召 + 账号信息 | CTA ≤15 字 |

**每页总字数硬上限：60 字（含标题）。超过就拆页。**

### Script-to-Slides Splitting Rules

When user provides a continuous script, split using these rules:

1. **每个核心观点 = 1 页**。不要把 2 个观点塞一页
2. **转折/递进处断开**。"但是"、"所以"、"更重要的是" = 新页面
3. **数据单独成页**。一个数字配一句解释，视觉冲击力最大
4. **金句单独成页**。短句大字，留白充足
5. **首页要有钩子**。问句或反常识陈述
6. **尾页要有 CTA**。"关注"、"点赞"、"评论区告诉我"

### Vertical Generation Rules

- Use [viewport-vertical.css](viewport-vertical.css) instead of viewport-base.css
- Fixed size: `width: 1080px; height: 1920px` (not viewport-relative)
- Font sizes use fixed px (not clamp) — because canvas size is fixed
- Chinese font stack: `'Noto Sans SC', 'PingFang SC', sans-serif` as fallback
- Display fonts still from Google Fonts / Fontshare for personality
- All slides in a single HTML file, vertically stacked, with scroll-snap
- Navigation: arrow keys + swipe
- **Safe zone**: Keep text within 72px padding on all sides (avoid phone notch/home indicator area)

### Vertical Style Presets (Recommended)

For short video, these presets work best in vertical:

| 适合场景 | 推荐风格 |
|---------|---------|
| 教育/知识 | Bold Signal, Swiss Modern, Paper & Ink |
| 情感/故事 | Dark Botanical, Vintage Editorial |
| 科技/互联网 | Neon Cyber, Terminal Green, Electric Studio |
| 亲子/生活 | Pastel Geometry, Split Pastel, Notebook Tabs |

### Step V.2: Video Recording Interactivity (录屏增强)

When the user plans to **screen-record** or **live-stream** with slides as background (not just screenshot), add three interactivity layers. **Before generating, read [video-recording-interactivity.md](video-recording-interactivity.md)** for full implementation details.

1. **Continuous Micro-Animations** — Decorative elements (dots, lines, shapes) float/breathe/shimmer infinitely. **Never animate text.** Keeps slides alive during 1-2 min talking segments.
2. **Click-to-Highlight** — Titles, labels, icons respond to tap with type-specific animations. **Never add click animation to body text or bullet points** (tested and rejected — looks bad).
3. **Canvas Pen Overlay** — Press P to toggle drawing mode. Red strokes auto-fade after 2s. Uses `pointer-events: none/auto` toggle so it doesn't interfere with click-highlight when off.

**Ask the user:** "需要录屏增强功能吗？（微动效 + 点击高亮 + 画笔标注）" If yes, include all three. If the user only wants screenshots/图文, skip this step.

### Output & Usage

1. Generate single HTML file to `~/Desktop/video-pipeline/` (or user-specified path)
2. Open in Chrome
3. User can:
   - **录屏**: Chrome 全屏 → 手机投屏或直接录屏翻页
   - **截图**: 每页截图做图文轮播（小红书/抖音图文）
   - **DevTools**: `Cmd+Option+I` → 设备模拟 → iPhone 14 Pro → 逐页截图
4. If recording interactivity enabled:
   - Micro-animations run automatically (decorative elements only)
   - Click/tap titles, labels, icons for highlight effects
   - Press **P** to toggle pen drawing mode, strokes auto-fade in 2s

---

## Phase 4: PPT Conversion

When converting PowerPoint files:

1. **Extract content** — Run `python scripts/extract-pptx.py <input.pptx> <output_dir>` (install python-pptx if needed: `pip install python-pptx`)
2. **Confirm with user** — Present extracted slide titles, content summaries, and image counts
3. **Style selection** — Proceed to Phase 2 for style discovery
4. **Generate HTML** — Convert to chosen style, preserving all text, images (from assets/), slide order, and speaker notes (as HTML comments)

---

## Phase 5: Delivery

1. **Clean up** — Delete `.claude-design/slide-previews/` if it exists
2. **Open** — Use `open [filename].html` to launch in browser
3. **Summarize** — Tell the user:
   - File location, style name, slide count
   - Navigation: Arrow keys, Space, scroll/swipe, click nav dots
   - How to customize: `:root` CSS variables for colors, font link for typography, `.reveal` class for animations
   - If inline editing was enabled: Hover top-left corner or press E to enter edit mode, click any text to edit, Ctrl+S to save

---

## Supporting Files

| File | Purpose | When to Read |
|------|---------|-------------|
| [STYLE_PRESETS.md](STYLE_PRESETS.md) | 12 curated visual presets with colors, fonts, and signature elements | Phase 2 (style selection) |
| [viewport-base.css](viewport-base.css) | Mandatory responsive CSS — copy into every landscape presentation | Phase 3 (generation) |
| [viewport-vertical.css](viewport-vertical.css) | Fixed 1080×1920 CSS — copy into every vertical presentation | Phase V (vertical video) |
| [html-template.md](html-template.md) | HTML structure, JS features, code quality standards | Phase 3 / Phase V (generation) |
| [animation-patterns.md](animation-patterns.md) | CSS/JS animation snippets and effect-to-feeling guide | Phase 3 / Phase V (generation) |
| [video-recording-interactivity.md](video-recording-interactivity.md) | Micro-animations, click-to-highlight, pen drawing overlay for screen recording | Phase V Step V.2 (when user records video) |
| [scripts/extract-pptx.py](scripts/extract-pptx.py) | Python script for PPT content extraction | Phase 4 (conversion) |
