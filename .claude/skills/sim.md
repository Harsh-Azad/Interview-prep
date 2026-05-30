# /sim — Mock Interview Simulator

**Usage:** `/sim [primer] [easy|medium|hard]`  
Runs a structured mock interview session for the chosen primer. Defaults to active primer and medium difficulty.

---

## Instructions

### Step 1 — Setup
Determine:
- **Primer**: from argument or active session (set by `/prep`)
- **Difficulty**:
  - `easy` — definition-level, single-concept questions
  - `medium` — scenario-based, trade-off questions (default)
  - `hard` — system design or multi-concept synthesis questions

Draw questions only from topics marked as covered ✅ in `progress/TRACKER.md` for the chosen primer. If fewer than 3 topics are covered, warn the user and suggest studying more first (offer to run `/prep` instead).

### Step 2 — Session Format
Run a 5-question session. Present one question at a time:

```
## Mock Interview — [Primer] ([Difficulty])
**Question [N]/5**

[Question text]

> Type your answer, then press Enter. I'll evaluate it.
```

Wait for user's answer before showing the next question or any feedback.

### Step 3 — Evaluate Each Answer
After the user answers, provide:
```
### Evaluation
**Score:** [X/10]
**What you got right:** [brief]
**What was missing:** [brief — max 2 points]
**Model answer (concise):** [1–3 bullets]
---
**Question [N+1]/5:** [next question]
```

Keep evaluations tight — this is a quiz, not a lecture. Save deep explanations for `/deep-dive`.

### Step 4 — Final Scorecard
After all 5 questions:
```
## Session Complete
**Final Score:** [X]/50 ([Y]%)

| Q | Topic | Score | Verdict |
|---|-------|-------|---------|
| 1 | [topic] | [X/10] | ✅ Strong / ⚠️ Shaky / ❌ Gap |
...

### Weak Spots Flagged
- [Topic] — consider `/deep-dive [topic]` to reinforce
```

Flag any topic scored ≤ 5 as a weak spot. Suggest follow-up actions.

### Step 5 — Post-Session
Ask: *"Want another round, go deeper on a weak spot, or call `/revise` to lock in what you know?"*
