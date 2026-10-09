# ST. MARGARET'S MEMORIAL HOSPITAL — Chaos Click Entry

> *"You visited the hospital's website at 11:47 PM. It looks perfectly normal.
> But the hospital knows you're here. Every click brings the night shift closer."*

A complete interactive horror-comedy website built for the **Chaos Click** hackathon.
Every click the user makes progressively corrupts a beautifully polished hospital
website into a haunted, surreal nightmare — blending *Halloween (1978)* stalking
dread, *Scary Movie* self-aware parody, and *Courage the Cowardly Dog* mismatched-art
surrealism.

---

## 🚀 How to Run Locally

No build step. No dependencies. No framework.

```
# Just open the file:
start index.html                 # Windows
open index.html                  # macOS
xdg-open index.html              # Linux
```

Or serve it locally (recommended — some browsers restrict `file://` audio):

```
# Python 3
python -m http.server 8080

# Node (if you have it)
npx --yes serve -l 8080 .

# PHP
php -S localhost:8080
```

Then open **http://localhost:8080/** in a modern browser (Chrome / Edge / Firefox / Safari).

### Demo shortcut
Press the **R key five times in a row** (not in a text field) to fully reset
corruption back to 0 for live presentations.

Or open the browser Console and call:
```js
ChaosClick.setLevel(5)   // jump straight to corruption level 0-10
ChaosClick.reset()       // full reset
```

---

## 📁 File Structure

```
Halloween/
├─ index.html              All pages: Home, Departments, Doctors, Appointment,
│                          Gallery, Contact, (hidden) Ward 13, (hidden) Privacy.
├─ css/
│  └─ styles.css           Entire design system + all corruption visuals.
├─ js/
│  ├─ corruption.js        CorruptionState engine (state, click tracking, velocity).
│  ├─ audio.js             AudioManager — procedural sound via Web Audio API.
│  ├─ receptionist.js      Receptionist NPC, dialogue bank + trigger system.
│  └─ main.js              UI controller, effects, games, navigation, wiring.
├─ assets/
│  ├─ audio/               (reserved for replacement WAV/MP3 files; see below)
│  └─ img/                 (reserved for replacement images)
└─ README.md               This file.
```

All four JS modules expose a single global:
- `CorruptionState`  — corruption.js
- `AudioManager`     — audio.js
- `Receptionist`     — receptionist.js
- `ChaosClick`       — main.js (debug helpers)

Script load order in `index.html` must stay:
`corruption.js → audio.js → receptionist.js → main.js`.

---

## 🎛️ How to Tune Corruption Thresholds

Open `js/corruption.js` and edit the `THRESHOLDS` array near the top:

```js
const THRESHOLDS = [0, 3, 8, 15, 25, 38, 54, 72, 94, 120, 150];
//                 L0 L1 L2 L3  L4  L5  L6  L7  L8  L9   L10
```

Each value is the **cumulative corruption score** needed to reach that level.
Default progression is about 3 clicks to L1, 150 to L10.

Other tunables in the same file:

| Setting | Default | What it does |
|---|---|---|
| `VELOCITY_WINDOW_MS` | 2000 | Window for "fast clicking" detection |
| `VELOCITY_BONUS_THRESHOLD` | 4 | >4 clicks in 2s → panic-click bonus corruption per click |
| `IDLE_REGEN_MS` | 12000 | Idle time after which sanity slowly regenerates |
| `IDLE_REGEN_RATE` | 0.4 | How much sanity per 12s idle (fractions supported) |

Per-click weights are in `recordClick()` — navigation clicks count 1.5×,
form clicks 1.3×, gallery clicks 1.6×, and fast clicks stack a velocity
bonus of +1 to +3 per click.

---

## 🔊 Audio: Replace / Add Real Sound Files

All audio is currently **fully procedural** (synthesized with the Web Audio API:
oscillators + filtered noise). No external asset files are required. This keeps
the hackathon build self-contained and file-size tiny.

### To swap in real audio files

1. Drop files into `assets/audio/` using these exact names:
   - `hum.wav`        — Layer 0, transformer / AC hum
   - `beeps.wav`      — Layer 1, heart monitor loop
   - `squeaks.wav`    — Layer 2, gurney / footsteps loop
   - `whispers.wav`   — Layer 3, whisper bed
   - `heartbeat.wav`  — Layer 4, heartbeat (triggered from Level 7+)
   - `sting-jump.mp3` — Daredevil jumpscare sting
2. In `js/audio.js` set `MGR.useBuffers = true` (or add the import lines yourself;
   the comments in `setupHum`, `setupBeeps`, etc. mark exactly where to swap the
   procedural source for `<audio>` element playback).
3. Tune per-layer gains in `applyLevelGain()`.

Per-layer activation levels:

| Layer | Activates at | Mix-in curve |
|---|---|---|
| hum       | L1 | 0 → 0.45 → 0.7 → 1.0 |
| beeps     | L2 | 0 → 0.35 → 0.55 → 0.7 |
| squeaks   | L3 | 0 → 0.5 → 0.75 → 0.95 |
| whispers  | L7 | 0 → 0.55 → 0.85 |
| heartbeat | L7 | 70 BPM → 90 → 110 → 130 at L10 |

### Mute / Volume control
The navbar exposes a speaker-icon mute toggle + volume slider. Browser policy
blocks audio before first click, so a toast appears at 1.2s suggesting the
user turn up volume; audio is automatically unlocked on first `pointerdown` /
`keydown` anywhere.

---

## 🎨 Visual: Replace / Add Images

Phase 1 hero + doctor avatars use **CSS gradient + initials** placeholders.
To add real photos:

```css
/* In css/styles.css, replace a doctor card photo: */
[data-doctor="0"] .doctor-photo {
  background: url("../assets/img/doctor-0.jpg") center/cover no-repeat;
}
[data-doctor="0"] .doc-initials { display: none; }
```

Gallery placeholders (`g-img-1`…`g-img-6`) are all gradient backgrounds.
Override with real photos the same way.

`Ward 13` paper look + `Privacy Policy` paper look use pure CSS; no images.

---

## ✨ Add a New Corruption Level Effect

The cleanest place is `onLevelChange()` in `js/main.js`:

```js
function onLevelChange({ oldLevel, newLevel }) {
  // ...existing effects...

  // Your new effect:
  if (newLevel >= 5 && oldLevel < 5) {
    document.body.style.background = "#ff0000";
  }
}
```

Or subscribe to corruption events from any module:

```js
CorruptionState.on('corruption:level', ({ newLevel }) => {
  if (newLevel >= 4) runMyEffect();
});
CorruptionState.on('corruption:click', ({ element }) => { /* per-click */ });
CorruptionState.on('corruption:page', pageId => { /* per-page navigation */ });
CorruptionState.on('corruption:idle-start', s => { /* user stopped clicking */ });
```

**CSS-only effects** are also easy: the `<body>` always has the class
`.corruption-N` (0 to 10). So you can add any rule:

```css
body.corruption-6 .hero-title { animation: wobble 1s infinite; color: red; }
```

---

## 💬 Add Receptionist Dialogue / Triggers

Lines live in `js/receptionist.js`, `DIALOGUE` object. Add a new key:

```js
const DIALOGUE = {
  // ...
  my_new_key: [
    "A normal line.",
    { text: "This gets typed…", delayDelete: 800, final: "…this replaces it." },
    "Random choice 3.",
  ],
};
```

Trigger it manually from anywhere:

```js
Receptionist.say('my_new_key');
```

Or add an auto-trigger:

```js
Receptionist.addTrigger(state => {
  if (state.totalClicks > 200 && !state._saidWow) {
    state._saidWow = true;
    return { key: 'my_new_key', cooldown: 20000 };
  }
  return null;
});
```

Trigger return shape: `{ key, from? ('bot'|'user'), opts?, cooldown? (ms) }`.
Return `null` when conditions are not met.

---

## 🎮 Add New Mini-Games (like Night Rounds)

1. Add markup block in `index.html` inside `<main>`, hidden by default.
2. Add CSS for it in `css/styles.css` under the §18 Night Rounds section.
3. Reveal it from `onLevelChange()` or a trigger at the corruption level you want.
4. Hook start button to your game loop. Call `C.recordInteraction(weight)` for
   any event that should advance corruption.
5. Reward lines to Receptionist via `Receptionist.say('your_key')`.

---

## 🏗️ Architecture Highlights

**No build step.** All scripts are plain ES5-strict-leaning-ES2020. No TypeScript,
no imports, no bundler. Works by double-clicking `index.html`.

**Single source of truth: `CorruptionState`.** Every other module reads and writes
*only* through the engine (`.on() / .emit() / recordClick() / recordPage()…`).
This keeps effects coherent — click velocity boosts the sanity meter, which
drives the heartbeat BPM, which the Receptionist can reference in dialogue,
all because everyone shares the same state object.

**Phase 1 is a real website.** Spend effort on the clean state — asymmetric hero
grid, editorial stats card, properly spaced sections, typed-out forms, footer
hours, a full working Privacy Policy. The betrayal of trust works only if the
first 60 seconds feel like a real hospital marketing site.

**Responsive.** Single column below 680px; two column under 960px; desktop above.
All card grids collapse gracefully; mobile menu exists.

**Accessibility / Motion.** `prefers-reduced-motion: reduce` automatically disables
particle canvas, screen shake, and parallax. Keyboard navigable; all buttons and
links have ARIA labels.

---

**Performance.** All animations use `transform` + `opacity` (GPU-friendly). The
grain overlay is an inline SVG turbulence data-URI (no image fetch). Particle
count caps at 70. Audio uses scheduled ramps rather than per-frame scripting.

---

## 🎨 THEME: Haunted Venice Gothic (Default Visual)

As of this build, the default visual presentation is the **Haunted Venice**
gothic theme, not the original teal/white hospital marketing look. This is a
pure-CSS reskin — zero DOM identifiers, classes, or JavaScript hooks were
changed. The entire corruption engine, NPC dialogue, mini-games, and finale
behave identically to the original spec.

### Palette (design tokens in `css/styles.css` §1)

| Token | Hex | Purpose |
|-------|-----|---------|
| `--void` | `#070a12` | Background, nav bars, footer |
| `--abyss` | `#0d1420` | Card & modal surfaces |
| `--blood` / `--blood-bright` | `#8B0000` / `#C1121F` | Accent, hover, splatter, CTAs |
| `--bone` / `--bone-dim` | `#E8E2C7` / `#b9b18e` | Body copy & secondary text |
| `--ghost-cyan` | `#3a6877` | Receptionist cyan, highlights |
| `--gold-line` | `#6a562e` | Ornate borders, chapter dividers |

### Typography (Google Fonts link in `index.html` <head>)

- **Cinzel** 400–700 — chapter numbers, navbar, buttons, headings (all-caps display)
- **Playfair Display** 400–700 italic — body copy, subtitles, testimonials
- **UnifrakturCook** 700 — finale "DISCHARGE STATUS: NEVER." and Ward 13 headings

### Decorative system (all inline SVG data URIs — no binary assets)

- **Pentagram sigil watermark**: `body::before` fixed-center, opacity tied to `--sigil-opacity` (L0 = 0.05 → L10 = 0.62)
- **6-point blood splatter overlay**: `body::after` fixed full-bleed, opacity tied to `--splatter-opacity` (0 → 1)
- **Chapter 01/02 numbering**: pure CSS counters (`counter-increment: dept`, `counter(feat, decimal-leading-zero)`) — no hardcoded numbers in markup
- **Skull-pile footer graphic**: `.site-footer::after` SVG data-URI
- **Venetian gondola-post hero**: vertical repeating-linear-gradient posts + gold-rule floor
- **Double-inset bronze frames**: `box-shadow: inset 0 0 0 4px #070a12, inset 0 0 0 5px #6a562e` pattern

### Corruption escalation curve (§2 `body.corruption-N`)

Level 0 is already gothic-calm (void background, dim sigil, faint bone text).
Each click-tier progressively:
1. Deepens `--c-bg` toward true `#000000`
2. Raises `--splatter-opacity` + `--sigil-opacity` toward 100%
3. Boosts `--shake-amount` from 0 → 10px
4. Thickens film grain `--grain-opacity` and water stains `--stain-opacity`
5. Accelerates heartbeat BPM (70 → 130) and whisper volumes

### Reverting to the original hospital theme

If you want the original teal/white Phase-1 aesthetic documented in the spec:

1. Swap the `<link>` in `index.html` line 10 back to the hospital font stack:
   ```
   Inter:wght@300;400;500;600;700;800&family=IM+Fell+English:ital@0;1&family=Ewert
   ```
2. Restore the previous `css/styles.css` baseline where `:root` uses
   `--c-bg: #ffffff`, `--c-accent: #0e7490`, and the corruption chain morphs
   from teal-white → sickly-green. All 24 JS hooks and 10 corruption level
   triggers in `body.corruption-0..10` will continue to work unchanged.

---

## 🕹️ Signature Features Checklist

| Feature | Status | Where |
|---|---|---|
| Corruption levels 0–10, smooth escalation | ✅ | `corruption.js` THRESHOLDS |
| Footer clock advances 1 min per page visit | ✅ | `updateFooterClock()` in main.js |
| Word → typewriter flicker L1-2 | ✅ | `flickerWords()` + `.type-flicker` CSS |
| Doctor photo expression change | ✅ | Hovering + night-staff swap L9 |
| Departments nav flicker L1-2 | ✅ | `@keyframes navFlicker` |
| Hospital ambience (hum, beeps, squeaks, whispers, heartbeat) | ✅ | `audio.js` 5 procedural layers |
| Testimonial rewrites L3+ | ✅ | `rewriteTestimonials()` |
| Self-scrolling corridor photo L3+ | ✅ | `.corridor-scene` CSS animation |
| Cursor static trail L3+ | ✅ | `cursor-canvas` |
| Hooded figure in corridor L5 | ✅ | `.hooded-figure` in gallery |
| Cartoon door in gallery (Ward 13) L6 | ✅ | `#ward13-door` |
| Following eyes (painting + chief of cardiology) L5 | ✅ | `setupEyes()` |
| Department dropdown gains creepy options L5 | ✅ | `applyDepartmentDropdown()` |
| Receptionist opens + comments on click behavior L5+ | ✅ | `receptionist.js` triggers |
| Button shivers + deadpan tooltips L5+ | ✅ | `maybeShiver()` |
| Fleeing buttons / cards L7+ | ✅ | `setupFleeing()` |
| Nav items rearrange L7+ | ✅ | `maybeScrambleNav()` |
| Screen vignette + shake L7+ | ✅ | `#vignette-overlay` + `maybeScreenShake()` |
| Mirror section L8+ | ✅ | `addMirror()` |
| Design rots L9 (teal → sickly green, white → stained) | ✅ | CSS vars in `.corruption-9/10` rules |
| Typography mixes horror serifs L9+ | ✅ | `--font-serif` / `--font-display` swap |
| Film grain + water stain overlays 0→100% | ✅ | `#grain-overlay`, `#stain-overlay` |
| All staff → night staff L9 | ✅ | `swapToNightStaff()` |
| Appointment form submits itself L10 + "DISCHARGE STATUS: NEVER." | ✅ | `autoSubmitAtLevel10()` + `triggerFinale()` |
| Ward 13 hidden 1974-style page | ✅ | `#ward13` section + retro CSS |
| Day / Night Shift toggle (partial reset refusal) | ✅ | `setupShiftToggle()` + label degradation |
| Sanity Meter bottom-edge progress bar | ✅ | `#sanity-meter` + label L8 |
| One Sane Element: full Privacy Policy | ✅ | `#privacy` + `.privacy-paper` |
| Night Rounds mini-game L4+ | ✅ | `runNightRounds()` |
| Nurse Station intercom | ✅ | `NURSE_MESSAGES` + `showIntercom()` |
| Daredevil Mode + DO NOT PRESS jumpscare | ✅ | `setupDaredevil()` |
| Cursor URGENT stamp L6+ | ✅ | `stampUrgent()` |
| Respects prefers-reduced-motion | ✅ | Media query + JS guard |
| Mute / volume control always visible | ✅ | Navbar audio controls |
| Volume toast, no autoplay before gesture | ✅ | `#audio-toast` + `unlock()` |
| Hidden demo reset: R × 5 | ✅ | `setupReset()` |
| Fully responsive mobile / tablet / desktop | ✅ | 960px / 680px breakpoints |

---

## 📜 License / Credits

Built for a hackathon. All code is original. All audio is procedurally generated.
All visuals are inline SVG data URIs or CSS gradients. No stock photos or third-party
audio are bundled, so you can safely use and modify this for any use.

Fonts loaded from Google Fonts: Inter (UI), IM Fell English (horror serif),
Ewert (horror display) — all licensed under the OFL.

Happy haunting. 🩸
