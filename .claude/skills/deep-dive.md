# /deep-dive — Concept Deep Dive

**Usage:** `/deep-dive <concept>`  
Performs an in-depth, interview-focused explanation of the given concept with analogies, cross-references, and mind-map placement.

---

## Instructions

### Step 1 — Locate the Concept
Find the concept in the active primer file (set by `/prep`) or in the most relevant primer if no active session. Read the relevant section(s) from the primer file to ground the explanation in actual source material.

### Step 2 — Structured Explanation
Deliver the deep dive in this exact order:

```
## Deep Dive: [Concept Name]
**Primer:** [Source primer] | **Section:** [Section name]

### The One-Line Crux
[What this is in ≤ 15 words — interview answer style]

### Real-World Analogy
[Analogy paragraph — make it vivid, concrete, memorable]

### How It Actually Works
[Technical explanation — bullet-heavy, no long paragraphs]
- Step 1: ...
- Step 2: ...

### Why It Matters (Interview Angle)
[What interviewers actually care about — trade-offs, failure modes, when NOT to use it]

### Key Terms
| Term | Definition (1 line) |
|------|---------------------|
| ... | ... |

### Cross-Primer Connections
- Related to **[Concept]** in [Other Primer] — [one sentence on how they connect]
(Only include concepts already covered per TRACKER.md. If nothing covered yet, skip section.)

### Mind Map Placement
> This concept lives under: [Primer] → [H3 section] → [H4 sub-concept]

### Common Interview Questions on This Topic
1. [Question]
2. [Question]
3. [Question]
```

### Step 3 — After the Deep Dive
- Ask: *"Want to go deeper on any part, test yourself with a question, or move to the next topic?"*
- Mark this topic as covered in `progress/TRACKER.md` (check the box for this concept)
- Update `progress/MINDMAPS.md` to expand the relevant node with key points from this deep dive
