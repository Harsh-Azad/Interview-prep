# /daily-brief — Daily Study Brief

**Usage:** `/daily-brief`  
Gives a smart morning briefing: what's stale, what to focus on today, and 3 warm-up questions.

---

## Instructions

### Step 1 — Read State
Read `progress/TRACKER.md` in full.

### Step 2 — Analyze Coverage
For each primer:
- Calculate coverage % (checked / total topics)
- Identify stale primers (last visited 3+ sessions ago or 7+ days ago)
- Identify the weakest primer (lowest coverage %)

### Step 3 — Generate Brief
Output in this format:
```
## Daily Brief — [Today's Date]

### Overview
| Primer | Coverage | Last Session | Status |
|--------|----------|--------------|--------|
| AI-ML  | 40%      | 2026-05-28   | 🔄 Active |
| SD     | 10%      | 2026-05-20   | ⚠️ Stale |
| DBMS   | 0%       | —            | ⬜ Untouched |
...

### Today's Recommendation
**Focus:** [Primer name]  
**Why:** [one sentence — e.g., "lowest coverage" or "not touched in 5 sessions"]

**Suggested topic order for today:**
1. [Topic] — [why: foundational / prerequisite for X / stale]
2. [Topic]
3. [Topic]

### 3 Warm-Up Questions
*(from topics already covered across all primers)*
1. [Q] → *[Primer, topic]*
2. [Q] → *[Primer, topic]*
3. [Q] → *[Primer, topic]*

### Cross-Primer Insight of the Day
[One interesting connection between a concept from today's focus primer and something already covered in another primer]
```

If TRACKER.md is empty (first session ever), skip the analysis and output:
```
## Daily Brief — First Session!
No history yet. Recommended starting primer: **System Design** (highest ROI for interviews).
Run `/prep sd` to begin.
```

### Step 4 — Follow-Up
After the brief, ask: *"Want to jump into today's focus with `/prep [primer]`, or do the warm-up questions first?"*
