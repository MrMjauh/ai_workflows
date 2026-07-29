---
name: game-design-doc
description: Produce a game design document as a single self-contained HTML page (never Markdown), following the required GDD template — concept document (name, genre, game elements, technical specs, game play outline, key features) plus modular design document (design guidelines, definitions, game flowchart, player properties and rewards, UI) — with mood boards and authored illustrations interleaved (SVG diagrams, CSS palettes, lighting and texture studies, silhouette scale charts, HUD wireframes, mermaid flowcharts). Use when the user wants a GDD, game concept doc, game pitch, mood board, art direction doc, or asks to flesh out / spawn ideas for a game concept.
---

# Game Design Doc + Mood Board

**The deliverable is one self-contained `.html` file** holding the entire design document with its visuals interleaved — written sections and mood boards in the same page. Publish it with the `Artifact` tool.

**Never produce a Markdown GDD.** No `GDD.md`, no `.md` companion, no "here's the markdown version too" — a second file drifts out of sync and splits the document. If the user asks for something they can paste elsewhere, point them at the HTML file or the artifact URL. The only exception is an explicit, direct request for Markdown.

## Hard constraint: no image generation

There is no text-to-image tool. Do **not** promise concept art, and never emit `<img>` pointing at a remote URL — the artifact CSP blocks external hosts, so it renders as a broken box.

Everything visual is *authored*: SVG, CSS, mermaid. Part 3 below has working techniques for palettes, textures, lighting studies, silhouette scale charts, cross-sections, HUD wireframes and diagrams. They produce genuinely good boards — not a consolation prize. User-supplied reference images get base64'd into `data:` URIs.

Say this once, plainly, early. Don't re-apologize for it.

---

# Step 1 — Establish the concept before writing anything

A GDD generated from a one-line prompt is filler. Pin down five things first, from the conversation or by asking:

- **Game elements** — the basic activities the player does for fun, as verbs. This is the spine; without it there's nothing to fill the template with.
- **Genre and comparables** — 2–3 real games, and explicitly *what is taken from each*.
- **The setting** — and whether it should differ from the comparables' settings. Users citing reference games usually want the loop, not the world. Ask if unclear; getting this wrong wastes the whole doc.
- **Tone** — cozy / tense / comic / melancholy. Drives the entire visual half.
- **Technical target** — 2D or 3D, view, platform, device. These four decide most of the rest.

At the brainstorm stage, pitch 3 distinct concepts in a few sentences each *before* committing. Don't write three full docs.

One round of questions maximum, then commit. Missing details become stated assumptions, not blockers.

---

# Step 2 — Write the document content

The section order in Part 1 below **is** the required template. Follow it. Genre changes what goes *inside* a section, never whether the section exists.

Non-negotiable regardless of genre:

- **Every template field gets a decided answer.** No "TBD" in the title block or technical specs.
- **Name every entity.** No `enemy_1` / `enemy_2`. "The Lazarus Fighter carries heavier armour than the Apollo Fighter" — named, with a consequence.
- **Answer "why is all this fun?" honestly.** A weak answer there is a finding to report, not a line to dress up.
- **Winning, losing, and level transition stated literally**, and consistent between the game play section and the design definitions.
- **Every player property names its player-facing feedback** when it changes.
- **The worst-layout test in the UI section** — describe the worst plausible layout, then say whether yours avoids it.

Concrete numbers throughout — hours per level, metres of map, seconds per cycle. Invented-but-plausible gives the reader something to argue with; vague gives them nothing.

---

# Step 3 — Build the page

Load the `artifact-design` skill for design calibration, then write the HTML file and publish it. Interleave visuals with the sections they support — don't quarantine the mood board into an appendix. Part 2 has the page structure; Part 3 has the techniques.

## Step 4 — Hand off

Report: the artifact URL, the `.html` file path, the answer to "why is all this fun?" in one sentence, and **the weakest part of the concept**. Give the honest structural concern — that's the part worth the user's attention, and flattery there costs them months.

## Updating

Re-read the HTML file before editing. Redeploy from the same file path to hold the URL stable, and keep the favicon unchanged unless the concept genuinely pivots.

---
---

# PART 1 — GDD STRUCTURE

Two parts:

- **Concept Document** (§1–§4) — what the game is, what it runs on, how it plays.
- **Design Document** (§5–§9) — how GameObjects behave, how they're controlled, what their properties are. The mechanics. This part is *modular*: one game can carry several Design Documents (one per mode, per character class, per major system) attached to a single Concept Document. If the game warrants that, emit them as sibling sections with their own numbering, not one bloated §5.

Sections marked **extension** are not in the template. Add them when the doc is a pitch that needs to survive scrutiny; drop them when the user wants the template and nothing else.

## PART A — CONCEPT DOCUMENT

### 1. Title block

A short definition list, not prose. Every field gets a real answer:

| Field | Guidance |
|---|---|
| **Game Name** | Commit to one. A working title in quotes is fine; "TBD" is not. |
| **Genre** | Existing genre words the reader already knows, plus a modifier: "colony sim with roguelike run structure". |
| **Game Elements** | The basic activities the player does for fun — verbs. "Dig, pipe, insulate, survive the night." Not "exploration and strategy". |
| **Player** | The number of players at once. Say the exact number and the mode: "1 (single-player), or 2–4 co-op over LAN". |

### 2. Technical specs

Same treatment — a table, one line each, every field decided:

| Field | Guidance |
|---|---|
| **Technical Form** | 2D or 3D. If it's a mix ("3D world, 2D sprite actors"), say which and why. |
| **View** | The camera the player experiences the game from: first-person, third-person over-shoulder, fixed isometric, 2D side-on, top-down orthographic. Name it precisely — this decides the art budget. |
| **Platform** | iOS, Android, Mac, PC. List the launch platform first and mark the rest as later. |
| **Language** | C#, C++, Ruby, Java. State the engine alongside it (Unity → C#, Unreal → C++/Blueprints, Godot → GDScript/C#). |
| **Device** | PC, Mobile, Console. Note the input device per target, because that constrains §9. |

Flag any conflict between fields out loud. Twin-stick controls on a mobile target is a design problem, not a table entry.

### 3. Game play — required

A descriptive paragraph — several, if needed — that makes the reader imagine they are *playing*. Second person, present tense, specific nouns.

The rule from the template, stated as a test: **no generic placeholder names, ever.** Not "enemy_1 has more hit points than enemy_2." Instead: "The Lazarus Fighter carries heavier armour than the Apollo Fighter, so it survives the minefield but arrives too slow to catch the convoy." Every entity in the doc gets a name and a consequence.

Then the **Game Play Outline**, in this order. It varies by game type — cut what genuinely doesn't apply and say you cut it:

- **Opening the game application** — what the player sees in the first 10 seconds, before any menu choice.
- **Game options** — what's configurable, and what deliberately isn't.
- **Story synopsis** — the premise in a short paragraph. Note delivery method and whether it's skippable.
- **Modes** — campaign, endless, versus, tutorial. One line each on what makes each distinct.
- **Game elements** — the verbs from §1, expanded: what each activity feels like to do repeatedly.
- **Game levels** — level/zone/act list with what changes across them. A table works well.
- **Player's controls** — the summary; the full mapping lives in §9.
- **Winning** — the exact condition. Be literal.
- **Losing** — the exact condition, plus the penalty and what the player restarts from.
- **End** — what happens after the win: credits, sandbox, NG+, leaderboard.
- **Why is all this fun?** — the honest answer. This is the highest-value line in the document. If the answer is "because it's like [game] but with [feature]", say that plainly; a weak answer here is a finding to report, not a line to dress up.

### 4. Key features

A list of game elements attractive to the *player* — phrased as what the player gets, not what the team builds. "Rebuild any wall you've broken, mid-fight" beats "destructible environment system". 5–8 items. Each one must be something a stranger would repeat to a friend.

**Extension — comparables.** A table. Rows are dimensions (pressure, expansion model, tone, endgame); columns are 2–3 real games plus this game. The this-game column must differ meaningfully in most rows. If it doesn't, that's the finding — report it.

## PART B — DESIGN DOCUMENT

Modular. Describes how GameObjects behave, how they're controlled, and their properties.

### 5. Design guidelines — required

The creative restrictions that must be respected, plus a brief statement of the overall design goal.

Write restrictions as prohibitions, because a constraint you can't violate is a constraint doing work: "No fail state the player can't undo within 30 seconds." "No text on screen during combat." "Nothing requires a second stick." Include real-world constraints too — team size, budget, accessibility targets, platform store rules.

Then one or two sentences on the overall goal of the design, in plain language.

### 6. Game design definitions — required

The definitions that establish the game play. Precise and testable:

- **How the player wins** — the literal condition.
- **How the player loses** — the literal condition.
- **How the player transitions between levels** — the trigger, the load/transition treatment, what carries over and what resets.
- **The main focus of the gameplay** — one sentence naming the activity that occupies the majority of playtime.

These must not contradict §3. Cross-check before finishing.

### 7. Game flowchart — required

A visual of how game elements and their properties interact, covering at minimum:

```
Menu → Synopsis → Game Play → Player Control → Game Over (win / lose) → back to Menu
```

Build it as mermaid (`graph TD`, or `stateDiagram-v2` when screen state matters). Represent **Objects, Properties, and Actions** — and give every node a reference number tying it to where it lives in this mechanics document (`[3.2 Player Control]`), so the flowchart and the prose stay navigable against each other.

Systems that touch nothing else on the chart are content, not systems.

### 8. Player definition — required

Three subsections, all three present.

**8a. Player definitions** — short descriptions defining the player. A suggested list:

- **Health** — how much, how it's represented, how it recovers.
- **Weapons** — named, with what makes each distinct.
- **Actions** — the full verb list: move, jump, interact, and whatever is game-specific.

**8b. Player properties** — each property is affected by the player's actions or by interaction with other game elements. For every property state: its range, what changes it, and — required — **the feedback the player receives when it changes.** A property that changes with no audible, visible, or haptic signal is a bug in the design. Say what the signal is: screen edge vignette, controller rumble, the lantern flame guttering.

A table works: Property | Range | Changed by | Effect on play | Feedback.

**8c. Player rewards (power-ups and pick-ups)** — every object that affects the player positively. For each: what effect it causes, how the player uses it, how long it lasts, and how the player recognises it on screen. Named objects only.

### 9. User interface (UI) — required

How the user controls the game.

- **Control mapping** — a table of physical button/key → in-game action, per input device from §2. Include the menu/pause layer, not just gameplay.
- **Why these buttons** — the reasoning. Which actions sit under the fingers that rest there.
- **The worst-layout test** — from the template, and worth doing literally: describe the worst plausible layout for this game, then state whether your UI avoids that failure and how. If your layout *is* close to it, that's the finding.
- **Screen layout** — HUD wireframe with each region labelled by what it's for, plus what's deliberately absent.
- **Accessibility** — remapping, hold-vs-toggle, colourblind-safe signals, text size.

**Extension — scope and MVP.** Engine recommendation with a reason grounded in the team. The vertical slice: what's in it, timeline, headcount, and the single question it answers. The cut list — what dies first under pressure.

**Extension — risks.** Real risks with mitigations, ranked by how likely they are to kill the project. Include at least one of the form "this might not be fun because ___". Skip generic entries like "scope creep" unless you say something specific about *this* project's scope creep.

## Writing style

- Numbers over adjectives. Invented-but-plausible beats vague — the reader can argue with a number.
- **Name every entity.** The template's central rule. A named building, fighter, or resource is real; "a mid-tier production structure" is not. No `enemy_1`.
- Every property change names its player-facing feedback.
- Present tense, active voice. Second person for game play prose.
- Tables for anything comparative, or for any template field list.
- No hedging stacks ("might potentially possibly"). Commit; the user can push back.
- Length follows scope: pitch 2–3 pages, buildable spec 8–12, vision doc as long as it needs.

---
---

# PART 2 — PAGE STRUCTURE

One HTML page holding the whole GDD, visuals interleaved with the sections they support. Section order follows Part 1.

```
Hero              game name · genre · game elements as tone words · palette strip as a thin band
Nav               sticky in-page anchors, one per numbered section

PART A — CONCEPT
§1 Title block    definition table: name · genre · game elements · players
§2 Tech specs     definition table: form · view · platform · language · device
§3 Game play      prose, second person + outline (opening → options → synopsis →
                  modes → elements → levels → controls → win → lose → end → why fun)
                  · lighting studies and material cards sit alongside the synopsis
                  · level table
§4 Key features   cards, player-facing phrasing

PART B — DESIGN
§5 Guidelines     prohibitions as a list, visually distinct callout
§6 Definitions    win / lose / level transition / main focus — compact table
§7 Flowchart      mermaid graph TD, full width, nodes numbered to sections
§8 Player         8a definitions · 8b properties table (+ stateDiagram for stateful ones)
                  · 8c rewards cards · silhouette scale study
§9 UI             HUD wireframe + control-map table + worst-layout note
(extensions)      comparables · scope/MVP callout · ranked risks
```

What earns its place on the page:

| Element | Technique | Establishes |
|---|---|---|
| Hero: game name, genre, tone words | Typography + palette band | The feel, in three seconds |
| Title block + technical specs | Two compact tables | The hard facts, above the fold |
| **Game flowchart** (menu → synopsis → game play → player control → game over) | Mermaid `graph TD` with numbered nodes | How the whole game hangs together — required |
| Palette | CSS swatches, hex **+ role** + frame ratio | Art direction, concretely |
| Lighting studies | Gradient panels, one shared silhouette | Time of day, mood range |
| Material cards | Layered gradients + `feTurbulence` | Surface language |
| Silhouette scale chart | Inline SVG on a ground line | World scale |
| Environment cross-section or top-down | Annotated SVG | Spatial structure, zone adjacency |
| **HUD wireframe + control map** | Crude SVG boxes; button → action table | What the player looks at and presses — required |
| Player properties | Table + `stateDiagram-v2` for any stateful one | What changes, and the feedback for it |
| Level / mode progression | Mermaid graph + table | The long arc |
| Reference touchstones | Text cards + representative swatch | Where the aesthetic comes from |

Layout rules:

- **Commit to the tone.** The page's palette should *be* the game's palette — a cozy game's doc should feel cozy. That's the cheapest available proof the direction works. A board that looks like a generic dashboard has failed regardless of content.
- Every figure gets a real `<figcaption>` stating what it establishes. Decoration without a claim is noise.
- Names must be consistent across the whole page — same palette names, fighters, buildings, level numbers.
- One column, `max-width: 68ch` for prose. Let figures and tables break wider — `width: min(100%, 1100px)` in a centered grid.
- Tables and wide SVGs go in `overflow-x:auto` wrappers. The body never scrolls sideways.
- Section rhythm: generous vertical space, hairline rules, no boxes-inside-boxes.
- Type: one display face for headings, one readable face for body. Long-form prose wants ~1.65 line-height and 17–18px.
- Theme-aware per the Artifact tool's requirements — unless the game's identity demands one committed look, in which case build that and say so.

---
---

# PART 3 — VISUAL RECIPES

Techniques for authoring game-doc visuals with no image generation. All are self-contained — no external hosts, per the artifact CSP.

Treat these as starting points, not templates to paste verbatim. Retune every value to the game's tone.

## Palette swatches

Every swatch carries a hex **and a role**. The role is the useful part — "ice shadow" tells an artist where it goes; "#2a3a4f" does not.

```html
<div class="palette">
  <figure style="--c:#0d1520"><figcaption><b>Basin Night</b><code>#0D1520</code><span>ambient base, 60% of frame</span></figcaption></figure>
  <figure style="--c:#ff9f45"><figcaption><b>Lamp Warm</b><code>#FF9F45</code><span>player-built light, sole warm source</span></figcaption></figure>
</div>
```
```css
.palette{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:1px}
.palette figure{margin:0}
.palette figure::before{content:"";display:block;aspect-ratio:1;background:var(--c)}
.palette figcaption{padding:.6rem .2rem;font-size:.78rem;line-height:1.5}
.palette b{display:block} .palette code{opacity:.7} .palette span{display:block;opacity:.55}
```

Order swatches by frame dominance, not by hue. State the intended ratio (60/30/10) — it's the single most useful art-direction fact in the doc.

## Texture / material cards

Layered gradients plus an SVG turbulence filter. Define the filter once, reuse by id.

```html
<svg width="0" height="0" style="position:absolute">
  <filter id="grain"><feTurbulence type="fractalNoise" baseFrequency="0.85" numOctaves="3"/>
    <feColorMatrix type="saturate" values="0"/></filter>
  <filter id="rough"><feTurbulence type="fractalNoise" baseFrequency="0.02 0.14" numOctaves="4"/>
    <feDisplacementMap in="SourceGraphic" scale="18"/></filter>
</svg>
```
```css
.mat{position:relative;aspect-ratio:3/2;overflow:hidden;border-radius:2px}
.mat::after{content:"";position:absolute;inset:-50%;filter:url(#grain);opacity:.18;mix-blend-mode:overlay}
.mat--ice{background:
  linear-gradient(160deg,#1b2b3d,#0d1520 60%),
  repeating-linear-gradient(72deg,#ffffff10 0 2px,transparent 2px 22px)}
.mat--steel{background:
  linear-gradient(180deg,#3b4048,#22262c),
  repeating-linear-gradient(90deg,#ffffff08 0 1px,transparent 1px 7px)}
```

`baseFrequency` low (0.01–0.05) = large organic blotches; high (0.6–1) = fine film grain. Anisotropic values (`0.02 0.14`) give directional grain — good for brushed metal, wood, wind-scoured snow.

## Lighting studies

The same scene silhouette under different light conditions, side by side. This communicates art direction faster than any prose.

```css
.light{aspect-ratio:16/9;position:relative}
.light--dawn{background:linear-gradient(#0a1626 0%,#2a3f5c 45%,#c98a5a 78%,#e8b57e 100%)}
.light--storm{background:linear-gradient(#1a1f26 0%,#3a4048 60%,#585e66 100%)}
.light--aurora{background:linear-gradient(#050b14 0%,#0d2438 50%,#1a4a44 74%,#2f7d63 92%)}
/* Overlay one shared SVG silhouette (dark, opaque) on each panel. */
```

Keep the silhouette identical across panels — only the light changes. That's the whole point of the comparison.

## Silhouette + scale study

An SVG line-up on a shared ground line: player, then key structures ascending. Nothing establishes a world's scale faster. Pairs with §8.

```html
<svg viewBox="0 0 900 240" role="img" aria-label="Scale comparison">
  <line x1="0" y1="210" x2="900" y2="210" stroke="currentColor" stroke-opacity=".25"/>
  <!-- player: 1.8m => 18px/m -->
  <rect x="30" y="178" width="11" height="32" rx="5" fill="currentColor"/>
  <text x="20" y="228" font-size="11" fill="currentColor" opacity=".6">Player · 1.8m</text>
  <path d="M120 210 v-72 h56 v72z" fill="currentColor" opacity=".8"/>
  <text x="118" y="228" font-size="11" fill="currentColor" opacity=".6">Burner · 4m</text>
</svg>
```

Declare the px-per-metre ratio in a comment and hold it across every element. Label each with its real-world dimension.

## Environment cross-section / top-down

Annotated SVG. A cross-section sells vertical layering (surface / substrate / buried); a top-down sells zone adjacency and travel distance. Pick whichever the game's expansion model actually uses.

Include a scale bar and a north arrow or depth axis. Annotate with leader lines to labels rather than cramming text into the drawing.

## HUD / screen wireframe

For §9. Deliberately crude — grey boxes, thin strokes, labels. A polished mockup invites arguments about the wrong things at this stage.

```html
<svg viewBox="0 0 640 360">
  <rect width="640" height="360" fill="none" stroke="currentColor" stroke-opacity=".3"/>
  <rect x="16" y="16" width="150" height="46" fill="currentColor" fill-opacity=".08"/>
  <text x="24" y="44" font-size="12" fill="currentColor">warmth meter</text>
  <rect x="200" y="300" width="240" height="44" fill="currentColor" fill-opacity=".08"/>
  <text x="208" y="327" font-size="12" fill="currentColor">hotbar · 8 slots</text>
</svg>
```

Annotate what each region is *for*, and note what is deliberately absent — an empty HUD is a design claim worth stating.

## Diagrams — mermaid

Artifacts render mermaid natively. Use ```mermaid fences in markdown, `<pre class="mermaid">` in HTML.

- **Game flowchart (§7, required)** → `graph TD`: menu → synopsis → game play → player control → game over (win / lose) → menu. Number each node to its section, e.g. `PC["3.2 Player Control"]`.
- Core loop → `graph LR` cycle
- Level / mode progression → `graph TD`
- Player property state (e.g. warmth, alert level) → `stateDiagram-v2`
- Session pacing → `gantt`

Keep node labels to 1–4 words. Mermaid is for structure; put the prose in the caption.

## User-supplied reference images

Remote URLs are blocked. Base64 and inline:

```bash
base64 -w0 ref.jpg > ref.b64   # then embed as data:image/jpeg;base64,...
```

Watch total page weight — downsize past ~200KB per image first. If a set of references is too heavy, describe them as text touchstone cards with representative swatches instead.
</content>
