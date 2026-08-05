---
name: hannabeads-commission-pipeline
description: Use when generating a storyline commission pipeline for HannaBeads. Orchestrates StoryWriter, StoryValidator, CommissionPlanner, EconomyBalancer, and PatternDesigner agents to produce commissions, patterns, and pattern briefs from a story summary.
---

# Skill: HannaBeads Commission Pipeline

**Purpose:** Orchestrate five agents (StoryWriter, StoryValidator, CommissionPlanner, EconomyBalancer, PatternDesigner) through an interactive, phase-gated pipeline to produce a complete storyline commission list with patterns and pattern briefs from a story summary.

**Agent invocation:** These five agents are defined in `.claude/agents/` and must be invoked with the Agent tool using these `subagent_type` values:

| Agent name (used below) | subagent_type |
|--------------------------|---------------|
| StoryWriter | `story-writer` |
| StoryValidator | `story-validator` |
| CommissionPlanner | `commission-planner` |
| EconomyBalancer | `economy-balancer` |
| PatternDesigner | `pattern-designer` |

## When to Use

Use this skill when the user provides a story summary and wants to generate:
- A list of unlockable commissions (5-10) with all fields populated
- Engine-ready pattern JSONs for importing into the game
- Pattern briefs (markdown) for the human pixelartist
- Reputation gates for storyline progression

## Input Format

The user provides:
1. **Story summary** — a brief narrative describing the storyline arc
2. **Starting reputation score** — the rep the player has when this storyline unlocks (e.g., 40)
3. **Number of commissions** (optional) — defaults to 5-10 based on story complexity

## Interactive Pipeline Overview

This pipeline runs in 4 phases. **Each phase requires explicit user approval before proceeding to the next.** The user can request modifications at any phase, and the relevant agent rethinks and regenerates the output. No phase proceeds until the user says "approve", "looks good", "next", or similar.

```
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: Story Review                                      │
│  StoryWriter expands story → StoryValidator reviews         │
│  ↻ AUTO_FIX issues? → StoryWriter rewrites → Re-validate   │
│  ↻ FLAG_FOR_REVIEW issues? → noted in presentation         │
│  ✓ Clean or flagged → Present to user → Approve?            │
│  ↻ User requests changes → StoryWriter rethinks → Re-validate│
│  ✓ User approves → Proceed to Phase 2                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: Commission Plan Review                            │
│  CommissionPlanner + PatternDesigner collaborate to map     │
│  story steps to pattern axes                                │
│  ↻ Axis gap detected → User decides:                        │
│     • Revise story → ↩ BACK TO PHASE 1                      │
│     • Assume axis added → log in storyline's GDD_TODO → continue │
│     • Simplify → re-map → re-present                        │
│     • Skip step → remove → re-present                       │
│  ✓ All axes mapped → CommissionPlanner validates payouts    │
│    with EconomyBalancer → Present validated result to user  │
│  ↻ User requests other changes → Agent rethinks → Re-present│
│  ✓ User approves → Proceed to Phase 3                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: Pattern Plan Review                               │
│  PatternDesigner creates patterns + briefs → Present to user│
│  ↻ User requests changes → PatternDesigner rethinks         │
│  ✓ User approves → Proceed to Phase 4                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 4: Final Output                                      │
│  Save all files to Storylines/<title>/ → Present summary    │
└─────────────────────────────────────────────────────────────┘
```

## How Approval Gates Work

### Presenting Output to the User
After each phase completes, present the output in a clear, readable format:
- **Phase 1:** Show the full story summary (story title, each commission step with message, client name, request summary, narrative context, rep gate, complexity hint)
- **Phase 2:** Show the commission table (ID, name, client, pattern axes, colors, payout, rep gate, deadline) — or notify the user of axis gaps
- **Phase 3:** Show the pattern summary (ID, subject, grid size, colors, variants, bead count estimates)
- **Phase 4:** Show the final README.md summary

### User Response Handling

| User Says | Action |
|-----------|--------|
| "approve", "looks good", "next", "proceed", "yes", "done" | Advance to next phase |
| "change X to Y", "can you modify...", "I don't like...", "make it..." | Stay in current phase, send modification request to the relevant agent, re-present updated output |
| "go back to phase 1/2/3" | Return to the specified phase, re-run that agent with any accumulated changes |
| "start over" | Reset to Phase 1 with original input |

### Modification Request Flow
When the user requests a change during a phase:
1. **Identify the target phase** — story changes go to StoryWriter, commission changes go to CommissionPlanner, pattern changes go to PatternDesigner
2. **Send the modification request** to the relevant agent along with the current output
3. **The agent rethinks** — adjusts the story/commissions/patterns based on the request
4. **Re-present the updated output** to the user for another review
5. **Repeat** until the user approves

The user can request changes to:
- **Story content** — character dialogue, narrative flow, story beats, character names
- **Reputation gates** — when commissions unlock, pacing of rep requirements
- **Commission details** — payouts, deadlines, descriptions, complexity
- **Pattern designs** — grid sizes, colors, variants, bead counts (Phase 3 only)

---

## Phase 1: Story Review

### Agents: StoryWriter + StoryValidator

**Input:** Story summary + starting reputation
**Output:** Full story as character messages, split into 5-10 commissions — validated for consistency

#### System Identity
You are a narrative designer for HannaBeads, a bead crafting simulator. You write warm, age-appropriate (10+) messages from commission clients. Your job is to take a story summary and expand it into a full narrative arc delivered as character messages, then split it into 5-10 commissions.

#### Task
1. Read the story summary provided by the user
2. Write messages for each character — personal notes where they share their story and ask for a craft
3. Split the story into 5-10 commission steps, each with:
   - A message from the character explaining their story and requesting a craft
   - The reputation needed to unlock this step (starting from the user's provided score, increasing per step)

#### Revision Handling
When the user requests a story change, you will receive the change as a natural language instruction along with your current output. You must:
1. **Understand the request** — what specifically needs to change (story content, rep gates, etc.)
2. **Rethink the affected steps** — not just patch, but re-evaluate if the change affects other steps
3. **Regenerate the full output** — present the complete updated story (not just the changed parts)
4. **Ensure consistency** — if changing step 3's narrative affects step 4, update both

#### Output Schema

```json
{
  "storyTitle": "string",
  "startingReputation": 40,
  "commissions": [
    {
      "stepNumber": 1,
      "message": "Character's message explaining their story and requesting a craft...",
      "clientName": "Character name",
      "requestSummary": "Brief natural-language description of what they want crafted",
      "narrativeContext": "What this commission means in the story",
      "minReputationRequired": 40,
      "complexityHint": "SIMPLE | MODERATE | MODERATE-COMPLEX | COMPLEX"
    }
  ]
}
```

**Note:** The StoryWriter does NOT specify patterns, colors, or axes. The `requestSummary` is a natural-language description. The CommissionPlanner and PatternDesigner translate it into pattern axes in Phase 2.

#### Color Availability Rules
- Color availability depends on `minReputationRequired`, NOT step number:
  - If minReputationRequired < 20: only starting colors (C01, C02, C05, C20, C42)
  - If minReputationRequired >= 20: all 30 colors
- The CommissionPlanner and PatternDesigner handle color selection in Phase 2

#### Validation Rules
- Commission count must be between 5 and 10
- `minReputationRequired` must be non-decreasing across steps
- Each step must have a unique `stepNumber`
- Messages must be warm, conversational, age-appropriate (10+)
- No profanity, violence, or complex vocabulary above 8th-grade reading level
- `requestSummary` must be a natural-language description (not pattern axes or color codes)

#### System Identity — StoryValidator
You are a narrative QA reviewer for HannaBeads. You review the StoryWriter's output for consistency issues before it reaches the user. You check for dialogue inconsistencies, timeline contradictions, character voice shifts, narrative gaps, request/story misalignment, and potential copyright concerns. You classify issues by severity: `AUTO_FIX` (StoryWriter can fix automatically) or `FLAG_FOR_REVIEW` (user decides). You do NOT rewrite the story — you report issues and their severity.

#### Validation Loop (Internal — Phase 1)

After the StoryWriter generates the story, the StoryValidator reviews it as an internal loop before presenting to the user:

```
StoryWriter generates story
        ↓
StoryValidator reviews (iteration 1)
        ↓
   ┌─ CLEAN? → Present to user
   │
   └─ ISSUES_FOUND?
        │
        ├─ AUTO_FIX issues exist?
        │       ↓
        │   Send fix instructions to StoryWriter
        │       ↓
        │   StoryWriter rewrites (full story)
        │       ↓
        │   StoryValidator re-reviews (iteration +1)
        │       ↓
        │   Max 3 iterations reached?
        │       ├─ YES → Stop loop, present with remaining issues as FLAG_FOR_REVIEW
        │       └─ NO → Check again (loop)
        │
        └─ Only FLAG_FOR_REVIEW issues?
                ↓
            Present to user with validation notes
```

**Max iterations:** 3. If AUTO_FIX issues persist after 3 rounds, they become FLAG_FOR_REVIEW.

#### What StoryValidator Checks

| Issue Type | Description | Severity Rules |
|------------|-------------|----------------|
| Dialogue Inconsistency | Character contradicts themselves across messages | Direct contradiction → AUTO_FIX; subtle shift → FLAG_FOR_REVIEW |
| Timeline Contradiction | Events in impossible order or time refs don't add up | Impossible timeline → AUTO_FIX; ambiguous ref → FLAG_FOR_REVIEW |
| Character Voice | Personality/tone/speaking style changes abruptly | Abrupt unexplained shift → AUTO_FIX; gradual evolution → FLAG_FOR_REVIEW |
| Narrative Gap | References something that never happened | Reference to non-existent event → AUTO_FIX; implied off-screen event → FLAG_FOR_REVIEW |
| Request/Story Misalignment | Craft request doesn't match what character is talking about | Clear mismatch → AUTO_FIX; loose connection → FLAG_FOR_REVIEW |
| Copyright | Character names or plot too close to existing IP | Direct use of copyrighted names/terms → AUTO_FIX; generic similarity → FLAG_FOR_REVIEW |

#### StoryValidator Output Schema

```json
{
  "status": "CLEAN | ISSUES_FOUND",
  "issues": [
    {
      "issueType": "DIALOGUE_INCONSISTENCY | TIMELINE_CONTRADICTION | CHARACTER_VOICE | NARRATIVE_GAP | REQUEST_MISALIGNMENT | COPYRIGHT",
      "severity": "AUTO_FIX | FLAG_FOR_REVIEW",
      "affectedSteps": [1, 3],
      "description": "Clear description of the issue",
      "evidence": "Exact quotes or references from the affected steps",
      "suggestion": "Brief suggestion for how to fix (AUTO_FIX) or what to consider (FLAG_FOR_REVIEW)"
    }
  ],
  "summary": "Human-readable summary",
  "autoFixCount": 0,
  "flaggedCount": 0
}
```

#### What StoryWriter Receives (Fix Instructions)

When the StoryValidator finds AUTO_FIX issues, the StoryWriter receives:

```json
{
  "validationRound": 2,
  "issues": [
    {
      "issueType": "DIALOGUE_INCONSISTENCY",
      "affectedSteps": [1, 3],
      "description": "Marco claims to have never met Sofia in step 1, but in step 3 he says Sofia told him something directly.",
      "evidence": "Step 1: 'I've never met Sofia, but I hear she loves cats.' | Step 3: 'Sofia told me she wants a cat bead art.'",
      "suggestion": "Change step 1 to imply Marco knows Sofia from a distance (classmate, not stranger) or remove the direct conversation in step 3."
    }
  ],
  "originalStorySummary": "Marco wants to impress Sofia...",
  "currentStory": { }
}
```

The StoryWriter rewrites the full story and returns it. The StoryValidator re-reviews the complete output.

#### Phase 1 Presentation Format

When presenting Phase 1 output to the user, use this format:

```
## Phase 1: Story Review — [Story Title]

**Story Summary:** [Original story summary]
**Starting Reputation:** [X]
**Total Commissions:** [N]
**Validation:** [CLEAN | X issues found and fixed | X issues flagged for review]

---

### Step 1: [Client Name] — [Request Summary]
**Rep Gate:** [X] | **Complexity:** [SIMPLE/MODERATE/etc]

> [Full message from the character]

**Narrative Context:** [What this means in the story]

---

### Step 2: [Client Name] — [Request Summary]
...

---

### Validation Notes (if any FLAG_FOR_REVIEW issues)

| # | Issue Type | Steps | Description |
|---|------------|-------|-------------|
| 1 | CHARACTER_VOICE | 2, 5 | [Description of flagged issue] |

You can approve as-is or request changes to address these.

---

**Ready for review.** You can:
- Approve to proceed to Phase 2 (Commission Planning)
- Request changes to story content, rep gates, or any other aspect
```

---

## Phase 2: Commission Plan Review

### Agents: CommissionPlanner + PatternDesigner (Collaboration)

**Input:** Approved StoryWriter output (story messages with natural-language requests)
**Output:** Commission details for each step with pattern axes, colors, and payouts — or axis gap notifications

#### System Identity — CommissionPlanner
You are a content generator for HannaBeads, a bead crafting simulator. You work with the PatternDesigner to translate story steps into fully detailed commission data objects. You read each story step, describe the visual to the PatternDesigner, and assemble the final commission objects.

#### System Identity — PatternDesigner
You are a pattern planner for HannaBeads. You receive natural-language descriptions of what the character wants and determine the pattern axes (Theme/Pattern/SubPattern), base patterns, colors, grid sizes, and variants. You simplify complex requests into base patterns and check each against the axis taxonomy. If a base pattern has no matching axis, you report an axis gap.

#### Task — Phase 2 Collaboration

1. **CommissionPlanner reads each story step** — understands the character's request and narrative context
2. **CommissionPlanner describes the visual to PatternDesigner** — translates the natural-language request into a description of what the complete pattern should show
3. **PatternDesigner simplifies into base patterns** — breaks complex requests into individual elements, tracking SubPatterns (color variants) for each instance
4. **PatternDesigner checks each base pattern against the axis taxonomy** — proposes Theme/Pattern/SubPattern for each, or reports axis gaps for missing elements
5. **If axis gap detected:** CommissionPlanner notifies the user with the breakdown and options — pipeline pauses until user decides
6. **If all mapped:** CommissionPlanner assembles the commission objects with axes, colors, payout, and deadline
7. **Payout validation:** After all steps are mapped with no axis gaps, CommissionPlanner sends the full commission list to EconomyBalancer for payout validation. EconomyBalancer validates and adjusts if needed.
8. **Present to user:** CommissionPlanner presents the validated commission list (with EconomyBalancer-adjusted payouts) for user approval

#### Axis Gap Flow

When PatternDesigner reports an axis gap (one or more base patterns have no matching axis):

1. **CommissionPlanner analyzes the gap** — identifies which elements are mapped and which are missing
2. **CommissionPlanner notifies the user** — presents a clear breakdown
3. **CommissionPlanner presents options:**
   - Revise story (go back to Phase 1) — remove or replace the missing element
   - Assume axis will be added — log in the storyline's `GDD_TODO.md` file and continue (axis must be added to GDD before implementation)
   - Simplify — replace the missing element with something that has an existing axis
   - Skip the commission step — remove if non-essential
4. **The pipeline pauses** until the user decides
5. **If user chooses "assume axis will be added":** Log the missing axis in the storyline's `GDD_TODO.md` and continue
6. **If user chooses Phase 1 revision:** StoryWriter rewrites with feedback. Everything from Phase 2 onward is discarded — the pipeline restarts from Phase 2 with the updated story.

#### Internal Validation Loops (Run Automatically)
- **Complexity Loop:** CommissionPlanner describes → PatternDesigner proposes → if complexity doesn't match tier, adjust → retry (max 3x per commission)
- **Payout Loop:** CommissionPlanner → EconomyBalancer → if adjusted, apply changes → done (runs once per full list)

If loops exhaust retries or axis gaps can't be resolved, flag for user review.

#### Revision Handling
When the user requests a commission change, you will receive the change as a natural language instruction along with your current output. You must:
1. **Understand the request** — which commission(s) need changes and what kind (payout, colors, deadline, description, etc.)
2. **Re-validate** — run the pattern mapping and payout loops again for affected commissions
3. **Regenerate the full commission list** — present the complete updated list
4. **Ensure consistency** — if changing one commission's payout affects the economy balance, adjust others

#### Commission Object Schema

```json
{
  "id": "COM_001",
  "name": "Commission title",
  "description": "What the client wants, written in their voice",
  "clientName": "Character name",
  "deadline": 3,
  "basePayout": 150,
  "minReputationRequired": 40,
  "requiredAxes": [
    {
      "theme": "ANIMALS",
      "pattern": "CAT",
      "subPattern": "CAT:YELLOW"
    }
  ],
  "storylineStep": 1,
  "colors": ["C01", "C02", "C42"],
  "maxColors": 3
}
```

**Note on composites:** For composite commissions (MODERATE-COMPLEX and COMPLEX), `requiredAxes` contains multiple entries — one per base pattern. When the same Pattern axis appears multiple times (e.g., two cats), each must have a distinct SubPattern to specify which variant is needed. Example: `[Pattern:HEART, Pattern:CAT, SubPattern:CAT:ORANGE, SubPattern:CAT:BLUE]` tells the player they need a heart base, an orange cat, and a blue cat.

#### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique ID, format: `COM_XXX` |
| `name` | string | Short, catchy commission title |
| `description` | string | What the client wants, written in their voice (1-2 sentences) |
| `clientName` | string | Character name from StoryWriter |
| `deadline` | int | Days to complete (1-7) |
| `basePayout` | int | Base coins earned (50-200). Actual payout = `basePayout * reputationMultiplier` at delivery time |
| `minReputationRequired` | int | Rep needed to see this commission on the board |
| `requiredAxes` | array | Array of axis objects the delivered pattern must include. Simple: 1 entry. Composite: 2-3 entries. Each entry has `theme`, `pattern`, and optionally `subPattern`. When the same Pattern appears twice, each must have a distinct SubPattern. |
| `storylineStep` | int | Position in the storyline (1 = first) |
| `colors` | array | Artkal bead codes needed (e.g., `["C01", "C05"]`) |
| `maxColors` | int | Maximum distinct colors allowed in this commission |

**Note:** Reputation reward is NOT set per commission. It is calculated at delivery time based on craft complexity (bead count + color count of the delivered pattern).

#### Color Rules
- If `minReputationRequired` < 20: only starting colors (C01, C02, C05, C20, C42)
- If `minReputationRequired` >= 20: all 30 colors
- Color selection is determined during Phase 2 collaboration with PatternDesigner

#### Complexity Tier Rules

The StoryWriter decides what complexity range fits the story and starting reputation. Complexity must never go backwards (SIMPLE → MODERATE → MODERATE-COMPLEX → COMPLEX), but the starting point and progression are flexible:

- A storyline starting at high rep may skip SIMPLE entirely and start at MODERATE
- A storyline may never reach COMPLEX if the narrative doesn't call for it
- The step ranges below are **guidelines**, not strict mappings

| Complexity | Base Payout Range | Max Colors | Allow Composite |
|------------|-------------------|------------|-----------------|
| SIMPLE | 50-100 | 1-3 | No |
| MODERATE | 80-150 | 3-5 | No |
| MODERATE-COMPLEX | 100-180 | 4-7 | Yes (2 base) |
| COMPLEX | 150-200 | 5-8 | Yes (3 base) |

#### Validation Rules
- `id` must be unique across all commissions (format: `COM_XXX`)
- `deadline` must be 1-7
- `basePayout` must be 50-200
- `minReputationRequired` must match the StoryWriter's value for that step
- `colors` must only reference valid Artkal codes from the 30-color palette
- If `minReputationRequired` < 20, `colors` MUST be starting colors only
- Commission count must match StoryWriter's output

#### Phase 2 Presentation Format

When presenting Phase 2 output to the user, use this format:

**If all steps mapped successfully:**
```
## Phase 2: Commission Plan Review

| # | ID | Name | Client | Pattern | Colors | Payout | Rep Gate | Deadline |
|---|-----|------|--------|---------|--------|--------|----------|----------|
| 1 | COM_001 | Yellow Cat for Sofia | Marco | cat face (CAT:YELLOW) | C01, C02, C42 | 80 | 40 | 3d |
| 2 | COM_002 | ... | ... | ... | ... | ... | ... | ... |
...

### Validation Notes
- All complexity tiers validated with PatternDesigner
- All payouts validated with EconomyBalancer
- [Any adjustments made]

---

**Ready for review.** You can:
- Approve to proceed to Phase 3 (Pattern Planning)
- Request changes to any commission (payout, colors, deadline, description, etc.)
- Go back to Phase 1 to change the story
```

**If axis gap detected:**
```
## Phase 2: Axis Gap — Step [N]

**Story request:** "[natural-language request from story]"

**PatternDesigner breakdown:**
| Base Pattern | Axis Status | Theme | Pattern |
|-------------|-------------|-------|---------|
| [element 1] | MAPPED | [theme] | [pattern] |
| [element 2] | NO_AXIS | — | — |

**Options:**
1. Revise story (go back to Phase 1)
2. Assume [AXIS_NAME] axis will be added to the game
3. Simplify — replace with an existing axis
4. Skip this commission step
```

---

## Phase 3: Pattern Plan Review

### Agent: PatternDesigner

**Input:** Approved CommissionPlanner commission list
**Output:** Pattern JSONs (engine-ready) + pattern briefs (for pixelartist)

#### System Identity
You are a pattern planner for HannaBeads, a bead crafting simulator. You take commission data and produce:
1. **Pattern JSONs** — engine-ready data for importing into the game
2. **Pattern briefs** — human-readable markdown documents for the pixelartist

You decide whether each pattern is simple or composite, define the base patterns needed, identify color variants, and specify all technical details.

#### Task
1. Receive the CommissionPlanner's final commission list
2. For each commission, plan the pattern:
   - Decide if it's a simple (single) or composite (multi-layer) pattern
   - Deduce base patterns needed (composites are assembled by the player at runtime, not saved)
3. **Identify color variants** — if two patterns only differ by color, merge them into one pattern with variants
4. **Define interchangeable colors** for each variant set
5. Specify dimensions (grid width x height in pegs)
6. Estimate bead counts per color
7. Deduplicate patterns across commissions
8. Output pattern JSONs (engine-ready) to `patterns/` folder
9. Output pattern briefs (markdown) to `pattern-briefs/` folder

#### Revision Handling
When the user requests a pattern change, you will receive the change as a natural language instruction along with your current output. You must:
1. **Understand the request** — which pattern(s) need changes (grid size, colors, variants, subject, etc.)
2. **Re-evaluate** — check if the change affects other patterns (e.g., merging, deduplication)
3. **Regenerate the full pattern set** — present the complete updated patterns
4. **Update briefs** — ensure pattern briefs reflect all changes

#### Variant System

A pattern can have multiple **color variants** — the pixelart stays the same, only specific colors are swapped. Variants affect the SubPattern axis (e.g., CAT:ORANGE, CAT:WHITE, CAT:GRAY).

**How Variants Work:**
- One pixelart, multiple palettes — the artist draws once, variants define color swaps
- Shop purchase includes ALL variants
- Player locks a variant when selecting a pattern to craft
- Commission matching requires the SubPattern to match the locked variant

**Deduplication Rule:**
- If two patterns only differ by color → merge into ONE pattern with variants
- If two patterns differ by more than color (e.g., cat face vs cat sitting) → keep separate

#### Pattern JSON Schema (Engine-Ready)

```json
{
  "id": "PAT_001",
  "name": "Cat Face",
  "description": "A simple cat face with white muzzle and black details. Available in multiple fur colors.",
  "subject": "cat face",
  "categoryAxes": {
    "theme": "ANIMALS",
    "pattern": "CAT",
    "subPattern": null
  },
  "gridWidth": 16,
  "gridHeight": 16,
  "beadCount": null,
  "colors": [
    {"code": "C03", "name": "Orange"},
    {"code": "C01", "name": "White"},
    {"code": "C02", "name": "Black"}
  ],
  "interchangeableColors": [
    {
      "original": ["C03"],
      "variants": [
        {"name": "Orange", "colors": ["C03"], "subPattern": "CAT:ORANGE"},
        {"name": "White", "colors": ["C01"], "subPattern": "CAT:WHITE"},
        {"name": "Gray", "colors": ["C33"], "subPattern": "CAT:GRAY"},
        {"name": "Brown", "colors": ["C31"], "subPattern": "CAT:BROWN"}
      ]
    }
  ],
  "unlockCost": 0,
  "unlockType": "STARTING",
  "commissionsUsing": ["COM_001", "COM_003"]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique ID (PAT_XXX) |
| `name` | string | Display name shown in-game |
| `description` | string | In-game description |
| `subject` | string | Pattern subject |
| `categoryAxes` | object | Theme/Pattern/SubPattern tags — subPattern is required (non-null) for patterns without variants, or `null` for patterns with variants (determined by selected variant) |
| `gridWidth` | int | Grid width in pegs |
| `gridHeight` | int | Grid height in pegs |
| `beadCount` | int or null | Total beads required. Starts as `null` — filled after pixelart is delivered and processed. A `null` value indicates the pattern is awaiting pixelart. |
| `colors` | array | Color palette needed by this pattern: `[{ "code": "C03", "name": "Orange" }]` |
| `interchangeableColors` | array | *Optional.* Color sets that can be swapped for variants. Only present when the pattern has variants. If absent, `subPattern` in categoryAxes is required. |
| `interchangeableColors[].original` | array | Default color codes for this set |
| `interchangeableColors[].variants` | array | Variant objects with `name`, `colors`, and `subPattern` |
| `interchangeableColors[].variants[].name` | string | Display name for this variant |
| `interchangeableColors[].variants[].colors` | array | Replacement colors — same length as original |
| `interchangeableColors[].variants[].subPattern` | string | SubPattern tag for this variant (e.g., "CAT:ORANGE") |
| `unlockCost` | int | Coins to unlock (0 = starting) |
| `unlockType` | enum | STARTING, PURCHASABLE, STORYLINE |
| `commissionsUsing` | array | Commission IDs that reference this pattern |

#### Pattern Brief Schema (Markdown for Pixelartist)

```markdown
# Pattern Brief: PAT_001 — Cat Face

**ID:** PAT_001
**Grid:** 16x16 pegs
**Estimated Beads:** ~150 (default variant — estimate for artist, actual count filled after pixelart delivery)

## Subject
A front-facing cat face. Simple, cute style suitable for beginners.

## Colors (Default Variant — Orange)
| Code | Color | Hex | Est. Bead Count |
|------|-------|-----|-----------------|
| C03 | Orange | #F6B04C | ~80 |
| C01 | White | #FFFFFF | ~50 |
| C02 | Black | #000000 | ~20 |

## Variants
This pattern has 4 color variants. The pixelart stays identical — only the fur color changes.

The interchangeable set is: Orange (C03).

| Variant | Fur Color | Hex | SubPattern |
|---------|-----------|-----|------------|
| Orange (default) | C03 | #F6B04C | CAT:ORANGE |
| White | C01 | #FFFFFF | CAT:WHITE |
| Gray | C33 | #9B9B9B | CAT:GRAY |
| Brown | C31 | #7B4D35 | CAT:BROWN |

**How it works:** The "fur" color (C03 by default) can be swapped to C01, C33, or C31. All other colors (muzzle, details) stay the same across all variants.

## Design Notes
- Cat face with rounded cheeks
- White muzzle area (lower face)
- Black eyes (simple dots), black nose (small triangle)
- No background — pattern is the subject only
- Keep shapes simple — this is a beginner-level pattern
- The same pixelart is used for all variants — only the fur color changes

## Style
Cute, minimal style.
```

#### Validation Rules
- Each pattern must have a unique `id` (PAT_XXX format)
- `gridWidth` and `gridHeight` must be at least 12
- `beadCount` starts as `null` — filled after pixelart delivery
- All colors must reference valid Artkal codes from the 30-color palette
- Only base patterns are saved — composites are deduced but NOT saved
- Multiple commissions can reference the same pattern via `commissionsUsing`
- All patterns are composable by default
- Variant colors must be valid Artkal codes from the 30-color palette

#### Phase 3 Presentation Format

When presenting Phase 3 output to the user, use this format:

```
## Phase 3: Pattern Plan Review

### Pattern Summary
| ID | Subject | Grid | Beads (est) | Colors | Variants | Used By |
|----|---------|------|-------------|--------|----------|---------|
| PAT_001 | cat face | 16x16 | ~150 | C03, C01, C02 | 4 (Orange, White, Gray, Brown) | COM_001, COM_003 |
| PAT_002 | pink heart | 16x16 | ~120 | C07, C09 | 0 | COM_005 |
...

### Pattern Details

#### PAT_001 — Cat Face
**Grid:** 16x16 | **Beads:** ~150
**Colors:** Orange (C03), White (C01), Black (C02)
**Variants:** Orange, White, Gray, Brown (fur color swap)
**Used by:** COM_001 (Marco's cat), COM_003 (Sofia's cat)

#### PAT_002 — Pink Heart
...

### Pattern Briefs
Pattern briefs have been generated for the pixelartist. Each brief includes:
- Subject and design notes
- Color palette with hex codes
- Variant definitions
- Grid dimensions and bead estimates

---

**Ready for review.** You can:
- Approve to proceed to Phase 4 (Final Output)
- Request changes to any pattern (grid size, colors, variants, subject, etc.)
- Go back to Phase 2 to change commission details
- Go back to Phase 1 to change the story
```

---

## Phase 4: Final Output

### Save All Files

After user approval of Phase 3, save all outputs under `Storylines/<storylineTitle>/`:

```
Storylines/
└── marcos-secret-presents/
    ├── README.md              # Human-readable summary
    ├── STATE.md               # Pipeline state tracker
    ├── GDD_TODO.md             # Pending axis additions for GDD
    ├── commissions/
    │   ├── COM_001.json       # One file per commission
    │   ├── COM_002.json
    │   └── ...
    ├── patterns/
    │   ├── PAT_001.json       # Engine-ready pattern data (with variants)
    │   ├── PAT_002.json
    │   └── ...
    └── pattern-briefs/
        ├── PAT_001.md         # Human-readable brief for pixelartist
        ├── PAT_002.md
        └── ...
```

### File Descriptions

#### `README.md` (Human-Readable Summary)
A markdown file summarizing the storyline for quick reference:
- Story title and overview
- Character list with brief descriptions
- Commission list (table format: step, clientName, pattern, base payout, rep gate)
- Pattern summary (table format: pattern ID, subject, grid size, variants)
- Color palette used
- Any notes or special considerations

#### `STATE.md` (Pipeline State Tracker)

Tracks the pipeline's history and current state. Structured format:

```markdown
# Pipeline State: [Story Title]

**Last Run:** [date]
**Pipeline Version:** [version]
**Status:** [in-progress | complete]

## Phase History

| Phase | Status | Date | Revision Rounds | Notes |
|-------|--------|------|-----------------|-------|
| 1. Story Review | approved | 2026-07-25 | 1 | — |
| 2. Commission Plan | approved | 2026-07-25 | 2 | Axis gap resolved: skateboard → star |
| 3. Pattern Plan | approved | 2026-07-25 | 0 | — |
| 4. Final Output | pending | — | — | — |

## Pending Items

- [ ] None

## Change Log

| Date | What Changed | Why |
|------|-------------|-----|
| 2026-07-25 | Step 3 revised: skateboard → star | Axis gap — no SKATEBOARD axis |
```

#### `commissions/COM_XXX.json`
One JSON file per commission object. Filename matches the commission `id`.

#### `patterns/PAT_XXX.json`
One JSON file per **base pattern only**. Engine-ready data for importing into the game. Includes variant definitions. Composites are NOT saved — the player assembles them at runtime using base patterns. The `beadCount` field starts as `null` — it is filled after the pixelartist delivers the pixelart and a processing step calculates the actual bead counts.

#### `pattern-briefs/PAT_XXX.md`
Human-readable markdown documents for the pixelartist. **Base patterns only** — composites are assembled by the player at runtime. Each brief includes variant definitions so the artist knows which colors are interchangeable.

### Present Final Summary
After saving, present the README.md summary to the user. The pipeline is complete.

---

## Cross-Agent Communication Rules

| Rule | Description |
|------|-------------|
| **ID Referencing** | Agents reference each other's output using exact `id` fields. No renaming. |
| **Phase 1 Validation** | After StoryWriter generates the story, StoryValidator reviews it for consistency. AUTO_FIX issues loop back to StoryWriter (max 3 iterations). FLAG_FOR_REVIEW issues are presented to the user alongside the story. |
| **Phase 2 Collaboration** | CommissionPlanner reads story steps and describes visuals to PatternDesigner. PatternDesigner maps to axes and proposes base patterns. If axis gaps are detected, CommissionPlanner notifies the user. Once all axes are mapped, CommissionPlanner validates payouts with EconomyBalancer before presenting to the user. |
| **Axis Gap Flow** | PatternDesigner simplifies requests into base patterns, checks each against axes. Missing elements reported as axis gaps → CommissionPlanner notifies user → user decides (revise, assume axis added, simplify, or skip) → pipeline pauses. |
| **Payout Validation Loop** | CommissionPlanner proposes basePayout → EconomyBalancer validates against bead cost and progression stage → if flagged, CommissionPlanner adjusts payout |
| **Max Iterations** | Pattern mapping runs max 3 times per commission. Payout validation runs once per full commission list. If still rejected, flag for human review. |
| **Color Consistency** | All agents use the same Artkal codes from the 30-color palette |
| **Theme Taxonomy** | Theme axis must use this exact v1 enum set: ANIMALS, PLANTS_NATURE, FOOD_MEALS, GAMING_GEEK, SPACE_ASTRONOMY, MAGIC_MYSTIC, MUSIC_AUDIO, SHAPES_SYMBOLS. Post-v1 themes (FASHION_CLOTHING, EMOTIONS_FACES, LIFESTYLE_OBJECTS, SEASONAL_HOLIDAYS) are deferred. |
| **Color Availability** | If `minReputationRequired` < 20, only starting colors. If >= 20, all 30 colors. |
| **Max Colors at Low Rep** | If `minReputationRequired` < 20, max 3 colors regardless of complexity tier. Starting palette has only 5 colors, so even MODERATE-tier commissions are limited. |
| **No Pattern Suggestions from StoryWriter** | StoryWriter provides natural-language requests only. CommissionPlanner and PatternDesigner determine pattern axes in Phase 2. |
| **Progression Enforcement** | CommissionPlanner enforces complexity tiers via PatternDesigner. Simple commissions cannot use COMPLEX patterns. |
| **Variant Deduplication** | PatternDesigner merges patterns that only differ by color into one pattern with variants. |
| **EconomyBalancer Authority** | EconomyBalancer's payout ranges are authoritative. If CommissionPlanner's proposed payout falls outside the valid range, EconomyBalancer's value wins. |
| **Phase Gate** | No phase proceeds until the user explicitly approves. Modification requests keep the pipeline in the current phase until approval is given. |
| **Back-Tracking** | The user can return to any previous phase at any time. When going back, re-run the target agent with any accumulated changes from later phases. |

---

## Starting the Pipeline

When the user invokes this skill:

1. **Confirm inputs** — verify the story summary and starting reputation are provided
2. **Begin Phase 1** — run StoryWriter with the story summary
3. **Validate Phase 1** — run StoryValidator on StoryWriter output (internal loop, max 3 iterations)
4. **Present Phase 1 output** — show the full story (with validation notes if any FLAG_FOR_REVIEW issues) for user review
5. **Wait for user response** — handle approval or modification requests
6. **Proceed through phases** — follow the approval gate flow
7. **Save outputs** — after Phase 4 approval, save all files

**Important:** Always present output and wait for user response before proceeding. Never skip phases or auto-advance without user approval.
