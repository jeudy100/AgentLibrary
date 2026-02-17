---
name: design-doc
description: Create or update a game design document (GDD) for the current project. Use this skill when the user wants to start a new design document, add or revise sections (systems, mechanics, features), or review the document for completeness. Handles both initial creation and incremental updates to an existing GDD.
argument-hint: "[section-or-topic]"
---

# design-doc

Create, update, and review a game design document (GDD) for the current project.

## Usage

```
/design-doc                          # Create new GDD or show overview of existing one
/design-doc <section-or-topic>       # Add or update a specific section
```

Examples:
- `/design-doc` — Create a new GDD from scratch (interactive) or summarize the existing one
- `/design-doc combat system` — Add or update the combat system section
- `/design-doc review` — Review the full document for gaps and completeness

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for a game design document. Find and read:
>
> 1. **Root CLAUDE.md** — Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Existing GDD** — Search for files matching: `**/DESIGN.md`, `**/GDD.md`, `**/design-doc*`, `**/game-design*`
> 3. **Docs folders** — Check for `docs/`, `doc/`, `documentation/` directories and scan their contents for existing design documents
> 4. **Project type** — Identify game engine, language, framework from project files
> 5. **Source structure** — Get a high-level view of the src/ or game/ directory to understand existing systems
>
> Return: GDD file path (if exists), docs folder path (if exists), game engine/framework, list of implemented systems found in code

From the Explore results, extract:
- Whether a GDD already exists (and its path)
- Whether a docs folder exists (prefer placing the GDD there if so)
- Game engine / framework in use
- Systems already visible in the codebase

---

### Step 2: Determine Operation

Based on the argument and Explore results:

**No argument + no existing GDD?** → Follow "Create" workflow below
**No argument + existing GDD?** → Follow "Review" workflow below
**Argument is "review"?** → Follow "Review" workflow below
**Argument is a section/topic?** → Follow "Update" workflow below

---

### Step 3a: Create Workflow

**Interactive collection — ask the user:**

1. **Game title** and one-line pitch
2. **Genre, perspective, and player count** (e.g., top-down 2D, first-person 3D, side-scroller; singleplayer, co-op, PvP)
3. **Design pillars** — 3-5 guiding principles that filter every design decision (e.g., "high-stakes tension", "player-driven economy", "readable combat")
4. **Player experience goals** — How should the player *feel*? What emotions does the game target?
5. **Reference games** — What existing games inspire this one? "X meets Y" comparisons
6. **Core gameplay loop** — What does the player do in a typical session? What is the moment-to-moment cycle?
7. **Session structure** — How long is a session? What marks the beginning and end? What carries over between sessions?
8. **Key systems** — Which systems should be documented first? Ask the user to list them, and suggest systems that are common for their stated genre

After collecting answers, generate the GDD using the template below. Populate answered sections with detail; leave unanswered sections as stubs with `<!-- TODO -->` markers.

**GDD Template:**

```markdown
# [Game Title]

> [One-line pitch]

---

**Version:** 0.1 | **Last Updated:** [date] | **Status:** Draft

## Table of Contents

<!-- Auto-generate from headings -->

---

## 1. Game Overview

| Field | Value |
|-------|-------|
| Genre | |
| Perspective | |
| Engine | |
| Target Platforms | |
| Player Count | |
| Target Session Length | |

### 1.1 Design Pillars

1. **[Pillar]** — [What it means for design decisions]
2. **[Pillar]** — [What it means for design decisions]
3. **[Pillar]** — [What it means for design decisions]

### 1.2 Player Experience Goals

[How should the player feel? What emotions does the game target?]

### 1.3 Reference Games

| Game | What We Take From It |
|------|---------------------|
| [Game] | [Specific mechanic or feel] |

### 1.4 Non-Goals

What this game deliberately does NOT do:
- [Non-goal 1]
- [Non-goal 2]

---

## 2. Core Loop

### 2.1 Moment-to-Moment Loop

[The fundamental cycle the player repeats — the smallest unit of gameplay]

### 2.2 Session Structure

[How a full session plays out from start to finish]

### 2.3 Meta Loop

[What happens between sessions — progression, unlocks, persistent consequences]

### 2.4 Loop Diagram

<!-- Flowchart or text diagram showing how loops nest -->

---

## 3. Systems

<!-- Each system follows this structure -->

### 3.1 [System Name]

**Purpose:** [What this system does for the player experience]

**Core Mechanics:**
- [Mechanic and how it works]

**Rules & Parameters:**
| Parameter | Value | Notes |
|-----------|-------|-------|
| [Tunable] | [Value] | [Why this value] |

**Feedback:** [How the game communicates this system's state to the player — visual, audio, haptic]

**Affects:** [Systems this one feeds into]
**Affected By:** [Systems that feed into this one]

<!-- Repeat ### for each system -->

---

## 4. Content

### 4.1 [Content Category]

<!-- Tables, lists, or descriptions of game content -->
<!-- Examples: weapons, items, enemies, characters, levels, abilities — adapt to genre -->

---

## 5. Level / World Design

### 5.1 Design Philosophy

[Overall approach to level layout, pacing, and spatial design]

### 5.2 [Level / Area / Zone Name]

**Overview:** [Purpose, mood, difficulty]
**Layout:** [Key landmarks, flow, points of interest]
**Content:** [What the player finds here — enemies, loot, events]

---

## 6. Narrative and World

### 6.1 Setting

[Where and when the game takes place]

### 6.2 Story

[Narrative arc, key beats — only if relevant to gameplay]

### 6.3 Characters / Factions

[Key characters or factions and their role in gameplay]

### 6.4 Lore

[Worldbuilding that affects player understanding — keep gameplay-relevant]

---

## 7. User Interface and UX

### 7.1 HUD

[Every persistent on-screen element and what it communicates]

### 7.2 Menu / Screen Flow

[Navigation structure — diagram or description of all screens]

### 7.3 Feedback Systems

[How the game communicates state changes: hit markers, damage numbers, alerts, audio cues]

### 7.4 Accessibility

[Colorblind modes, remapping, difficulty options, subtitle support]

---

## 8. Art Direction

### 8.1 Visual Style

[Art style, color palette, mood references]

### 8.2 Character Design

[Silhouette language, visual distinction rules]

### 8.3 Environment Style

[Lighting, color theory, visual storytelling]

### 8.4 VFX

[Particle effects, screen effects, impact feedback]

---

## 9. Audio Design

### 9.1 Music

[Genre, mood, adaptive/dynamic triggers]

### 9.2 Sound Effects

[Key SFX categories and their role in gameplay feedback]

### 9.3 Spatial Audio

[How audio communicates position, distance, threat — especially important for certain perspectives]

---

## 10. Technical Notes

### 10.1 Platform and Performance

[Target specs, frame rate, resolution targets]

### 10.2 Networking

[Client-server model, tick rate, authority model, latency tolerance — if multiplayer]

### 10.3 Persistence / Save System

[What persists, when it saves, server vs. client authority]

### 10.4 System Dependency Map

[Diagram or table showing how major systems connect]

```
[System A] → affects → [System B]
[System B] → affects → [System C]
[System C] → affects → [System A]  (feedback loop — label positive/negative)
```

---

## 11. Economy and Progression

<!-- Include if the game has currencies, trading, unlocks, or persistent progression -->

### 11.1 Currencies

| Currency | Source (Faucet) | Spend (Sink) |
|----------|----------------|--------------|
| [Currency] | [How earned] | [How spent] |

### 11.2 Progression Path

[How the player grows — levels, unlocks, skill trees, reputation]

### 11.3 Balance Targets

[Key ratios, pacing curves, time-to-X targets]

---

## 12. Monetization

<!-- Include if applicable -->

[Business model, what is sold, pay-to-win policy]

---

## 13. Analytics and Live Operations

<!-- Include if applicable -->

### 13.1 Key Metrics

[Session length, retention, economy health — what you track and why]

### 13.2 Post-Launch Plan

[Content cadence, seasonal structure, update philosophy]

---

## Open Questions

- [ ] [Unresolved design question]

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | [date] | Initial draft |
```

**Section applicability:** Not every game needs every section. After gathering the genre and scope from the user, omit sections that don't apply (e.g., skip Monetization for a hobby project, skip Networking for singleplayer-only, skip Economy for games without currencies). Mark omitted sections with a brief note: `<!-- Omitted: not applicable for [reason] -->`.

**Genre-specific systems:** Based on the user's stated genre, suggest systems they likely need but haven't mentioned. Examples:
- Extraction shooters: raid entry/exit, gear loss on death, extraction points, stash management, loot tables
- RPGs: character classes, skill trees, dialogue systems, quest structures
- Platformers: movement physics, level hazards, checkpoint systems
- Strategy: resource gathering, tech trees, unit production, fog of war
- Roguelikes: run structure, procedural generation rules, permanent unlocks

---

### Step 3b: Update Workflow

1. Read the existing GDD file.
2. Find the section matching the user's topic (fuzzy match on heading text and numbering).
   - If the section exists → rewrite/expand it based on the user's input.
   - If the section doesn't exist → add it under the appropriate parent heading using the system template structure (Purpose, Core Mechanics, Rules & Parameters, Feedback, Affects/Affected By).
3. Ask the user targeted questions about the section:
   - Purpose, core mechanics, rules/parameters, feedback, interactions with other systems
4. Write the updated section back into the GDD, preserving the rest of the document.
5. After updating:
   - Check if any **Affects / Affected By** references point to sections that don't exist yet and list them as stubs to fill later.
   - Update the **Changelog** table at the bottom with the change.
   - Update the **Version** and **Last Updated** fields in the header.

---

### Step 3c: Review Workflow

1. Read the full GDD.
2. Evaluate each section for:
   - **Completeness** — Is it more than a stub? Does it explain mechanics clearly enough to implement?
   - **Consistency** — Do cross-references between systems align? Are there contradictions?
   - **Coverage** — Are there systems visible in the codebase that aren't documented?
   - **Specificity** — Are there vague statements that should have concrete values? (e.g., "damage will feel good" → needs numbers)
   - **Dependency clarity** — Are Affects/Affected By fields filled in? Are feedback loops identified?
   - **Open questions** — Are TODO markers or open questions still unresolved?
3. Output a review summary:

```
## GDD Review

**Version:** [current] | **Sections**: X total, Y complete, Z stubs

### Complete
- [Section] — well documented

### Needs Work
- [Section] — missing [specific gap]

### Missing
- [System found in code but not in GDD]

### Vague (Needs Specificity)
- [Section] — "[vague statement]" needs concrete values

### Contradictions
- [Description of conflict between sections]

### Unresolved Dependencies
- [System A] references [System B] but [System B] is a stub

### Open Questions
- [ ] [Unresolved item]
```

---

### Step 4: Confirm and Save

- Show the user a summary of changes before writing.
- Write the file. Default path logic:
  1. If a GDD already exists → update in place
  2. If a `docs/` (or `doc/`, `documentation/`) folder exists → write to `docs/DESIGN.md`
  3. Otherwise → write to `DESIGN.md` at project root
- If the user prefers a different location, respect that.

---

### Step 5: Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.
