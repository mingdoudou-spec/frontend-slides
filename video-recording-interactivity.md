# Video Recording Interactivity Patterns

When HTML slides are used as backgrounds for short video recording (not just live presenting), static slides look "dead" while the presenter talks for 1-2 minutes per slide. These three patterns solve that problem.

**When to use:** Vertical video mode (Phase V) when the user plans to screen-record or live-stream with slides as background.

---

## Pattern 1: Continuous Micro-Animations (Ambient Motion)

Add subtle infinite CSS animations to **decorative elements only** -- never to text content. These keep the slide alive during long talking segments.

### Animation Library

```css
/* Float — gentle vertical drift for icons, shapes, decorative dots */
@keyframes float {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-12px); }
}

/* Breathe — scale pulse for badges, accent shapes, circular elements */
@keyframes breathe {
    0%, 100% { transform: scale(1); opacity: 0.7; }
    50% { transform: scale(1.08); opacity: 1; }
}

/* Shimmer — light sweep for borders, dividers, decorative lines */
@keyframes shimmer {
    0% { background-position: -200% center; }
    100% { background-position: 200% center; }
}

/* Drift — slow horizontal float for background shapes */
@keyframes drift {
    0%, 100% { transform: translate(0, 0) rotate(0deg); }
    25% { transform: translate(8px, -5px) rotate(1deg); }
    50% { transform: translate(-3px, 8px) rotate(-1deg); }
    75% { transform: translate(-8px, -3px) rotate(0.5deg); }
}

/* Rotate-slow — for decorative icons, stars, geometric shapes */
@keyframes rotate-slow {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

/* Gradient-shift — for background gradient animation */
@keyframes gradient-shift {
    0%, 100% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
}
```

### Application Rules

| Element Type | Animation | Duration | Delay Strategy |
|-------------|-----------|----------|---------------|
| Decorative icons | `float` | 3-4s | Stagger by 0.5s per icon |
| Accent shapes (circles, dots) | `breathe` | 4-5s | Stagger by 0.8s |
| Divider lines | `shimmer` | 3s | No delay needed |
| Background shapes | `drift` | 8-12s | Randomize |
| Small geometric elements | `rotate-slow` | 15-20s | Randomize |
| Gradient backgrounds | `gradient-shift` | 8s | No delay |

### Critical Rules

1. **NEVER animate text content** (titles, body text, bullet points). Moving text is distracting and hard to read.
2. **NEVER animate the entire slide**. Only individual decorative elements.
3. **Keep amplitudes tiny**. `translateY(-12px)` max for float, `scale(1.08)` max for breathe. Anything larger distracts from the speaker.
4. **Use `ease-in-out` timing** for all ambient animations. Linear looks mechanical.
5. **Stagger delays** so elements don't move in unison (looks robotic).
6. **Use `animation-fill-mode: both`** and `animation-iteration-count: infinite`.
7. **Performance**: Only animate `transform` and `opacity`. Never animate `width`, `height`, `margin`, or `top/left`.

### Adding Decorative Elements for Animation

If a slide has no decorative elements, add subtle ones:

```html
<!-- Floating accent dots (place 2-3 per slide in corners/margins) -->
<div class="deco-dot" style="top: 15%; right: 8%;"></div>
<div class="deco-dot" style="bottom: 20%; left: 5%;"></div>

<!-- Animated divider line -->
<div class="shimmer-line"></div>
```

```css
.deco-dot {
    position: absolute;
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: var(--accent);
    opacity: 0.4;
    animation: breathe 4s ease-in-out infinite;
}

.shimmer-line {
    height: 2px;
    background: linear-gradient(90deg, transparent, var(--accent), transparent);
    background-size: 200% 100%;
    animation: shimmer 3s ease-in-out infinite;
}
```

---

## Pattern 2: Click-to-Highlight (Presenter Tap Feedback)

Makes structural elements respond to click/tap with type-specific highlight animations. Designed for presenters who tap their screen while talking to draw attention to specific points.

### Element Classification

| Element Type | Click Animation | Duration | Rationale |
|-------------|----------------|----------|-----------|
| Titles (h1, h2) | Scale up 1.05x + glow shadow | 600ms | Draws attention to topic |
| Labels / tags | Background fill + color invert | 400ms | Visually "selects" the label |
| Icons / emojis | Spin 360° + scale bounce | 500ms | Playful, eye-catching |
| Cards / containers | Border glow + subtle lift | 500ms | Highlights the section |
| Body text / paragraphs | **NO ANIMATION** | — | Looks "dumb" (user feedback) |
| Bullet points | **NO ANIMATION** | — | Too much movement, distracting |

### Implementation

```javascript
function initClickHighlight() {
    // Title click — scale + glow
    document.querySelectorAll('h1, h2, .slide-title').forEach(el => {
        el.style.cursor = 'pointer';
        el.addEventListener('click', () => {
            el.classList.add('highlight-active');
            setTimeout(() => el.classList.remove('highlight-active'), 600);
        });
    });

    // Label/tag click — fill color
    document.querySelectorAll('.tag, .label, .badge').forEach(el => {
        el.style.cursor = 'pointer';
        el.addEventListener('click', () => {
            el.classList.add('highlight-active');
            setTimeout(() => el.classList.remove('highlight-active'), 400);
        });
    });

    // Icon click — spin + bounce
    document.querySelectorAll('.icon, .emoji-icon').forEach(el => {
        el.style.cursor = 'pointer';
        el.addEventListener('click', () => {
            el.classList.add('highlight-active');
            setTimeout(() => el.classList.remove('highlight-active'), 500);
        });
    });

    // Explicitly skip body text
    // document.querySelectorAll('p, li') — DO NOT add click handlers
}
```

```css
/* Title highlight */
h1.highlight-active, h2.highlight-active, .slide-title.highlight-active {
    animation: title-highlight 0.6s ease-out;
}
@keyframes title-highlight {
    0% { transform: scale(1); text-shadow: none; }
    40% { transform: scale(1.05); text-shadow: 0 0 20px var(--accent); }
    100% { transform: scale(1); text-shadow: none; }
}

/* Label highlight */
.tag.highlight-active, .label.highlight-active {
    animation: label-highlight 0.4s ease-out;
}
@keyframes label-highlight {
    0% { background: transparent; color: var(--text); }
    40% { background: var(--accent); color: var(--bg); }
    100% { background: transparent; color: var(--text); }
}

/* Icon highlight */
.icon.highlight-active, .emoji-icon.highlight-active {
    animation: icon-highlight 0.5s ease-out;
}
@keyframes icon-highlight {
    0% { transform: scale(1) rotate(0deg); }
    50% { transform: scale(1.3) rotate(180deg); }
    100% { transform: scale(1) rotate(360deg); }
}
```

### Key Insight

Body text click animation was explicitly tested and rejected by the user ("看着很蠢" / "looks dumb"). The reason: body text is dense and occupies large areas -- animating it creates a jarring, unprofessional effect. Only **structural** elements (titles, labels, icons) benefit from tap feedback because they're visually distinct and isolated.

---

## Pattern 3: Canvas Pen Drawing Overlay

A transparent canvas overlay that lets the presenter draw temporary annotations (like Keynote's laser pointer). Strokes auto-fade after 2 seconds.

### Architecture

```
┌─────────────────────────┐
│  Canvas (pen overlay)   │ ← pointer-events: none (default)
│  z-index: 9999          │    pointer-events: auto (pen mode)
├─────────────────────────┤
│  Slide content          │ ← Normal click interactions work
│  (with click-highlight) │    when pen mode is OFF
└─────────────────────────┘
```

### Implementation

```javascript
function initPenOverlay() {
    const canvas = document.createElement('canvas');
    canvas.id = 'pen-overlay';
    canvas.style.cssText = `
        position: fixed; top: 0; left: 0;
        width: 100%; height: 100%;
        z-index: 9999;
        pointer-events: none;
        cursor: crosshair;
    `;
    document.body.appendChild(canvas);

    const ctx = canvas.getContext('2d');
    let penMode = false;
    let isDrawing = false;
    let strokes = []; // Array of {points, startTime}

    // Resize canvas to match window
    function resize() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    }
    resize();
    window.addEventListener('resize', resize);

    // Toggle pen mode with 'P' key
    document.addEventListener('keydown', (e) => {
        if (e.key === 'p' || e.key === 'P') {
            penMode = !penMode;
            canvas.style.pointerEvents = penMode ? 'auto' : 'none';
            document.body.style.cursor = penMode ? 'crosshair' : '';
            // Visual indicator
            showPenIndicator(penMode);
        }
    });

    // Drawing handlers
    canvas.addEventListener('mousedown', (e) => {
        if (!penMode) return;
        isDrawing = true;
        strokes.push({
            points: [{x: e.clientX, y: e.clientY}],
            startTime: Date.now()
        });
    });

    canvas.addEventListener('mousemove', (e) => {
        if (!isDrawing) return;
        const current = strokes[strokes.length - 1];
        current.points.push({x: e.clientX, y: e.clientY});
    });

    canvas.addEventListener('mouseup', () => { isDrawing = false; });
    canvas.addEventListener('mouseleave', () => { isDrawing = false; });

    // Touch support for iPad/phone recording
    canvas.addEventListener('touchstart', (e) => {
        if (!penMode) return;
        e.preventDefault();
        const t = e.touches[0];
        isDrawing = true;
        strokes.push({
            points: [{x: t.clientX, y: t.clientY}],
            startTime: Date.now()
        });
    });

    canvas.addEventListener('touchmove', (e) => {
        if (!isDrawing) return;
        e.preventDefault();
        const t = e.touches[0];
        strokes[strokes.length - 1].points.push({x: t.clientX, y: t.clientY});
    });

    canvas.addEventListener('touchend', () => { isDrawing = false; });

    // Render loop: draw strokes + fade old ones
    function render() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        const now = Date.now();
        const FADE_DURATION = 2000; // 2 seconds

        strokes = strokes.filter(stroke => {
            const age = now - stroke.startTime;
            if (age > FADE_DURATION && !isDrawing) return false;

            const opacity = Math.max(0, 1 - (age / FADE_DURATION));
            ctx.beginPath();
            ctx.strokeStyle = `rgba(255, 50, 50, ${opacity})`; // Red pen
            ctx.lineWidth = 3;
            ctx.lineCap = 'round';
            ctx.lineJoin = 'round';

            const pts = stroke.points;
            if (pts.length < 2) return true;

            ctx.moveTo(pts[0].x, pts[0].y);
            // Smooth curves using quadratic bezier
            for (let i = 1; i < pts.length - 1; i++) {
                const midX = (pts[i].x + pts[i + 1].x) / 2;
                const midY = (pts[i].y + pts[i + 1].y) / 2;
                ctx.quadraticCurveTo(pts[i].x, pts[i].y, midX, midY);
            }
            // Last segment
            const last = pts[pts.length - 1];
            ctx.lineTo(last.x, last.y);
            ctx.stroke();

            return true;
        });

        requestAnimationFrame(render);
    }
    render();

    // Pen mode indicator
    function showPenIndicator(active) {
        let indicator = document.getElementById('pen-indicator');
        if (!indicator) {
            indicator = document.createElement('div');
            indicator.id = 'pen-indicator';
            indicator.style.cssText = `
                position: fixed; top: 20px; right: 20px;
                padding: 8px 16px; border-radius: 20px;
                font-size: 14px; font-weight: bold; z-index: 10000;
                transition: opacity 0.3s, transform 0.3s;
                pointer-events: none;
            `;
            document.body.appendChild(indicator);
        }
        indicator.textContent = active ? '🖊 Pen ON' : '';
        indicator.style.background = active ? 'rgba(255,50,50,0.9)' : 'transparent';
        indicator.style.color = 'white';
        indicator.style.opacity = active ? '1' : '0';
    }
}
```

### Configuration Options

| Setting | Default | Adjust When |
|---------|---------|------------|
| Fade duration | 2000ms | Longer for tutorial-style videos (3-4s), shorter for rapid annotation (1s) |
| Stroke color | `rgba(255, 50, 50, opacity)` red | Match brand colors or use yellow for dark themes |
| Line width | 3px | Thicker (5px) for phone screens, thinner (2px) for desktop recording |
| Toggle key | P | Change if conflicts with other shortcuts |

### Interaction with Other Patterns

- When pen mode is **OFF**: `pointer-events: none` on canvas, click-to-highlight works normally on slide elements.
- When pen mode is **ON**: `pointer-events: auto` on canvas, captures all mouse/touch events. Click-to-highlight is temporarily disabled (by design -- you're drawing, not clicking elements).
- Micro-animations continue running regardless of pen mode (they're CSS, not affected by pointer events).

---

## Integration Checklist

When generating vertical video slides with recording interactivity, include all three patterns:

- [ ] Add micro-animation CSS keyframes in `<style>`
- [ ] Add decorative elements (dots, lines, shapes) to each slide with appropriate animations
- [ ] Add click-highlight CSS and JS for titles, labels, icons
- [ ] Explicitly skip `<p>` and `<li>` elements from click handlers
- [ ] Add pen overlay canvas and JS at end of `<body>`
- [ ] Include `prefers-reduced-motion` media query to disable all animations
- [ ] Test: micro-animations run continuously, clicks highlight structural elements, P toggles pen

```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}
```
