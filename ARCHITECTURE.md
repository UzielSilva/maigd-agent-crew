# Architecture

## Agent Connections

```mermaid
graph TD
    User["👤 User<br/><i>Story summary + starting rep</i>"]

    subgraph Phase1["Phase 1: Story Review"]
        SW["StoryWriter<br/><i>Expands story into<br/>character messages</i>"]
    end

    subgraph Phase2["Phase 2: Commission Plan Review"]
        CP["CommissionPlanner<br/><i>Maps story steps to<br/>pattern axes</i>"]
        PD["PatternDesigner<br/><i>Simplifies requests into<br/>base patterns + axes</i>"]
        EB["EconomyBalancer<br/><i>Validates payouts<br/>against bead costs</i>"]
    end

    subgraph Phase3["Phase 3: Pattern Plan Review"]
        PD2["PatternDesigner<br/><i>Creates pattern JSONs<br/>+ pixelartist briefs</i>"]
    end

    subgraph Phase4["Phase 4: Final Output"]
        FS["File System<br/><i>Storylines/&lt;title&gt;/</i>"]
    end

    User -->|"1. Provide story summary"| SW
    SW -->|"2. Full story + commission steps"| User
    User -->|"3. Approve story"| CP

    CP -->|"4. Describe visual"| PD
    PD -->|"5. Base patterns + axes<br/>(or axis gap)"| CP
    CP -->|"6. Axis gap?"| User
    User -->|"7. Decide (revise/assume/simplify/skip)"| CP

    CP -->|"8. Commission list"| EB
    EB -->|"9. Validated payouts"| CP
    CP -->|"10. Commission table"| User
    User -->|"11. Approve commissions"| PD2

    PD2 -->|"12. Pattern JSONs + briefs"| User
    User -->|"13. Approve patterns"| FS
```

## Phase Flow with Human-in-the-Loop

```mermaid
flowchart TD
    Start(["Start: User provides story summary + starting rep"]) --> P1

    P1["Phase 1: Story Review<br/>StoryWriter expands story"]
    P1 --> P1Review{"User reviews story"}
    P1Review -->|"Request changes"| P1Revise["StoryWriter rethinks"]
    P1Revise --> P1
    P1Review -->|"Approve"| P2

    P2["Phase 2: Commission Plan Review<br/>CommissionPlanner + PatternDesigner<br/>map story steps to pattern axes"]
    P2 --> AxisCheck{"Axis gaps?"}
    AxisCheck -->|"Yes"| AxisGap{"User decides"}
    AxisGap -->|"Revise story"| P1
    AxisGap -->|"Assume axis added"| LogAxis["Log in GDD_TODO.md"]
    LogAxis --> P2Continue
    AxisGap -->|"Simplify"| P2
    AxisGap -->|"Skip step"| P2
    AxisCheck -->|"No gaps"| P2Validate["CommissionPlanner validates<br/>payouts with EconomyBalancer"]
    P2Validate --> P2Continue

    P2Continue["Present validated commission table"]
    P2Continue --> P2Review{"User reviews commissions"}
    P2Review -->|"Request changes"| P2Revise["Agents rethinks"]
    P2Revise --> P2Continue
    P2Review -->|"Approve"| P3

    P3["Phase 3: Pattern Plan Review<br/>PatternDesigner creates<br/>pattern JSONs + briefs"]
    P3 --> P3Review{"User reviews patterns"}
    P3Review -->|"Request changes"| P3Revise["PatternDesigner rethinks"]
    P3Revise --> P3
    P3Review -->|"Approve"| P4

    P4["Phase 4: Final Output<br/>Save all files to<br/>Storylines/&lt;title&gt;/"]
    P4 --> Done(["Done"])

    style P1 fill:#4a9eff,color:#fff
    style P2 fill:#ff9f43,color:#fff
    style P2Validate fill:#ff9f43,color:#fff
    style P3 fill:#2ecc71,color:#fff
    style P4 fill:#9b59b6,color:#fff
    style User fill:#e74c3c,color:#fff
```

## Agent Responsibilities

| Agent | Role | Phase(s) |
|-------|------|----------|
| **StoryWriter** | Expands story summaries into character messages with complexity hints | Phase 1 |
| **CommissionPlanner** | Maps story steps to pattern axes, assembles commission objects, validates payouts | Phase 2 |
| **PatternDesigner** | Simplifies requests into base patterns, checks axes, creates pattern JSONs and briefs | Phase 2, Phase 3 |
| **EconomyBalancer** | Validates payouts against bead costs and progression stage | Phase 2 |

## Cross-Agent Communication

| From | To | What |
|------|----|------|
| StoryWriter | CommissionPlanner | Story messages, narrative context, complexity hints |
| CommissionPlanner | PatternDesigner | Natural-language descriptions of what each pattern should show |
| PatternDesigner | CommissionPlanner | Base patterns with axes (or axis gap reports) |
| CommissionPlanner | EconomyBalancer | Full commission list for payout validation |
| EconomyBalancer | CommissionPlanner | Validated/adjusted payouts |
| PatternDesigner | Human Pixelartist | Pattern briefs (markdown) |
