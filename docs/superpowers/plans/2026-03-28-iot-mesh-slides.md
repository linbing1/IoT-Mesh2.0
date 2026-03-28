# IoT-Mesh 2.0 HTML Slides Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert 6 pages (10-15) of the IoT-Mesh 2.0 PDF into an interactive HTML slide deck with SVG diagrams, deployed to Vercel.

**Architecture:** Vite + TypeScript project reusing the slide engine from `working-with-ai`. Each slide is a `<section class="slide">` in `index.html` with `data-f` fragment attributes. All diagrams are inline SVG/CSS.

**Tech Stack:** Vite, TypeScript, CSS, inline SVG

---

## File Structure

| File | Responsibility |
|------|---------------|
| `package.json` | Vite + TS dependencies, scripts |
| `tsconfig.json` | TypeScript config (from working-with-ai) |
| `vite.config.ts` | Vite config (minimal) |
| `index.html` | All 6 slides HTML content + SVG diagrams |
| `src/main.ts` | Slide engine (stripped from working-with-ai) |
| `src/style.css` | Base styles + IoT theme + slide-specific styles |
| `public/favicon.svg` | Simple IoT mesh favicon |

---

### Task 1: Project scaffolding

**Files:**
- Create: `package.json`
- Create: `tsconfig.json`
- Create: `vite.config.ts`
- Create: `public/favicon.svg`
- Create: `.gitignore`

- [ ] **Step 1: Initialize git repo**

```bash
cd /Users/linbing/Project/github/IoT-Mesh2.0_HTML
git init
```

- [ ] **Step 2: Create package.json**

Create `package.json`:

```json
{
  "name": "iot-mesh-2.0-slides",
  "version": "0.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite dev",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "typescript": "~5.7.0",
    "vite": "^6.0.0"
  }
}
```

Note: Use standard vite (not vite-plus) to keep it simple and widely compatible.

- [ ] **Step 3: Create tsconfig.json**

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2023", "DOM", "DOM.Iterable"],
    "types": ["vite/client"],
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

- [ ] **Step 4: Create vite.config.ts**

Create `vite.config.ts`:

```ts
import { defineConfig } from "vite";

export default defineConfig({});
```

- [ ] **Step 5: Create .gitignore**

Create `.gitignore`:

```
node_modules
dist
.DS_Store
```

- [ ] **Step 6: Create favicon**

Create `public/favicon.svg` — a simple mesh/network icon:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <circle cx="16" cy="6" r="3" fill="#00d4aa"/>
  <circle cx="6" cy="26" r="3" fill="#00d4aa"/>
  <circle cx="26" cy="26" r="3" fill="#00d4aa"/>
  <circle cx="16" cy="18" r="2.5" fill="#3b82f6"/>
  <line x1="16" y1="6" x2="6" y2="26" stroke="#00d4aa" stroke-width="1" opacity="0.4"/>
  <line x1="16" y1="6" x2="26" y2="26" stroke="#00d4aa" stroke-width="1" opacity="0.4"/>
  <line x1="6" y1="26" x2="26" y2="26" stroke="#00d4aa" stroke-width="1" opacity="0.4"/>
  <line x1="16" y1="6" x2="16" y2="18" stroke="#3b82f6" stroke-width="1" opacity="0.4"/>
  <line x1="6" y1="26" x2="16" y2="18" stroke="#3b82f6" stroke-width="1" opacity="0.4"/>
  <line x1="26" y1="26" x2="16" y2="18" stroke="#3b82f6" stroke-width="1" opacity="0.4"/>
</svg>
```

- [ ] **Step 7: Install dependencies and verify**

```bash
cd /Users/linbing/Project/github/IoT-Mesh2.0_HTML
npm install
```

Expected: `node_modules` created, no errors.

- [ ] **Step 8: Commit**

```bash
git add package.json tsconfig.json vite.config.ts public/favicon.svg .gitignore
git commit -m "feat: scaffold Vite + TypeScript project"
```

---

### Task 2: Slide engine (main.ts)

**Files:**
- Create: `src/main.ts`

Copy the slide engine from `working-with-ai/src/main.ts`, removing all demo-specific code (JPEG demo, emoji quiz, AI taste toggle, iframe overlay). Keep: slide switching, fragment system, keyboard nav, overview mode, mobile nav, progress bar, FOUC guard.

- [ ] **Step 1: Create src/main.ts**

Create `src/main.ts`:

```ts
import "./style.css";

// ============================================================
// Slide engine with fragment support
// ============================================================

const slides = Array.from(document.querySelectorAll<HTMLElement>(".slide"));
let current = 0;

const fragmentState = new Map<number, number>();

function getFragments(slideIdx: number): HTMLElement[] {
  return Array.from(
    slides[slideIdx].querySelectorAll<HTMLElement>("[data-f]"),
  ).sort((a, b) => Number(a.dataset.f) - Number(b.dataset.f));
}

function maxFragment(slideIdx: number): number {
  const frags = getFragments(slideIdx);
  if (frags.length === 0) return 0;
  const unique = new Set(frags.map((el) => Number(el.dataset.f)));
  return unique.size;
}

function currentFragment(slideIdx: number): number {
  return fragmentState.get(slideIdx) ?? 0;
}

function showFragmentsUpTo(slideIdx: number, n: number) {
  const frags = getFragments(slideIdx);
  const vals = [...new Set(frags.map((el) => Number(el.dataset.f)))].sort(
    (a, b) => a - b,
  );
  const visibleVals = new Set(vals.slice(0, n));
  frags.forEach((el) => {
    el.classList.toggle("f-visible", visibleVals.has(Number(el.dataset.f)));
  });
  fragmentState.set(slideIdx, n);
}

function goto(index: number, direction?: "forward" | "backward") {
  const clamped = Math.max(0, Math.min(index, slides.length - 1));
  if (clamped === current) return;
  const dir = direction ?? (clamped > current ? "forward" : "backward");
  const prevSlide = slides[current];
  const nextSlide = slides[clamped];
  prevSlide.classList.remove("active");
  prevSlide.classList.add(dir === "forward" ? "exit-left" : "exit-right");
  nextSlide.classList.remove("exit-left", "exit-right");
  nextSlide.classList.add("active");
  const prevIdx = current;
  current = clamped;
  if (dir === "backward") {
    showFragmentsUpTo(clamped, maxFragment(clamped));
  } else {
    showFragmentsUpTo(clamped, 0);
  }
  updateProgress();
  setTimeout(
    () => slides[prevIdx].classList.remove("exit-left", "exit-right"),
    600,
  );
}

function stepForward() {
  const cur = currentFragment(current);
  const max = maxFragment(current);
  if (cur < max) {
    showFragmentsUpTo(current, cur + 1);
  } else {
    goto(current + 1, "forward");
  }
}

function stepBackward() {
  const cur = currentFragment(current);
  if (cur > 0) {
    showFragmentsUpTo(current, cur - 1);
  } else {
    goto(current - 1, "backward");
  }
}

function updateProgress() {
  const bar = document.getElementById("progress-bar") as HTMLElement | null;
  if (bar) bar.style.width = `${((current + 1) / slides.length) * 100}%`;
  const counter = document.getElementById("slide-counter");
  if (counter) counter.textContent = `${current + 1} / ${slides.length}`;
}

// ============================================================
// Overview mode
// ============================================================

let overviewActive = false;

const REF_W = 1280;
const REF_H = 800;

function scaleOverviewThumbs(container: HTMLElement) {
  const thumbs = container.querySelectorAll<HTMLElement>(".overview-thumb");
  if (thumbs.length === 0) return;
  requestAnimationFrame(() => {
    const thumbW = thumbs[0].clientWidth;
    if (thumbW === 0) return;
    const scale = thumbW / REF_W;
    container.querySelectorAll<HTMLElement>(".overview-inner").forEach((inner) => {
      inner.style.width = REF_W + "px";
      inner.style.height = REF_H + "px";
      inner.style.transform = `scale(${scale})`;
    });
  });
}

function buildOverview() {
  let container = document.getElementById("overview");
  if (container) {
    scaleOverviewThumbs(container);
    return container;
  }
  container = document.createElement("div");
  container.id = "overview";
  container.className = "overview";
  document.body.appendChild(container);

  slides.forEach((slide, i) => {
    const thumb = document.createElement("div");
    thumb.className = "overview-thumb" + (i === current ? " current" : "");
    thumb.dataset.index = String(i);

    const inner = document.createElement("div");
    inner.className = "overview-inner";
    const clone = slide.cloneNode(true) as HTMLElement;
    clone.querySelectorAll("[data-f]").forEach((el) => {
      el.classList.add("f-visible");
    });
    inner.innerHTML = clone.innerHTML;
    thumb.appendChild(inner);

    const label = document.createElement("div");
    label.className = "overview-label";
    const titleEl =
      slide.querySelector(".title-mega") ||
      slide.querySelector(".title-large");
    const titleText = titleEl ? titleEl.textContent?.trim() ?? "" : "";
    label.textContent = titleText
      ? `${i + 1}. ${titleText}`
      : `${i + 1}`;
    thumb.appendChild(label);

    thumb.addEventListener("click", () => {
      toggleOverview(false);
      if (i === current) {
        showFragmentsUpTo(current, 0);
      } else {
        goto(i, "forward");
      }
    });
    container.appendChild(thumb);
  });

  scaleOverviewThumbs(container);
  return container;
}

function toggleOverview(show?: boolean) {
  overviewActive = show ?? !overviewActive;
  const container = buildOverview();

  container.querySelectorAll(".overview-thumb").forEach((el, i) => {
    el.classList.toggle("current", i === current);
  });

  container.classList.toggle("active", overviewActive);
  document.body.classList.toggle("overview-active", overviewActive);
}

// ============================================================
// Navigation
// ============================================================

document.addEventListener("keydown", (e) => {
  if (e.key === "Escape") {
    e.preventDefault();
    toggleOverview();
    return;
  }

  if (overviewActive) {
    if (["ArrowRight", "ArrowDown"].includes(e.key)) {
      e.preventDefault();
      const next = Math.min(current + 1, slides.length - 1);
      goto(next);
      const container = document.getElementById("overview");
      container?.querySelectorAll(".overview-thumb").forEach((el, i) => {
        el.classList.toggle("current", i === current);
      });
    } else if (["ArrowLeft", "ArrowUp"].includes(e.key)) {
      e.preventDefault();
      const prev = Math.max(current - 1, 0);
      goto(prev);
      const container = document.getElementById("overview");
      container?.querySelectorAll(".overview-thumb").forEach((el, i) => {
        el.classList.toggle("current", i === current);
      });
    } else if (e.key === "Enter" || e.key === " ") {
      e.preventDefault();
      toggleOverview(false);
    }
    return;
  }

  if (["ArrowRight", " ", "PageDown"].includes(e.key)) {
    e.preventDefault();
    goto(current + 1, "forward");
  } else if (["ArrowLeft", "PageUp"].includes(e.key)) {
    e.preventDefault();
    goto(current - 1, "backward");
  } else if (e.key === "ArrowDown") {
    e.preventDefault();
    stepForward();
  } else if (e.key === "ArrowUp") {
    e.preventDefault();
    stepBackward();
  }
});

// ============================================================
// Init
// ============================================================

if (slides.length > 0) {
  slides[0].classList.add("active");
  showFragmentsUpTo(0, maxFragment(0));
}
updateProgress();

// Mobile nav bar
document.getElementById("mnav-prev")?.addEventListener("click", () => goto(current - 1, "backward"));
document.getElementById("mnav-next")?.addEventListener("click", () => goto(current + 1, "forward"));
document.getElementById("mnav-up")?.addEventListener("click", () => stepBackward());
document.getElementById("mnav-down")?.addEventListener("click", () => stepForward());
document.getElementById("mnav-overview")?.addEventListener("click", () => toggleOverview());

// Remove FOUC guard
requestAnimationFrame(() => {
  requestAnimationFrame(() => {
    const app = document.getElementById("app");
    if (app) {
      app.style.opacity = "";
      app.classList.add("ready");
    }
  });
});
```

- [ ] **Step 2: Verify TypeScript compiles**

```bash
cd /Users/linbing/Project/github/IoT-Mesh2.0_HTML
npx tsc --noEmit
```

Expected: No errors (style.css import will be resolved by Vite at runtime; tsc may warn but should not fail since we have `"skipLibCheck": true`). If tsc complains about the missing CSS file, that's expected at this stage — it will resolve once we create style.css in Task 3.

- [ ] **Step 3: Commit**

```bash
git add src/main.ts
git commit -m "feat: add slide engine with fragment, overview, and navigation support"
```

---

### Task 3: Base styles (style.css)

**Files:**
- Create: `src/style.css`

Base styles adapted from `working-with-ai/src/style.css` with IoT color scheme. Includes: reset, slide transitions, fragment system, progress bar, overview mode, mobile nav, typography utilities, and the 1.0-vs-2.0 comparison layout used across all slides.

- [ ] **Step 1: Create src/style.css**

Create `src/style.css`:

```css
/* ============================================================
   Base
   ============================================================ */

@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@300;400;700;900&family=Space+Grotesk:wght@400;600;700&family=JetBrains+Mono:wght@400;700&display=swap');

:root {
  --bg: #0a0a12;
  --fg: #e8e8f0;
  --accent: #00d4aa;
  --accent2: #3b82f6;
  --dim: #666;
  --old: #ff6b6b;
  --card-bg: rgba(255,255,255,0.04);
  --card-border: rgba(255,255,255,0.08);
  --font-sans: 'Space Grotesk','Noto Sans SC',system-ui,sans-serif;
  --font-mono: 'JetBrains Mono','Menlo',monospace;
  --font-cn: 'Noto Sans SC',system-ui,sans-serif;
}

*,*::before,*::after { margin:0; padding:0; box-sizing:border-box; }
html,body { width:100%; height:100%; overflow:hidden; background:var(--bg); color:var(--fg); font-family:var(--font-sans); -webkit-font-smoothing:antialiased; touch-action:manipulation; }

/* Slides */
#app { width:100vw; height:100vh; position:relative; overflow:hidden; }
#app.ready { opacity:1; }
.slide { position:absolute; inset:0; display:flex; flex-direction:column; align-items:center; justify-content:center; padding:3rem 5rem; opacity:0; pointer-events:none; transform:translateX(60px); transition:opacity .5s ease, transform .5s ease; overflow-y:auto; }
.slide.active { opacity:1; pointer-events:auto; transform:translateX(0); }
.slide.exit-left { opacity:0; transform:translateX(-60px); }
.slide.exit-right { opacity:0; transform:translateX(60px); }

/* Fragment system */
[data-f] { opacity:0; transform:translateY(15px); transition:opacity .4s ease, transform .4s ease; pointer-events:none; }
[data-f].f-visible { opacity:1; transform:translateY(0); pointer-events:auto; }

/* Progress */
#progress { position:fixed; bottom:0; left:0; width:100%; height:3px; background:rgba(255,255,255,0.06); z-index:100; }
#progress-bar { height:100%; background:linear-gradient(90deg,var(--accent),var(--accent2)); transition:width .4s ease; width:0; }
#slide-counter { position:fixed; bottom:12px; right:24px; font-family:var(--font-mono); font-size:.75rem; color:var(--dim); z-index:100; }

/* Typography */
.title-large { font-size:clamp(2rem,4.5vw,3.5rem); font-weight:700; line-height:1.2; letter-spacing:-.02em; text-align:center; }
.mono { font-family:var(--font-mono); }
.cn { font-family:var(--font-cn); }
.dim { color:var(--dim); }
.text-center { text-align:center; }
.mt-1 { margin-top:.5rem; }
.mt-2 { margin-top:1rem; }
.mt-3 { margin-top:1.5rem; }
.flex-col { display:flex; flex-direction:column; }
.gap-2 { gap:1rem; }
.gap-3 { gap:1.5rem; }

/* Gradient text */
.gradient-text { background:linear-gradient(135deg,var(--accent),var(--accent2)); -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text; }

/* Nav hint */
.nav-hint { position:fixed; bottom:24px; left:50%; transform:translateX(-50%); font-size:.75rem; color:var(--dim); font-family:var(--font-mono); z-index:100; opacity:.5; pointer-events:none; }

/* ============================================================
   1.0 vs 2.0 comparison layout
   ============================================================ */

.compare { display:grid; grid-template-columns:1fr 1fr; gap:3rem; width:100%; max-width:1000px; align-items:start; }
.compare-side { display:flex; flex-direction:column; align-items:center; gap:1rem; }

.compare-badge { font-family:var(--font-mono); font-size:.85rem; font-weight:700; padding:.3em .8em; border-radius:100px; }
.compare-badge.old { color:var(--old); border:1px solid var(--old); opacity:0.7; }
.compare-badge.new { color:var(--accent); border:1px solid var(--accent); }

.metric { font-family:var(--font-mono); font-size:1.1rem; font-weight:700; }
.metric.old { color:var(--old); opacity:0.7; }
.metric.new { color:var(--accent); }

.desc { font-family:var(--font-cn); font-size:.85rem; color:var(--dim); text-align:center; max-width:380px; line-height:1.5; }
.desc .highlight { color:var(--accent); font-weight:700; }
.desc .highlight-blue { color:var(--accent2); font-weight:700; }

/* ============================================================
   SVG diagram shared styles
   ============================================================ */

.diagram { width:100%; max-width:400px; }
.diagram svg { width:100%; height:auto; }

/* ============================================================
   Overview
   ============================================================ */

.overview { position:fixed; inset:0; z-index:50; background:var(--bg); display:flex; flex-wrap:wrap; align-content:flex-start; gap:1rem; padding:2rem; overflow-y:auto; opacity:0; pointer-events:none; transition:opacity .3s ease; }
.overview.active { opacity:1; pointer-events:auto; }
.overview-thumb { position:relative; aspect-ratio:16/10; width:calc((100% - 3rem) / 4); border:2px solid var(--card-border); border-radius:8px; overflow:hidden; cursor:pointer; transition:border-color .2s, transform .2s; background:var(--bg); flex-shrink:0; }
.overview-thumb:hover { border-color:var(--accent); transform:scale(1.03); }
.overview-thumb.current { border-color:var(--accent); box-shadow:0 0 12px rgba(0,212,170,0.3); }
.overview-inner { position:absolute; top:0; left:0; transform-origin:top left; pointer-events:none; overflow:hidden; padding:3rem 5rem; display:flex; flex-direction:column; align-items:center; justify-content:center; }
.overview-label { position:absolute; bottom:4px; right:6px; font-family:var(--font-mono); font-size:.65rem; color:var(--dim); background:rgba(10,10,18,0.7); padding:1px 5px; border-radius:3px; }
body.overview-active #progress,
body.overview-active #slide-counter,
body.overview-active .nav-hint { opacity:0; pointer-events:none; }

/* ============================================================
   Mobile nav
   ============================================================ */

.mobile-nav { display:none; }

@media (max-width:768px) {
  .slide { padding:2rem 1.5rem; }
  .compare { grid-template-columns:1fr; gap:2rem; }
  .nav-hint { display:none; }

  .mobile-nav { display:flex; position:fixed; bottom:0; left:0; right:0; z-index:100; justify-content:center; align-items:center; gap:0; background:rgba(10,10,18,0.85); backdrop-filter:blur(8px); border-top:1px solid var(--card-border); padding:6px 0; padding-bottom:max(6px,env(safe-area-inset-bottom)); }
  .mobile-nav-btn { flex:1; max-width:72px; height:36px; border:none; background:transparent; color:var(--dim); font-size:1rem; cursor:pointer; display:flex; align-items:center; justify-content:center; transition:color .15s; -webkit-tap-highlight-color:transparent; }
  .mobile-nav-btn:active { color:var(--accent); }

  .overview { gap:.5rem; padding:.5rem; padding-bottom:4rem; }
  .overview-thumb { width:calc((100% - .5rem) / 2); }
  body.overview-active .mobile-nav-btn:not(.mobile-nav-overview) { opacity:0.3; pointer-events:none; }
  body.overview-active .mobile-nav-overview { color:var(--accent); }
}

/* ============================================================
   Slide-specific: Reliability (mesh topology)
   ============================================================ */

.mesh-node { fill:var(--accent); }
.mesh-node-center { fill:var(--accent2); }
.mesh-link { stroke:var(--accent); stroke-width:1.5; opacity:0.3; }
.mesh-link-active { stroke:var(--accent); stroke-width:2; opacity:0.8; animation:meshPulse 2s ease-in-out infinite; }
@keyframes meshPulse { 0%,100% { opacity:0.4; } 50% { opacity:1; } }

/* Retransmission bars */
.retrans-bars { display:flex; gap:2rem; justify-content:center; }
.retrans-group { display:flex; flex-direction:column; align-items:center; gap:.3rem; }
.retrans-group-label { font-family:var(--font-mono); font-size:.7rem; color:var(--dim); }
.bar-row { display:flex; gap:3px; align-items:flex-end; }
.bar { width:6px; border-radius:2px 2px 0 0; }
.bar-green { background:var(--accent); }
.bar-blue { background:var(--accent2); }

@keyframes barGrow { from { transform:scaleY(0); } to { transform:scaleY(1); } }
.bar-animated { transform-origin:bottom; animation:barGrow .4s ease-out both; }

/* ============================================================
   Slide-specific: Consistency (broadcast sets)
   ============================================================ */

.broadcast-sets { display:flex; gap:3px; align-items:flex-end; height:80px; }
.bcast-bar { width:8px; border-radius:2px 2px 0 0; }
.bcast-1 { background:var(--accent2); }
.bcast-2 { background:var(--accent); }

/* ============================================================
   Slide-specific: OTA (routing path)
   ============================================================ */

.ota-path-node { fill:var(--accent); }
.ota-path-line { stroke:var(--accent); stroke-width:2; stroke-dasharray:6 3; animation:dashFlow 1.5s linear infinite; }
@keyframes dashFlow { to { stroke-dashoffset:-18; } }

/* ============================================================
   Slide-specific: Provisioning (sequence diagram)
   ============================================================ */

.seq-diagram { font-family:var(--font-cn); }
.seq-lifeline { stroke:var(--dim); stroke-width:1; stroke-dasharray:4 3; }
.seq-arrow { stroke:var(--dim); stroke-width:1.5; fill:none; marker-end:url(#arrowhead-dim); }
.seq-arrow-new { stroke:var(--accent); stroke-width:1.5; fill:none; marker-end:url(#arrowhead-accent); }
.seq-label { font-size:12px; fill:var(--fg); }
.seq-label-dim { font-size:12px; fill:var(--dim); }

/* ============================================================
   Slide-specific: Multi-device (fan arrows)
   ============================================================ */

.fan-arrow { stroke:var(--accent); stroke-width:2; fill:none; opacity:0.8; }

/* ============================================================
   Slide-specific: Power (waveforms)
   ============================================================ */

.wave-old { stroke:var(--dim); stroke-width:2; fill:none; }
.wave-new { stroke:var(--accent); stroke-width:2; fill:none; }
.wave-new-glow { stroke:var(--accent); stroke-width:4; fill:none; opacity:0.2; }

@keyframes waveGlow {
  0%,100% { opacity:0.1; }
  50% { opacity:0.4; }
}
.wave-animated { animation:waveGlow 2s ease-in-out infinite; }

/* ============================================================
   Device icons (CSS)
   ============================================================ */

.device-gateway { width:60px; height:40px; background:linear-gradient(135deg,#2a2a35,#1a1a25); border-radius:6px; border:1px solid rgba(255,255,255,0.1); position:relative; }
.device-gateway::after { content:''; position:absolute; top:-6px; left:50%; transform:translateX(-50%); width:4px; height:6px; background:var(--accent2); border-radius:2px 2px 0 0; }

.device-bulb { width:28px; height:36px; position:relative; }
.device-bulb::before { content:''; position:absolute; top:0; left:50%; transform:translateX(-50%); width:24px; height:24px; background:radial-gradient(circle at 40% 35%,#ffd54f,#ff9800); border-radius:50%; }
.device-bulb::after { content:''; position:absolute; bottom:0; left:50%; transform:translateX(-50%); width:12px; height:10px; background:#888; border-radius:0 0 4px 4px; }

.device-bulb-dim::before { background:radial-gradient(circle at 40% 35%,#555,#333); }
```

- [ ] **Step 2: Verify Vite dev server starts**

```bash
cd /Users/linbing/Project/github/IoT-Mesh2.0_HTML
npx vite --open=false &
sleep 2
curl -s http://localhost:5173 | head -5
kill %1
```

Expected: HTML response (will be empty since index.html doesn't exist yet, but Vite should start without errors).

- [ ] **Step 3: Commit**

```bash
git add src/style.css
git commit -m "feat: add base styles with IoT color scheme and comparison layout"
```

---

### Task 4: HTML shell + Slide 1 (Reliability)

**Files:**
- Create: `index.html`

Build the HTML shell with FOUC guard, progress bar, mobile nav, and the first slide. The first slide is "可靠性" with a mesh topology SVG and retransmission bars.

- [ ] **Step 1: Create index.html with shell + slide 1**

Create `index.html`:

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>IoT-Mesh 2.0</title>
    <style>
      body { background: #0a0a12; color: #e8e8f0; }
      #app:not(.ready) { opacity: 0; }
      #app:not(.ready) .slide,
      #app:not(.ready) [data-f] { transition: none !important; }
    </style>
  </head>
  <body>
    <div id="app" style="opacity:0">

      <!-- ============================== 1. Reliability ============================== -->
      <section class="slide">
        <div class="flex-col gap-3" style="align-items:center">
          <div class="title-large cn">IoT-Mesh 2.0 <span class="gradient-text">可靠性</span></div>
          <div class="metric new mono">2.0 控制成功率 >99.9%</div>

          <div class="compare mt-2">
            <!-- Left: Mesh topology -->
            <div class="compare-side" data-f="1">
              <div class="diagram">
                <svg viewBox="0 0 300 220" fill="none">
                  <!-- Mesh nodes -->
                  <circle cx="150" cy="30" r="12" class="mesh-node"/>
                  <circle cx="50" cy="110" r="12" class="mesh-node"/>
                  <circle cx="250" cy="110" r="12" class="mesh-node"/>
                  <circle cx="100" cy="200" r="12" class="mesh-node"/>
                  <circle cx="200" cy="200" r="12" class="mesh-node"/>
                  <circle cx="150" cy="120" r="10" class="mesh-node-center"/>
                  <!-- Links -->
                  <line x1="150" y1="30" x2="50" y2="110" class="mesh-link-active"/>
                  <line x1="150" y1="30" x2="250" y2="110" class="mesh-link-active"/>
                  <line x1="50" y1="110" x2="100" y2="200" class="mesh-link-active"/>
                  <line x1="250" y1="110" x2="200" y2="200" class="mesh-link-active"/>
                  <line x1="100" y1="200" x2="200" y2="200" class="mesh-link"/>
                  <line x1="50" y1="110" x2="150" y2="120" class="mesh-link-active"/>
                  <line x1="250" y1="110" x2="150" y2="120" class="mesh-link-active"/>
                  <line x1="150" y1="120" x2="100" y2="200" class="mesh-link"/>
                  <line x1="150" y1="120" x2="200" y2="200" class="mesh-link"/>
                  <!-- Relay label -->
                  <text x="150" y="170" text-anchor="middle" fill="#666" font-size="12" font-family="Noto Sans SC">默认开启 Relay</text>
                </svg>
              </div>
              <div class="desc cn">可靠性来源于消息积累<br>所有设备<span class="highlight">默认开启Relay</span></div>
            </div>

            <!-- Right: Retransmission bars -->
            <div class="compare-side" data-f="2">
              <div class="retrans-bars">
                <div class="retrans-group">
                  <div class="bar-row">
                    <div class="bar bar-green bar-animated" style="height:50px;animation-delay:0s"></div>
                    <div class="bar bar-green bar-animated" style="height:45px;animation-delay:.05s"></div>
                    <div class="bar bar-green bar-animated" style="height:55px;animation-delay:.1s"></div>
                    <div class="bar bar-blue bar-animated" style="height:48px;animation-delay:.15s"></div>
                    <div class="bar bar-blue bar-animated" style="height:52px;animation-delay:.2s"></div>
                  </div>
                  <div class="retrans-group-label">控制命令 1</div>
                </div>
                <div class="retrans-group">
                  <div class="bar-row">
                    <div class="bar bar-green bar-animated" style="height:52px;animation-delay:.4s"></div>
                    <div class="bar bar-green bar-animated" style="height:48px;animation-delay:.45s"></div>
                    <div class="bar bar-green bar-animated" style="height:50px;animation-delay:.5s"></div>
                    <div class="bar bar-blue bar-animated" style="height:55px;animation-delay:.55s"></div>
                    <div class="bar bar-blue bar-animated" style="height:45px;animation-delay:.6s"></div>
                  </div>
                  <div class="retrans-group-label">控制命令 2</div>
                </div>
                <div class="retrans-group">
                  <div class="bar-row">
                    <div class="bar bar-green bar-animated" style="height:48px;animation-delay:.8s"></div>
                    <div class="bar bar-green bar-animated" style="height:55px;animation-delay:.85s"></div>
                    <div class="bar bar-green bar-animated" style="height:50px;animation-delay:.9s"></div>
                    <div class="bar bar-blue bar-animated" style="height:52px;animation-delay:.95s"></div>
                    <div class="bar bar-blue bar-animated" style="height:50px;animation-delay:1s"></div>
                  </div>
                  <div class="retrans-group-label">控制命令 3</div>
                </div>
              </div>
              <div class="desc cn">网关/子设备<span class="highlight-blue">网络层重发</span><br>网关设置消息发送队列，动态调整<span class="highlight">重发次数</span>和间隔</div>
            </div>
          </div>
        </div>
      </section>

    </div>

    <!-- Progress bar -->
    <div id="progress"><div id="progress-bar"></div></div>
    <div id="slide-counter" class="mono"></div>
    <div class="nav-hint">← → 翻页 · ↑ ↓ 步进 · ESC 总览</div>

    <!-- Mobile nav -->
    <nav class="mobile-nav">
      <button class="mobile-nav-btn" id="mnav-prev">◀</button>
      <button class="mobile-nav-btn" id="mnav-up">▲</button>
      <button class="mobile-nav-btn mobile-nav-overview" id="mnav-overview">▣</button>
      <button class="mobile-nav-btn" id="mnav-down">▼</button>
      <button class="mobile-nav-btn" id="mnav-next">▶</button>
    </nav>

    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

- [ ] **Step 2: Verify dev server renders slide 1**

```bash
cd /Users/linbing/Project/github/IoT-Mesh2.0_HTML
npx vite --open
```

Expected: Browser opens, dark background, slide 1 shows "IoT-Mesh 2.0 可靠性" title with green metric. Press ↓ to step through fragments: mesh topology appears, then retransmission bars with animated bars growing from bottom.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML shell and slide 1 (reliability with mesh topology)"
```

---

### Task 5: Slide 2 — Consistency & Real-time (一致性/实时性)

**Files:**
- Modify: `index.html` (add slide 2 after slide 1's closing `</section>`)

- [ ] **Step 1: Add slide 2 HTML**

Insert after the first `</section>` and before `</div><!-- app -->`:

```html
      <!-- ============================== 2. Consistency / Real-time ============================== -->
      <section class="slide">
        <div class="flex-col gap-3" style="align-items:center">
          <div class="title-large cn">IoT-Mesh 2.0 <span class="gradient-text">一致性/实时性</span></div>
          <div class="mono" style="font-size:1.1rem">
            <span class="metric new">2.0 一致性 &lt;100ms</span>
            &nbsp;&nbsp;
            <span class="metric new">实时性 &lt;200ms</span>
          </div>

          <div class="compare mt-2">
            <!-- Left: Broadcast sets -->
            <div class="compare-side" data-f="1">
              <div class="desc cn" style="margin-bottom:.5rem">
                <span class="highlight-blue">广播集 1</span> &nbsp;
                <span class="highlight">广播集 2</span>
              </div>
              <div class="broadcast-sets">
                <div class="bcast-bar bcast-1" style="height:60px"></div>
                <div class="bcast-bar bcast-1" style="height:55px"></div>
                <div class="bcast-bar bcast-1" style="height:65px"></div>
                <div class="bcast-bar bcast-2" style="height:50px"></div>
                <div class="bcast-bar bcast-2" style="height:58px"></div>
                <div class="bcast-bar bcast-1" style="height:62px"></div>
                <div class="bcast-bar bcast-1" style="height:55px"></div>
                <div class="bcast-bar bcast-1" style="height:60px"></div>
                <div class="bcast-bar bcast-2" style="height:52px"></div>
                <div class="bcast-bar bcast-2" style="height:56px"></div>
              </div>
              <div class="desc cn mt-1">Relay两个消息，每个重发3次<br>设备支持多<span class="highlight-blue">广播集</span>，Relay消息可插入广播集重发空隙</div>
            </div>

            <!-- Right: Gateway to bulbs with random delay -->
            <div class="compare-side" data-f="2">
              <div class="diagram">
                <svg viewBox="0 0 300 200" fill="none">
                  <!-- Gateway -->
                  <rect x="120" y="20" width="60" height="40" rx="6" fill="#2a2a35" stroke="rgba(255,255,255,0.1)"/>
                  <rect x="147" y="14" width="6" height="8" rx="2" fill="#3b82f6"/>
                  <!-- Arrows to bulbs -->
                  <line x1="150" y1="60" x2="80" y2="130" stroke="#00d4aa" stroke-width="1.5" opacity="0.6"/>
                  <line x1="150" y1="60" x2="150" y2="130" stroke="#00d4aa" stroke-width="1.5" opacity="0.6"/>
                  <line x1="150" y1="60" x2="220" y2="130" stroke="#00d4aa" stroke-width="1.5" opacity="0.6"/>
                  <!-- Bulbs -->
                  <circle cx="80" cy="140" r="12" fill="#ffd54f"/>
                  <circle cx="150" cy="140" r="12" fill="#ffd54f"/>
                  <circle cx="220" cy="140" r="12" fill="#ffd54f"/>
                  <!-- Random delay labels -->
                  <text x="95" y="115" fill="#666" font-size="10" font-family="JetBrains Mono">+Δt₁</text>
                  <text x="155" y="105" fill="#666" font-size="10" font-family="JetBrains Mono">+Δt₂</text>
                  <text x="195" y="115" fill="#666" font-size="10" font-family="JetBrains Mono">+Δt₃</text>
                </svg>
              </div>
              <div class="desc cn">状态消息回传前增加<span class="highlight">随机延时</span><br>建立消息<span class="highlight-blue">优先级队列</span>，保证控制命令优先Relay<br>降低单位时间内消息数量</div>
            </div>
          </div>
        </div>
      </section>
```

- [ ] **Step 2: Verify in browser**

Reload dev server. Press → to go to slide 2. Verify title, metrics, broadcast set bars on left, gateway-bulbs diagram on right with fragment stepping.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add slide 2 (consistency and real-time with broadcast sets)"
```

---

### Task 6: Slide 3 — Remote OTA (远程OTA)

**Files:**
- Modify: `index.html` (add slide 3)

- [ ] **Step 1: Add slide 3 HTML**

Insert after slide 2's `</section>`:

```html
      <!-- ============================== 3. Remote OTA ============================== -->
      <section class="slide">
        <div class="flex-col gap-3" style="align-items:center">
          <div class="title-large cn">IoT-Mesh 2.0 <span class="gradient-text">远程OTA</span></div>

          <div class="compare mt-2">
            <!-- Left: 1.0 old way -->
            <div class="compare-side" data-f="1">
              <div class="compare-badge old">1.0</div>
              <div class="metric old">单设备耗时 >2hours</div>
              <div class="diagram">
                <svg viewBox="0 0 300 160" fill="none">
                  <!-- Device icon -->
                  <rect x="20" y="50" width="50" height="35" rx="6" fill="#2a2a35" stroke="rgba(255,255,255,0.1)"/>
                  <!-- Arrow -->
                  <line x1="80" y1="67" x2="200" y2="67" stroke="#666" stroke-width="1.5" stroke-dasharray="4 3"/>
                  <polygon points="200,62 210,67 200,72" fill="#666"/>
                  <!-- Mesh messages -->
                  <text x="110" y="55" fill="#666" font-size="11" font-family="Noto Sans SC">固件200k</text>
                  <text x="110" y="90" fill="#666" font-size="11" font-family="Noto Sans SC">Mesh消息25000条</text>
                  <!-- Result icon -->
                  <circle cx="240" cy="67" r="12" fill="#555"/>
                </svg>
              </div>
              <div class="desc cn">使用Mesh网络OTA，消息数量多，引发<span style="color:var(--old)">广播风暴</span><br>Payload短，OTA时间长</div>
            </div>

            <!-- Right: 2.0 new way -->
            <div class="compare-side" data-f="2">
              <div class="compare-badge new">2.0</div>
              <div class="metric new">远程广播 耗时 &lt;10min</div>
              <div class="diagram">
                <svg viewBox="0 0 300 160" fill="none">
                  <!-- Gateway -->
                  <rect x="20" y="50" width="50" height="35" rx="6" fill="#2a2a35" stroke="rgba(255,255,255,0.1)"/>
                  <rect x="42" y="44" width="6" height="8" rx="2" fill="#3b82f6"/>
                  <!-- Routing path nodes -->
                  <circle cx="110" cy="67" r="6" class="ota-path-node"/>
                  <circle cx="160" cy="50" r="6" class="ota-path-node"/>
                  <circle cx="210" cy="67" r="6" class="ota-path-node"/>
                  <!-- Path lines -->
                  <line x1="70" y1="67" x2="104" y2="67" class="ota-path-line"/>
                  <line x1="116" y1="63" x2="154" y2="52" class="ota-path-line"/>
                  <line x1="166" y1="53" x2="204" y2="65" class="ota-path-line"/>
                  <!-- Target bulb -->
                  <circle cx="250" cy="67" r="12" fill="#ffd54f"/>
                  <line x1="216" y1="67" x2="238" y2="67" class="ota-path-line"/>
                </svg>
              </div>
              <div class="desc cn"><span class="highlight">寻路算法</span>，只做路径上转发，有效降低广播风暴<br>使用<span class="highlight-blue">BLE扩展广播</span>，有效Payload增大</div>
            </div>
          </div>
        </div>
      </section>
```

- [ ] **Step 2: Verify in browser**

Navigate to slide 3. Check 1.0 (gray, old) vs 2.0 (green, routing path with dashed animated line). Fragment stepping reveals left then right.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add slide 3 (remote OTA with routing path animation)"
```

---

### Task 7: Slide 4 — Fast Provisioning (秒级配网)

**Files:**
- Modify: `index.html` (add slide 4)

- [ ] **Step 1: Add slide 4 HTML**

Insert after slide 3's `</section>`:

```html
      <!-- ============================== 4. Fast Provisioning ============================== -->
      <section class="slide">
        <div class="flex-col gap-3" style="align-items:center">
          <div class="title-large cn">IoT-Mesh 2.0 <span class="gradient-text">秒级配网</span></div>

          <div class="compare mt-2">
            <!-- Left: 1.0 GATT-based -->
            <div class="compare-side" data-f="1">
              <div class="compare-badge old">1.0</div>
              <div class="metric old">单设备配网 >5s</div>
              <div class="diagram">
                <svg viewBox="0 0 300 260" fill="none">
                  <!-- Arrow marker -->
                  <defs>
                    <marker id="arrowhead-dim" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto">
                      <polygon points="0 0, 8 3, 0 6" fill="#666"/>
                    </marker>
                  </defs>
                  <!-- Lifelines -->
                  <text x="60" y="20" text-anchor="middle" fill="#e8e8f0" font-size="13" font-family="Noto Sans SC">手机</text>
                  <line x1="60" y1="30" x2="60" y2="250" class="seq-lifeline"/>
                  <text x="240" y="20" text-anchor="middle" fill="#e8e8f0" font-size="13" font-family="Noto Sans SC">设备</text>
                  <line x1="240" y1="30" x2="240" y2="250" class="seq-lifeline"/>
                  <!-- Steps -->
                  <line x1="240" y1="55" x2="60" y2="55" class="seq-arrow"/>
                  <text x="150" y="50" text-anchor="middle" class="seq-label-dim">Adv</text>
                  <line x1="60" y1="95" x2="240" y2="95" class="seq-arrow"/>
                  <text x="150" y="90" text-anchor="middle" class="seq-label-dim">Connect</text>
                  <line x1="60" y1="135" x2="240" y2="135" class="seq-arrow"/>
                  <line x1="240" y1="145" x2="60" y2="145" class="seq-arrow"/>
                  <text x="150" y="130" text-anchor="middle" class="seq-label-dim">证书链双向认证</text>
                  <line x1="60" y1="185" x2="240" y2="185" class="seq-arrow"/>
                  <text x="150" y="180" text-anchor="middle" class="seq-label-dim">基于GATT配置</text>
                  <line x1="60" y1="225" x2="240" y2="225" class="seq-arrow"/>
                  <text x="150" y="220" text-anchor="middle" class="seq-label-dim">Terminate</text>
                </svg>
              </div>
              <div class="desc cn">建立GATT直连，基于<span style="color:var(--old)">证书链</span>双向认证</div>
            </div>

            <!-- Right: 2.0 broadcast-based -->
            <div class="compare-side" data-f="2">
              <div class="compare-badge new">2.0</div>
              <div class="metric new">单设备配网 &lt;1s</div>
              <div class="diagram">
                <svg viewBox="0 0 300 260" fill="none">
                  <defs>
                    <marker id="arrowhead-accent" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto">
                      <polygon points="0 0, 8 3, 0 6" fill="#00d4aa"/>
                    </marker>
                  </defs>
                  <!-- Lifelines -->
                  <text x="60" y="20" text-anchor="middle" fill="#e8e8f0" font-size="13" font-family="Noto Sans SC">手机</text>
                  <line x1="60" y1="30" x2="60" y2="250" class="seq-lifeline"/>
                  <text x="240" y="20" text-anchor="middle" fill="#e8e8f0" font-size="13" font-family="Noto Sans SC">设备</text>
                  <line x1="240" y1="30" x2="240" y2="250" class="seq-lifeline"/>
                  <!-- Steps — only 2 -->
                  <line x1="240" y1="80" x2="60" y2="80" class="seq-arrow-new"/>
                  <text x="150" y="75" text-anchor="middle" class="seq-label" fill="#00d4aa">Adv</text>
                  <line x1="60" y1="140" x2="240" y2="140" class="seq-arrow-new"/>
                  <line x1="240" y1="150" x2="60" y2="150" class="seq-arrow-new"/>
                  <text x="150" y="135" text-anchor="middle" class="seq-label" fill="#00d4aa">三元组双向认证</text>
                  <line x1="60" y1="200" x2="240" y2="200" class="seq-arrow-new"/>
                  <text x="150" y="195" text-anchor="middle" class="seq-label" fill="#00d4aa">基于扩展广播配置</text>
                </svg>
              </div>
              <div class="desc cn">使用<span class="highlight-blue">BLE扩展广播</span>，基于<span class="highlight">三元组</span>双向认证</div>
            </div>
          </div>
        </div>
      </section>
```

- [ ] **Step 2: Verify in browser**

Navigate to slide 4. Left shows 5-step GATT sequence (gray), right shows 3-step broadcast sequence (green). The visual difference in step count conveys "fast".

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add slide 4 (fast provisioning with sequence diagrams)"
```

---

### Task 8: Slide 5 — Multi-device Control (多设备控制/配置)

**Files:**
- Modify: `index.html` (add slide 5)

- [ ] **Step 1: Add slide 5 HTML**

Insert after slide 4's `</section>`:

```html
      <!-- ============================== 5. Multi-device Control ============================== -->
      <section class="slide">
        <div class="flex-col gap-3" style="align-items:center">
          <div class="title-large cn">IoT-Mesh 2.0 <span class="gradient-text">多设备控制/配置</span></div>

          <div class="compare mt-2">
            <!-- Left: 1.0 multiple messages -->
            <div class="compare-side" data-f="1">
              <div class="compare-badge old">1.0</div>
              <div class="metric old" style="font-size:.95rem">多条消息、配置时间长</div>
              <div class="diagram">
                <svg viewBox="0 0 300 200" fill="none">
                  <!-- Gateway -->
                  <rect x="30" y="70" width="60" height="40" rx="6" fill="#2a2a35" stroke="rgba(255,255,255,0.1)"/>
                  <!-- Separate arrows (multiple messages) -->
                  <line x1="90" y1="80" x2="200" y2="50" stroke="#666" stroke-width="1.5"/>
                  <polygon points="200,45 210,50 200,55" fill="#666"/>
                  <line x1="90" y1="90" x2="200" y2="90" stroke="#666" stroke-width="1.5"/>
                  <polygon points="200,85 210,90 200,95" fill="#666"/>
                  <line x1="90" y1="100" x2="200" y2="140" stroke="#666" stroke-width="1.5"/>
                  <polygon points="200,135 210,140 200,145" fill="#666"/>
                  <!-- Bulbs (dim) -->
                  <circle cx="230" cy="50" r="12" fill="#555"/>
                  <circle cx="230" cy="90" r="12" fill="#555"/>
                  <circle cx="230" cy="140" r="12" fill="#555"/>
                  <!-- Message labels -->
                  <text x="145" y="72" fill="#666" font-size="10" font-family="Noto Sans SC">msg1</text>
                  <text x="145" y="85" fill="#666" font-size="10" font-family="Noto Sans SC">msg2</text>
                  <text x="145" y="125" fill="#666" font-size="10" font-family="Noto Sans SC">msg3</text>
                </svg>
              </div>
              <div class="desc cn">Payload短，必须先将多设备<span style="color:var(--old)">分组</span>，再进行组控</div>
            </div>

            <!-- Right: 2.0 single message -->
            <div class="compare-side" data-f="2">
              <div class="compare-badge new">2.0</div>
              <div class="metric new" style="font-size:.95rem">一条消息、配置快、同步性好</div>
              <div class="diagram">
                <svg viewBox="0 0 300 200" fill="none">
                  <!-- Gateway -->
                  <rect x="30" y="70" width="60" height="40" rx="6" fill="#2a2a35" stroke="rgba(255,255,255,0.1)"/>
                  <rect x="57" y="64" width="6" height="8" rx="2" fill="#3b82f6"/>
                  <!-- Single fan arrow -->
                  <path d="M90,90 Q150,50 210,50" class="fan-arrow"/>
                  <path d="M90,90 L210,90" class="fan-arrow"/>
                  <path d="M90,90 Q150,130 210,140" class="fan-arrow"/>
                  <!-- Arrow tips -->
                  <polygon points="207,46 215,50 207,54" fill="#00d4aa" opacity="0.8"/>
                  <polygon points="207,86 215,90 207,94" fill="#00d4aa" opacity="0.8"/>
                  <polygon points="207,136 215,140 207,144" fill="#00d4aa" opacity="0.8"/>
                  <!-- Bulbs (lit) -->
                  <circle cx="230" cy="50" r="12" fill="#ffd54f"/>
                  <circle cx="230" cy="90" r="12" fill="#ffd54f"/>
                  <circle cx="230" cy="140" r="12" fill="#ffd54f"/>
                  <!-- Single message label -->
                  <text x="140" y="80" fill="#00d4aa" font-size="11" font-family="Noto Sans SC" font-weight="700">1 msg</text>
                </svg>
              </div>
              <div class="desc cn">使用<span class="highlight-blue">BLE扩展广播</span>，一个消息，直接进行多设备<span class="highlight">控制/配置</span></div>
            </div>
          </div>
        </div>
      </section>
```

- [ ] **Step 2: Verify in browser**

Navigate to slide 5. Left: gateway with 3 separate gray arrows to dim bulbs. Right: gateway with single fan-spread green arrows to lit bulbs. Clear visual contrast.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add slide 5 (multi-device control with fan arrow diagram)"
```

---

### Task 9: Slide 6 — Ultra-low Power (超低功耗)

**Files:**
- Modify: `index.html` (add slide 6)

- [ ] **Step 1: Add slide 6 HTML**

Insert after slide 5's `</section>`:

```html
      <!-- ============================== 6. Ultra-low Power ============================== -->
      <section class="slide">
        <div class="flex-col gap-3" style="align-items:center">
          <div class="title-large cn">IoT-Mesh 2.0 <span class="gradient-text">超低功耗</span></div>

          <div class="compare mt-2">
            <!-- Left: 1.0 high power -->
            <div class="compare-side" data-f="1">
              <div class="compare-badge old">1.0</div>
              <div class="metric old">平均电流 >0.8mA</div>
              <div class="diagram">
                <svg viewBox="0 0 300 120" fill="none">
                  <!-- Dense square wave (continuous scanning) -->
                  <polyline points="10,90 10,30 30,30 30,90 40,90 40,30 60,30 60,90 70,90 70,30 90,30 90,90 100,90 100,30 120,30 120,90 130,90 130,30 150,30 150,90 160,90 160,30 180,30 180,90 190,90 190,30 210,30 210,90 220,90 220,30 240,30 240,90 250,90 250,30 270,30 270,90 290,90" class="wave-old"/>
                  <!-- Label -->
                  <text x="150" y="110" text-anchor="middle" fill="#666" font-size="11" font-family="Noto Sans SC">有限扫描窗口</text>
                </svg>
              </div>
              <div class="desc cn">网关更密集发送控制命令<br>设备功耗高，<span style="color:var(--old)">响应速度慢</span></div>
            </div>

            <!-- Right: 2.0 low power -->
            <div class="compare-side" data-f="2">
              <div class="compare-badge new">2.0</div>
              <div class="metric new">平均电流 &lt;0.2mA</div>
              <div class="diagram">
                <svg viewBox="0 0 300 120" fill="none">
                  <!-- Sparse narrow pulses (periodic broadcast) -->
                  <polyline points="10,90 50,90 50,30 55,30 55,90 120,90 120,30 125,30 125,90 190,90 190,30 195,30 195,90 260,90 260,30 265,30 265,90 290,90" class="wave-new"/>
                  <!-- Glow layer -->
                  <polyline points="10,90 50,90 50,30 55,30 55,90 120,90 120,30 125,30 125,90 190,90 190,30 195,30 195,90 260,90 260,30 265,30 265,90 290,90" class="wave-new-glow wave-animated"/>
                  <!-- Label -->
                  <text x="150" y="110" text-anchor="middle" fill="#00d4aa" font-size="11" font-family="Noto Sans SC">BLE周期广播</text>
                </svg>
              </div>
              <div class="desc cn">网关与设备<span class="highlight">精确时间同步</span><br>设备功耗低，响应时间<span class="highlight">&lt;200ms</span></div>
            </div>
          </div>
        </div>
      </section>
```

- [ ] **Step 2: Verify in browser**

Navigate to slide 6. Left: dense gray square wave (continuous scanning, high power). Right: sparse green pulses with subtle glow animation (periodic broadcast, low power). The visual density difference immediately communicates the power savings.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add slide 6 (ultra-low power with waveform comparison)"
```

---

### Task 10: Final verification and deploy prep

**Files:**
- No new files

- [ ] **Step 1: Run full build**

```bash
cd /Users/linbing/Project/github/IoT-Mesh2.0_HTML
npm run build
```

Expected: `dist/` directory created with `index.html` and bundled JS/CSS. No TypeScript or build errors.

- [ ] **Step 2: Preview production build**

```bash
npx vite preview --open
```

Expected: All 6 slides render correctly. Test:
- ← → switches slides with animation
- ↑ ↓ steps through fragments on each slide
- ESC opens overview grid showing all 6 slides
- Progress bar updates
- Mobile nav buttons work (test by resizing browser to narrow width)

- [ ] **Step 3: Verify all slides content matches PDF**

Walk through each slide and confirm:
1. Reliability — mesh topology + retransmission bars, >99.9%
2. Consistency — broadcast sets + gateway-bulbs, <100ms / <200ms
3. OTA — old path vs routing path, >2h vs <10min
4. Provisioning — 5-step vs 3-step sequence, >5s vs <1s
5. Multi-device — multiple arrows vs fan arrow, multiple vs single message
6. Power — dense wave vs sparse pulses, >0.8mA vs <0.2mA

- [ ] **Step 4: Commit final state**

```bash
git add -A
git commit -m "feat: complete all 6 slides, ready for Vercel deploy"
```

- [ ] **Step 5: Deploy to Vercel**

If the user has Vercel CLI installed:

```bash
cd /Users/linbing/Project/github/IoT-Mesh2.0_HTML
npx vercel --prod
```

Or push to GitHub and connect via Vercel dashboard. Vite projects are auto-detected by Vercel.
