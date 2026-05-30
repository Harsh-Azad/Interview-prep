# /prep — Interview Prep Session Initializer

**Usage:** `/prep [primer-name]`  
**Aliases for primer-name:** `ai`, `ml`, `gen-ai` | `sd`, `system-design` | `dbms`, `database` | `os`, `operating-system` | `cn`, `networks` | `devops`

---

## Instructions

### Step 1 — Identify Active Primer
- If argument provided, map to primer file:
  - `os` / `operating-system` → `Operating_System_Primer.md`
  - `dbms` / `database` → `DBMS_Primer.md`
  - `sd` / `system-design` → `System_Design_Primer.md`
  - `cn` / `networks` → `Computer_Networks_Primer.md`
  - `devops` → `Devops_Primer.md`
  - `ai` / `ml` / `gen-ai` → `AI-ML_Gen_AI_Advance_Primer.md`
- If no argument: read `progress/TRACKER.md`, recommend the primer with lowest coverage % or longest time since last visit. Present top 2 and let user choose.

### Step 2 — Read and Display Progress
Read `progress/TRACKER.md`. For the active primer extract and show:
- Checked topics (already covered)
- Unchecked topics (TODO list)
- Coverage % = checked / total
- Any cross-primer concept links (concepts in this primer that appear in other primers' covered topics)

### Step 3 — Suggest Today's Focus
Pick 2–3 uncovered topics from TRACKER.md for this session. If primer is untouched, start from the first major section. Give a one-line reason for each suggestion.

### Step 4 — Set Session Behavior (persist for entire conversation)
After `/prep` runs, maintain this behavior throughout the session:
- **Active primer** = the identified primer. All explanations draw from it.
- **Cross-linking**: when explaining a concept, check TRACKER.md for related covered concepts in other primers. Naturally weave in: *"This is similar to X we covered in [Primer]"* or *"Recall [concept] from [Primer] — same idea applies here."*
- **TODO surfacing**: every 3–4 exchanges, naturally mention an uncovered topic: *"We haven't touched [topic] yet — want to get to that next?"*
- **Analogy-first**: default to real-world analogies before technical definitions.
- **Session log**: mentally track which concepts were discussed. These will be written to TRACKER.md when `/revise` is called.

### Step 5 — Output Format
```
## Session Start — [Primer Name]
**Date:** [today's date]

### Progress
- Covered: X / Y topics ([Z]%)
- Stale (not visited in 2+ sessions): [list or "none"]

### TODO — Remaining Topics
- [ ] Topic A
- [ ] Topic B
- [ ] Topic C
...

### Suggested Focus Today
1. **[Topic]** — [one-line reason why]
2. **[Topic]** — [one-line reason why]

### Cross-Primer Hooks
- [This concept] ↔ [Related concept] in [Other Primer] (covered [date])
(or "None yet — this is the first primer!" if tracker is empty)
```

---

## Updating TRACKER.md
When the user marks a topic as done, or when `/revise` is called, update the relevant checkbox in `progress/TRACKER.md` and update the `Last visited` and `Sessions` fields for the active primer.
