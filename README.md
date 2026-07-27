# HannaBeads Commission Pipeline

An interactive, multi-agent skill for generating storyline commission content for HannaBeads — a first-person bead crafting simulator where players run a small Hama Beads workshop.

## What is HannaBeads?

HannaBeads is a cozy crafting game where players:

- **Accept commissions** from clients who share their story and request a specific bead craft
- **Search for patterns** in a library of pixel art designs
- **Place beads** on pegboards by dragging them from jars
- **Iron the design** to fuse the beads together
- **Deliver** the completed work and earn money + reputation

Storylines are narrative arcs where characters share their story across 5-10 commissions. Each commission is a personal message from the character, explaining their story while asking for a specific craft.

## Key Concepts

### Required Axes (not Required Pattern)

Commissions specify **required axes** — an array of Theme/Pattern/SubPattern tags the delivered pattern must include. For simple commissions, this is one entry (e.g., `CAT:ORANGE`). For composites, it's multiple entries (e.g., `HEART` + `CAT:ORANGE` + `CAT:BLUE`).

When the same Pattern axis appears twice (e.g., two cats), each must have a distinct SubPattern to specify which color variant is needed.

### Axis Gaps

When the story requests something that doesn't map to any existing pattern axis (e.g., "skateboard"), the pipeline reports an **axis gap**. The user decides how to handle it:

1. **Revise story** — rewrite the narrative to use existing axes
2. **Assume axis will be added** — log in `GDD_TODO.md` and proceed
3. **Simplify** — replace with something that has an existing axis
4. **Skip** — remove the commission step entirely

### Complexity Tiers

| Tier | Payout Range | Max Colors | Composites |
|------|-------------|------------|------------|
| SIMPLE | 50-100 | 3 | No |
| MODERATE | 80-150 | 5 | No |
| MODERATE-COMPLEX | 100-180 | 7 | Yes (2 base) |
| COMPLEX | 150-200 | 8 | Yes (3 base) |

Complexity is **story-driven** — the StoryWriter decides the starting point and progression. It must never go backwards (SIMPLE → MODERATE → MODERATE-COMPLEX → COMPLEX), but a storyline can start at MODERATE or never reach COMPLEX.

### Color Availability

- If `minReputationRequired` < 20: only 5 starting colors (White, Black, Red, Blue, Yellow), max 3 colors
- If `minReputationRequired` >= 20: all 30 Artkal bead colors available

### Axis Reference (v1)

8 themes with 6 pattern categories each (48 total). The Pattern axis is a category — multiple pixelart designs can share one Pattern value (e.g., Cat Face, Cat Sleeping, Cat Sitting all use `CAT`).

| Theme | Pattern Axis |
|-------|-------------|
| **ANIMALS** | CAT, DOG, BIRD, BUNNY, FROG, BUTTERFLY |
| **PLANTS_NATURE** | FLOWER, MUSHROOM, CACTUS, TREE, SUNFLOWER, RAINBOW |
| **FOOD_MEALS** | PIZZA, CUPCAKE, SUSHI, RAMEN BOWL, COFFEE CUP, ICE CREAM |
| **GAMING_GEEK** | GAME CONTROLLER, PIXEL HEART, ROBOT, D20 DIE, RUBIK'S CUBE, ARCADE CABINET |
| **SPACE_ASTRONOMY** | PLANET, ROCKET, STAR, MOON, ALIEN, ASTRONAUT |
| **MAGIC_MYSTIC** | CRYSTAL BALL, WAND, POTION BOTTLE, DRAGON, UNICORN, SPELL BOOK |
| **MUSIC_AUDIO** | GUITAR, MUSIC NOTE, HEADPHONES, MICROPHONE, VINYL RECORD, DRUM |
| **SHAPES_SYMBOLS** | HEART, STAR, LIGHTNING BOLT, DIAMOND, ARROW, SNOWFLAKE |

**SubPattern** is `Pattern:Color` format — e.g., `CAT:ORANGE`, `FLOWER:RED`, `HEART:PINK`. When the same Pattern appears twice in a commission, each must have a distinct SubPattern.

### Reputation

Reputation is the **central progression metric** in HannaBeads. It affects nearly everything:

- **Payouts:** All commission payouts are multiplied by `1.0 + (reputation / 100)`. Higher rep = more coins for the same work.
- **Storyline gates:** Each storyline chapter has a reputation threshold. Reaching it unlocks the next commission in the arc.
- **Color access:** At rep < 20, commissions are limited to 5 starting colors. At rep >= 20, all 30 colors are available.
- **Recovery:** There is no game over. Low reputation reduces payouts but is always recoverable.

**How reputation is earned:**

| Source | Amount |
|--------|--------|
| Commission delivery | `clamp((beadCount × 0.0028) + (colorCount × 0.4) + (isComposite ? 0.5 : 0), 1, 5)` pts |
| Failed deadline | -2 pts |
| Tutorial completion | +10 pts |

**Reputation multipliers at different stages:**

| Stage | Reputation | Multiplier | Effect |
|-------|-----------|------------|--------|
| Early game | 0-20 | 1.0x - 1.2x | Base payouts. Limited to starting colors. First storylines unlock. |
| Mid game | 30-50 | 1.3x - 1.5x | 50% more coins. Full color palette. Moderate-complex patterns available. |
| Late game | 60-80 | 1.6x - 1.8x | Near-double payouts. All patterns unlocked. Composite patterns available. |
| Endgame | 90-100 | 1.9x - 2.0x | Double payouts. All storylines complete. Full workshop. |

## What Does This Skill Do?

Given a **story summary** and a **starting reputation score**, this skill produces:

| Output | Description |
|--------|-------------|
| **Commission list** | 5-10 fully detailed commission objects with IDs, names, descriptions, pattern axes, colors, payouts, reputation gates, and deadlines |
| **Pattern JSONs** | Engine-ready pattern data for importing into the game (grid sizes, color palettes, variant definitions) |
| **Pattern briefs** | Human-readable markdown documents for the pixelartist (design notes, color specs, variant details) |

## How to Use

### Requirements

[Claude Code](https://docs.claude.com/en/docs/claude-code/overview),
installed and authenticated.

### Setup

1. Clone the repo:
   ```bash
   git clone https://github.com/GixGosu/gdd-review-kit.git
   ```
2. Open Claude Code in that folder.
3. You get:
   - `.claude/agents/` with four agent definitions (StoryWriter,
     CommissionPlanner, PatternDesigner, EconomyBalancer)
   - `.claude/skills/hannabeads-commission-pipeline/` with the pipeline
     skill that orchestrates them
   - `Storylines/` where output lands, one folder per storyline

### Input

First invoke the skill with `/hannabeads-commission-pipeline`, then provide a story summary and starting reputation:

```
/hannabeads-commission-pipeline

Story: "Marco wants to impress Sofia, who loves cats. He starts asking for
commissions to hide secret presents for her. Eventually he confesses, and
Sofia asks us to make a pink heart with two cats to gift back to him."

Starting reputation: 40
```

### Pipeline Phases

The skill runs in 4 phases, each requiring your explicit approval:

**Phase 1 — Story Review**
The StoryWriter expands your summary into character messages with reputation gates and complexity hints. Review the narrative and approve or request changes.

**Phase 2 — Commission Plan Review**
The CommissionPlanner and PatternDesigner collaborate to map story steps to pattern axes. If an axis gap is found, you decide how to resolve it. Once all axes are mapped, payouts are validated with the EconomyBalancer. Review the commission table and approve.

**Phase 3 — Pattern Plan Review**
The PatternDesigner creates pattern JSONs and briefs for the pixelartist. Review grid sizes, colors, variants, and bead count estimates.

**Phase 4 — Final Output**
All files are saved to `Storylines/<storyline-title>/`:

```
Storylines/
└── marcos-secret-presents/
    ├── README.md              # Human-readable summary
    ├── STATE.md               # Pipeline state tracker
    ├── GDD_TODO.md            # Pending axis additions (if any)
    ├── commissions/
    │   ├── COM_001.json
    │   └── ...
    ├── patterns/
    │   ├── PAT_001.json
    │   └── ...
    └── pattern-briefs/
        ├── PAT_001.md
        └── ...
```

### The Crew

Four agents collaborate through the pipeline, each with a specific role:

| Agent | Purpose | Phase |
|-------|---------|-------|
| **StoryWriter** | Expands your story summary into character messages, splitting the narrative into 5-10 commission steps with complexity hints. Purely narrative — does not specify patterns, colors, or axes. | Phase 1 |
| **CommissionPlanner** | Reads each story step and works with the PatternDesigner to translate it into pattern axes. Assembles commission objects with axes, colors, payouts, and deadlines. Validates payouts with the EconomyBalancer before presenting to the user. | Phase 2 |
| **PatternDesigner** | Simplifies complex requests into base patterns, checks each against the axis taxonomy, and proposes Theme/Pattern/SubPattern axes. Creates engine-ready pattern JSONs and human-readable briefs for the pixelartist. Also decides on design choices like shading. | Phase 2, Phase 3 |
| **EconomyBalancer** | Validates commission payouts against bead costs and progression stage. Ensures payouts are fair — not too generous for simple patterns, not too stingy for complex ones. Adjusts payouts when needed. | Phase 2 |

For a visual diagram of how these agents connect and where human approval happens, see [ARCHITECTURE.md](./ARCHITECTURE.md).

### Modifying at Any Phase

You can request changes at any point:

- **Story changes** → "Can you change step 3's request from a mushroom to a flower?"
- **Reputation gates** → "The rep gates are too close together, spread them out"
- **Commission details** → "COM_003's payout is too low, increase it to 150"
- **Pattern designs** → "Make the cat pattern bigger — 24x24 instead of 16x16"
- **Go back** → "Go back to Phase 1, I want to revise the story"
