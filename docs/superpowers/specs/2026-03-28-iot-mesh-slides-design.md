# IoT-Mesh 2.0 HTML Slides Design Spec

## Overview

Convert pages 10-15 of the IoT-Mesh 2.0 PDF presentation into an interactive HTML slide deck, deployed to Vercel. Reuse the slide engine from `working-with-ai` project. All diagrams rebuilt with HTML/CSS/SVG (no screenshots).

## Source Material

PDF: `蔺冰 IoT-Mesh2.0.pdf` pages 10-15 (2023 开发者大会)

## Architecture

### Project Structure

```
IoT-Mesh2.0_HTML/
├── index.html          # 6 slides content
├── src/
│   ├── main.ts         # Slide engine (from working-with-ai, stripped down)
│   └── style.css       # Dark theme + IoT color scheme
├── public/
│   └── favicon.svg
├── package.json        # Vite + TypeScript
├── tsconfig.json
└── vite.config.ts
```

### Slide Engine

Copied from `working-with-ai/src/main.ts`, keeping only:
- Slide switching with CSS translateX animation
- Fragment system (`data-f` attributes for step-by-step reveal)
- Keyboard navigation (← → slide, ↑ ↓ fragment step, ESC overview)
- Mobile navigation bar
- Overview mode (thumbnail grid)
- Progress bar + slide counter

Removed (not needed for 6 technical slides):
- JPEG compression demo
- Emoji quiz
- AI taste toggle
- iframe overlay

## Visual Design

### Color Scheme

```css
--bg: #0a0a12;       /* dark blue-black */
--fg: #e8e8f0;       /* cool white */
--accent: #00d4aa;   /* tech green — 2.0 new approach */
--accent2: #3b82f6;  /* blue — network/BLE accent */
--dim: #666;         /* 1.0 old approach, muted */
--old: #ff6b6b;      /* old approach metrics, red warning */
```

### Typography

- Space Grotesk — headings, English
- Noto Sans SC — Chinese text
- JetBrains Mono — metrics, technical values

### Layout Pattern

Every slide uses a consistent 1.0 vs 2.0 left-right comparison:
- Left side: dimmed/gray (old approach)
- Right side: bright/green (new approach)
- Title centered at top with key metric

### Device Icons

Gateway (dark gray box) and light bulbs drawn as pure CSS/SVG, no images.

## Slide Content

### Slide 1 — Reliability (可靠性)

- **Title**: `IoT-Mesh 2.0 可靠性` + `控制成功率 >99.9%`
- **Left**: SVG mesh topology — 5-6 circle nodes connected by lines, label "默认开启 Relay"
- **Right**: 3 groups of vertical bars (control commands 1/2/3) with CSS fade-in animation for "network layer retransmission"
- **Fragments**: topology first, then retransmission bars

### Slide 2 — Consistency & Real-time (一致性/实时性)

- **Title** + metrics: `一致性 <100ms  实时性 <200ms`
- **Left**: Two groups of colored vertical bars (广播集1/广播集2) interleaved, labeled "重发间隔"
- **Right**: Gateway → multiple bulbs with arrows, random delay annotation
- **Bottom**: Text explaining multi-broadcast-set insertion and priority queue

### Slide 3 — Remote OTA (远程OTA)

- **1.0 vs 2.0** left-right comparison
- **Left**: Device → arrow → "固件200k / 25000条" in gray, labeled >2hours
- **Right**: Gateway → routing path (3-4 node path line) → bulbs in green, labeled <10min
- **Bottom**: Routing algorithm + BLE extended advertising explanation

### Slide 4 — Fast Provisioning (秒级配网)

- **1.0 vs 2.0** left-right comparison
- **Left**: Sequence diagram — phone/device 5-step interaction (Adv → Connect → certificate chain → GATT config → Terminate), SVG horizontal lines + arrows, labeled >5s
- **Right**: Simplified 2-step (Adv → triplet auth + extended broadcast config), labeled <1s
- Fewer steps = visually conveys "fast"

### Slide 5 — Multi-device Control (多设备控制/配置)

- **Left**: Gateway → multiple separate arrows to each bulb (multiple messages), gray
- **Right**: Gateway → single fan-shaped arrow to all bulbs (one message), green
- **Bottom**: Explanation text

### Slide 6 — Ultra-low Power (超低功耗)

- **1.0 vs 2.0** left-right comparison
- **Left**: Dense gray square waves (有限扫描窗口), labeled >0.8mA
- **Right**: Sparse green narrow pulses (BLE 周期广播), labeled <0.2mA
- Waveforms via SVG path or CSS, right side may have subtle pulse animation

## Deployment

- Vite build → static output
- Deploy to Vercel (zero-config for Vite projects)

## Diagram Approach

Simplified representations — convey core concepts without pixel-perfect reproduction of the original PPT. CSS animations may enhance diagrams where appropriate (e.g., pulse animations for waveforms, fade-in for retransmission bars).
