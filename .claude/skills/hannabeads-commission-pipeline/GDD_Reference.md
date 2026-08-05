# GDD Reference for Commission Pipeline

**Purpose:** Single source of truth for all GDD data referenced by the pipeline skill and agents. When this file conflicts with the main GDD (`Docs/docs/game-design/` pages), the main GDD wins — update this file to match.

**Source:** `Docs/docs/game-design/` pages (v0.14)

**Axis Naming:**
| Axis | Old Name | New Name | What It Checks |
|------|----------|----------|----------------|
| 1 | Theme | **Category** | The broad domain (ANIMALS, FOOD_MEALS, etc.) |
| 2 | Pattern | **Subject** | The specific thing (CAT, FLOWER, HEART, etc.) |
| 3 | SubPattern | **Variant** | The color-specific version (CAT:ORANGE, FLOWER:RED) |

---

## 1. 30-Color Artkal Palette

All colors use Artkal Mini C-2.6mm beads.

| Code | Name | R | G | B | Hex | Color Group |
|------|------|---|---|---|-----|-------------|
| C01 | White | 255 | 255 | 255 | #FFFFFF | Neutral |
| C02 | Black | 0 | 0 | 0 | #000000 | Neutral |
| C03 | Orange | 246 | 176 | 76 | #F6B04C | Warm |
| C05 | Red | 225 | 6 | 0 | #E10600 | Warm |
| C07 | Pink | 241 | 167 | 220 | #F1A7DC | Warm |
| C09 | Hot Pink | 219 | 33 | 82 | #DB2152 | Warm |
| C10 | Light Yellow | 242 | 240 | 161 | #F2F0A1 | Warm |
| C12 | Light Green | 173 | 220 | 145 | #ADDC91 | Cool |
| C13 | Green | 135 | 216 | 57 | #87D839 | Cool |
| C14 | Teal | 36 | 158 | 107 | #249E6B | Cool |
| C15 | Dark Teal | 0 | 124 | 88 | #007C58 | Cool |
| C17 | Bright Orange | 255 | 103 | 31 | #FF671F | Warm |
| C19 | Light Blue | 65 | 182 | 230 | #41B6E6 | Cool |
| C20 | Blue | 0 | 144 | 218 | #0090DA | Cool |
| C21 | Dark Blue | 0 | 51 | 153 | #003399 | Cool |
| C22 | Salmon | 252 | 191 | 169 | #FCBFA9 | Warm |
| C23 | Tan | 204 | 153 | 102 | #CC9966 | Warm |
| C25 | Light Purple | 167 | 123 | 202 | #A77BCA | Cool |
| C26 | Purple | 160 | 94 | 181 | #A05EB5 | Cool |
| C31 | Brown | 123 | 77 | 53 | #7B4D35 | Warm |
| C32 | Dark Brown | 92 | 71 | 56 | #5C4738 | Warm |
| C33 | Gray | 155 | 155 | 155 | #9B9B9B | Neutral |
| C34 | Dark Gray | 118 | 119 | 119 | #767777 | Neutral |
| C42 | Yellow | 250 | 224 | 83 | #FAE053 | Warm |
| C43 | Maroon | 165 | 0 | 52 | #A50034 | Warm |
| C47 | Light Peach | 243 | 207 | 179 | #F3CFB3 | Warm |
| C51 | Cream | 252 | 251 | 205 | #FCFBCD | Warm |
| C52 | Dark Purple | 74 | 31 | 135 | #4A1F87 | Cool |
| C57 | Crimson | 188 | 4 | 35 | #BC0423 | Warm |
| C88 | Silver | 209 | 209 | 209 | #D1D1D1 | Neutral |

**Starting colors (5):** C01 (White), C02 (Black), C05 (Red), C20 (Blue), C42 (Yellow)

**Color group counts:**
- Warm (16): C03, C05, C07, C09, C10, C17, C22, C23, C31, C32, C42, C43, C47, C51, C57
- Cool (10): C12, C13, C14, C15, C19, C20, C21, C25, C26, C52
- Neutral (5): C01, C02, C33, C34, C88

---

## 2. Color Availability

All 30 colors are available in the shop from the start — same price, no reputation gates, no level locking. Buy what you need when you have the money.

| Rule | Detail |
|---|---|
| **Starting stock** | 5 colors (C01, C02, C05, C20, C42) with 100 beads each (500 total) |
| **Shop availability** | All 30 colors, always available |
| **Pricing** | Same pack prices for all colors (15/25/50 coins for 50/100/250 beads) |
| **No reputation gate** | Colors are not locked behind reputation — only money limits purchases |
| **Commission color constraint** | If `minReputationRequired` <= 20, only request colors from the starting 5 (C01, C02, C05, C20, C42). The player likely hasn't bought other colors yet. |

---

## 3. Category Taxonomy (v1 — 8 Categories)

| Category (Enum) | Example Subjects |
|---|---|
| ANIMALS | Cats, dogs, birds, bunnies, frogs, butterflies |
| PLANTS_NATURE | Flowers, trees, mushrooms, cacti, sunflowers, rainbows |
| FOOD_MEALS | Pizza, cupcakes, sushi, ramen, coffee, ice cream |
| GAMING_GEEK | Controllers, pixel icons, robots, dice, arcade cabinets |
| SPACE_ASTRONOMY | Planets, rockets, stars, moons, aliens, astronauts |
| MAGIC_MYSTIC | Crystal balls, wands, potions, dragons, unicorns, spell books |
| MUSIC_AUDIO | Guitars, music notes, headphones, microphones, vinyl, drums |
| SHAPES_SYMBOLS | Hearts, stars, lightning bolts, diamonds, arrows, snowflakes |

**Post-v1 (deferred):** FASHION_CLOTHING, EMOTIONS_FACES, LIFESTYLE_OBJECTS, SEASONAL_HOLIDAYS

---

## 4. Subject Axis Taxonomy (by Category)

Each Subject axis value is a broad group — multiple specific pixelart designs can share it (e.g., Cat Face, Cat Sleeping, Cat Sitting all use Subject:CAT).

| Category | Subject Axis Values |
|---|---|
| ANIMALS | CAT, DOG, BIRD, BUNNY, FROG, BUTTERFLY |
| PLANTS_NATURE | FLOWER, MUSHROOM, CACTUS, TREE, SUNFLOWER, RAINBOW |
| FOOD_MEALS | PIZZA, CUPCAKE, SUSHI, RAMEN BOWL, COFFEE CUP, ICE CREAM |
| GAMING_GEEK | GAME CONTROLLER, PIXEL HEART, ROBOT, D20 DIE, RUBIK'S CUBE, ARCADE CABINET |
| SPACE_ASTRONOMY | PLANET, ROCKET, STAR, MOON, ALIEN, ASTRONAUT |
| MAGIC_MYSTIC | CRYSTAL BALL, WAND, POTION BOTTLE, DRAGON, UNICORN, SPELL BOOK |
| MUSIC_AUDIO | GUITAR, MUSIC NOTE, HEADPHONES, MICROPHONE, VINYL RECORD, DRUM |
| SHAPES_SYMBOLS | HEART, STAR, LIGHTNING BOLT, DIAMOND, ARROW, SNOWFLAKE |

**Variant format:** `SUBJECT:COLOR` (e.g., CAT:ORANGE, FLOWER:RED, HEART:PINK)

---

## 5. v1 Pattern Library (48 Base Designs)

6 designs per category across 8 categories.

### ANIMALS (6)
| Design | Notes |
|---|---|
| Cat | Variants: orange, black, white, gray, calico |
| Dog | Variants: golden, black, brown, white |
| Bird | Variants: robin, parrot, blue jay |
| Bunny | Sitting, standing |
| Frog | Sitting, jumping |
| Butterfly | Wing patterns |

### PLANTS_NATURE (6)
| Design | Notes |
|---|---|
| Flower | Rose, sunflower, tulip, daisy, cherry blossom, lavender |
| Mushroom | Red cap, white spots |
| Cactus | Barrel, saguaro, prickly pear |
| Tree | Oak, palm, pine, bonsai |
| Sunflower | Tall stem, face |
| Rainbow | Arc with bands |

### FOOD_MEALS (6)
| Design | Notes |
|---|---|
| Pizza | Slice, whole pie |
| Cupcake | Frosting, sprinkles |
| Sushi | Roll, nigiri |
| Ramen bowl | Chopsticks, steam |
| Coffee cup | Steam, latte art |
| Ice cream | Cone, sundae |

### GAMING_GEEK (6)
| Design | Notes |
|---|---|
| Game controller | Rounded, retro |
| Pixel heart | 8-bit style |
| Robot | Cute, boxy |
| D20 die | Face showing |
| Rubik's cube | Solved, scrambled |
| Arcade cabinet | Side view |

### SPACE_ASTRONOMY (6)
| Design | Notes |
|---|---|
| Planet | Saturn, Earth, Mars, Jupiter |
| Rocket | Launch, flying |
| Star | Five-point, twinkling |
| Moon | Crescent, full, half |
| Alien | Green, cute |
| Astronaut | Helmet, floating |

### MAGIC_MYSTIC (6)
| Design | Notes |
|---|---|
| Crystal ball | Glowing, on stand |
| Wand | Star tip, sparkles |
| Potion bottle | Round, tall, corked |
| Dragon | Cute, cartoon |
| Unicorn | Horn, mane |
| Spell book | Open, closed, glowing |

### MUSIC_AUDIO (6)
| Design | Notes |
|---|---|
| Guitar | Acoustic, electric |
| Music note | Eighth, quarter, treble clef |
| Headphones | Over-ear, on-ear |
| Microphone | Handheld, stand |
| Vinyl record | With label |
| Drum | Snare, bass, bongo |

### SHAPES_SYMBOLS (6)
| Design | Notes |
|---|---|
| Heart | Solid, outlined, pixel |
| Star | Five-point, six-point, shooting |
| Lightning bolt | Zigzag |
| Diamond | Gem cut, square rotated |
| Arrow | Up, right, double |
| Snowflake | Six-fold symmetry |

**Starting designs (6):** Cat, Heart, Dog, Flower, Tree, Coffee Cup

---

## 6. Complexity Tier Rules

The StoryWriter decides the starting complexity and progression. Complexity must never go backwards.

| Complexity | Base Payout Range | Max Colors | Allow Composite |
|------------|-------------------|------------|-----------------|
| SIMPLE | 50-100 | 1-3 | No |
| MODERATE | 80-150 | 3-5 | No |
| MODERATE-COMPLEX | 100-180 | 4-7 | Yes (2 base) |
| COMPLEX | 150-200 | 5-8 | Yes (3 base) |

**Note:** Actual payout = `basePayout * reputationMultiplier` at delivery time. The multiplier is `1.0 + (reputation / 100)`.

---

## 7. Reputation System

### Reputation Formula (earned at delivery)
```
reputationEarned = clamp(
    (beadCount × 0.0028) + (colorCount × 0.4) + (isComposite ? 0.5 : 0),
    1, 5
)
```

| Field | Type | Description |
|-------|------|-------------|
| `beadCount` | int | Total beads in delivered pattern |
| `colorCount` | int | Distinct bead colors used |
| `isComposite` | bool | True if assembled from 2+ base designs |

**Approximate tier mapping:**

| Rep | Bead Range (2 colors) | Bead Range (4 colors) | Bead Range (6 colors) |
|-----|----------------------|----------------------|----------------------|
| 1 pt | 1–350 | 1–210 | 1–140 |
| 2 pts | 351–710 | 211–430 | 141–360 |
| 3 pts | 711–1,070 | 431–710 | 361–640 |
| 4 pts | 1,071–1,430 | 711–1,000 | 641–930 |
| 5 pts | 1,431+ | 1,001+ | 931+ |

**Color count is the primary lever** — going from 2→6 colors on a 300-bead pattern jumps from tier 2 to tier 4.

### Reputation Multiplier
- Formula: `1.0 + (reputation / 100)`
- At rep 0: 1.0x, rep 50: 1.5x, rep 100: 2.0x
- Applied at delivery time. Commission stores only basePayout.

### Losing Reputation
- Failed delivery (deadline missed): -2 rep
- No game over — player always recovers
- Failed delivery wastes materials but no rep loss if deadline hasn't passed

---

## 8. Economy Values

### Bead Pack Pricing
| Pack Size | Price | Cost Per Bead | Bulk Discount |
|---|---|---|---|
| 50 beads | 15 | 0.30 | — |
| 100 beads | 25 | 0.25 | 17% cheaper per bead |
| 250 beads | 50 | 0.20 | 33% cheaper per bead |

### Tool Costs
| Tool | Price | Unlock Condition |
|---|---|---|
| Tweezers | Free | Start |
| Bead Pen | 200 | None |
| Magnifying Glass | 250 | Rep 15+ |
| Bead Tweezers | 350 | Rep 20+ |
| Better Iron | 400 | Rep 25+ |
| Work Apron | 500 | Rep 25+ |
| Gloves | 600 | Rep 30+ |
| Projector | 800 | Rep 35+ |

### Pegboard Upgrades
| Board | Price | Unlock Condition |
|---|---|---|
| Small (25×25) | Free | Start |
| Medium (50×50) | 200 | None |
| Large (75×75) | 500 | Rep 25+, Medium owned |
| XL (100×100) | 1000 | Rep 35+, Large owned |

### Pattern Unlock Costs
| Tier | Price | Count |
|---|---|---|
| Simple | 30 | 10 |
| Moderate | 60 | 16 |
| Complex | 120 | 16 |

- Total purchasable: 42 patterns, total cost: 3,180 coins
- Shop only, no rep gate

### Ironing Paper
- Consumable, 1 sheet per ironing session
- Cost: 5 coins per sheet (10 sheets per pack)

---

## 9. Commission Structure

Each commission includes:
- **id** — Unique ID (format: `COM_XXX`)
- **name** — Short, catchy title
- **description** — What the client wants, written in their voice
- **clientName** — Character name from StoryWriter
- **deadline** — Days to complete (1-7)
- **basePayout** — Base coins earned (50-200). Actual payout = basePayout × reputationMultiplier
- **minReputationRequired** — Rep needed to see commission on board
- **requiredAxes** — Array of axis objects the delivered design must include
  - Each entry has `category`, `subject`, and optionally `variant`
  - Simple: 1 entry. Composite: 2-3 entries
  - When same Subject appears twice, each must have distinct Variant
- **storylineStep** — Position in storyline (1 = first)
- **colors** — Artkal bead codes needed
- **maxColors** — Maximum distinct colors allowed

### Commission Matching Rules
- Delivery must match ALL specified axes
- Unspecified axes are flexible
- Non-matching delivery = failed, materials wasted, no rep loss if deadline hasn't passed

---

## 10. Pattern System

### Multi-Axis Tags
Each design tagged across three axes:
- **Category** — broad domain (e.g., ANIMALS)
- **Subject** — specific thing (e.g., CAT)
- **Variant** — color-specific version (e.g., CAT:ORANGE)

### Design Variants
- One pixelart, multiple color palettes
- Shop purchase includes ALL variants
- Player locks a variant when selecting a design
- Commission matching requires Variant to match locked variant

### Deduplication Rules
- If two designs only differ by color → merge into ONE design with variants
- If two designs differ by more than color → keep separate

### Pattern JSON Schema
```json
{
  "id": "PAT_XXX",
  "name": "string",
  "description": "string",
  "subject": "string",
  "axes": {
    "category": "CATEGORY_ENUM",
    "subject": "SUBJECT_ENUM",
    "variant": "SUBJECT:COLOR" | null
  },
  "gridWidth": 12,
  "gridHeight": 12,
  "beadCount": null,
  "colors": [{"code": "CXX", "name": "Color Name"}],
  "interchangeableColors": [
    {
      "original": ["CXX"],
      "variants": [
        {"name": "Variant Name", "colors": ["CXX"], "variant": "SUBJECT:COLOR"}
      ]
    }
  ],
  "unlockCost": 0,
  "unlockType": "STARTING | PURCHASABLE | STORYLINE",
  "commissionsUsing": ["COM_XXX"]
}
```

**Rules:**
- `variant` is required (non-null) for designs without interchangeable colors
- `variant` is null for designs with interchangeable colors (determined by selected variant)
- `beadCount` starts as null — filled after pixelart delivery
- Grid minimum: 12×12 pegs
- Only base designs saved — composites NOT saved

---

## 11. Activity Time Costs

| Activity | Hours |
|---|---|
| Search Pattern | 0 (free) |
| Create Composite Pattern | 0 (free) |
| Place Beads | bead count × 0.00055, ceiled to 0.5 hr |
| Iron Design | 1 hr |
| Apply Masking Tape | 0.5 hrs (automatic) |
| Pack & Send Commission | 1 hr |
| Shop (Beads & Supplies) | 0 (free) |
| Accept Commission | 0 (free) |

---

## 12. Game Rules

- **Win condition:** Grow workshop via reputation. Open-ended.
- **Loss condition:** No game over. Low rep = reduced payouts, storyline pauses. Always recoverable.
- **Starting reputation:** 10 (from tutorial completion or skip)
- **Starting money:** 100 coins
- **Commission capacity:** 3 active commissions (fixed)
- **Day length:** 8 hours of work time per in-game day
- **Money flow:** Earned from commissions → spent at shop. No selling.
- **Bead waste:** Failed commissions waste beads + ironing paper. Masking tape always available.
