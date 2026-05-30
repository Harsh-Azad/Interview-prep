# /interview-round — Company-Specific Interview Simulation

**Usage:** `/interview-round <company-slug> <round-type>`  
**company-slug:** matches the folder name in `companies/` (e.g., `Google-Senior-SWE`)  
**round-type:** `screening` | `technical-1` | `technical-2` | `system-design` | `behavioral` | `managerial` | `case-study` | `bar-raiser`

---

## Instructions

### Step 1 — Load Company Context
Read the following files from `companies/[company-slug]/`:
- `JD.md` — role requirements, seniority, inferred round format
- `ROADMAP.md` — Tier 1/2 topics, interview schedule
- `TOPIC-COVERAGE.md` — which topics are marked covered (use only covered topics for primary questions; uncovered topics can appear as stretch questions)
- `interview-rounds/RESULTS-SUMMARY.md` — check prior round results to adjust difficulty (if past rounds show weak spots, focus questions there)

### Step 2 — Research Past Interview Questions (PYQ)
Use WebSearch to find real past questions for this company and round type. Run these searches:
1. `"[Company]" "[Role OR domain]" "[round-type]" interview questions site:leetcode.com/discuss`
2. `"[Company]" "[Role]" interview experience "[round-type]" site:glassdoor.com`
3. `"[Company]" SWE OR engineer interview "[round-type]" questions site:teamblind.com`
4. `"[Company]" interview questions "[year]" "[topic from Tier 1]" site:geeksforgeeks.org OR site:interviewbit.com`

Extract 8–12 real questions. Cite the source URL for each question used.

For **behavioral/managerial rounds**, also search:
- `"[Company]" leadership principles behavioral questions`
- `"[Company]" manager interview questions site:glassdoor.com`

### Step 3 — Build Question Set
Compose a set of questions for the round. Mix:
- 60% from PYQ research (cite sources)
- 40% generated based on JD Tier 1 requirements + role seniority

**By round type:**

#### `screening`
- 2 resume/background questions (e.g., "Walk me through your most impactful project")
- 1–2 lightweight technical questions (concept definitions, trade-offs)
- 1 motivation question ("Why [Company]? Why this role?")
- Total: 4–5 questions, 30-min format

#### `technical-1` / `technical-2`
- 1–2 coding-style concept questions (not actual LeetCode problems, but conceptual)
- 2–3 deep technical questions on Tier 1 topics from JD
- 1 debugging/troubleshooting scenario question
- 1 trade-off question ("How would you choose between X and Y?")
- Total: 5–7 questions, 45-min format

#### `system-design`
- 1 open-ended system design question (at appropriate scale for seniority)
  - Junior: design a simple CRUD service
  - Senior: design a real-time feature, distributed cache, notification system
  - Staff/Principal: design at internet scale with fault tolerance + cost constraints
- 3–4 follow-up drill-down questions (scaling, failure modes, trade-offs, monitoring)
- Total: 1 main + 4 follow-ups, 60-min format

#### `behavioral`
- 5–7 behavioral questions using STAR format
- For companies with published frameworks (e.g., Amazon Leadership Principles, Google's Googleyness), tailor questions to those specific principles
- Cite which LP/value each question targets

#### `managerial` / `bar-raiser`
- Mix of behavioral, leadership, and technical judgment questions
- Include questions on team conflict, delivery under pressure, cross-functional collaboration
- Include at least 1 "tell me about a failure" question

#### `case-study`
- 1 business/product case question relevant to the company's domain
- Follow-up questions on metrics, prioritization, trade-offs

---

### Step 4 — Conduct the Simulation
Present one question at a time. Format:

```
## [Company] — [Round Type] Simulation
**Round [N] of [Total rounds from ROADMAP]**
*Based on PYQ from: [source list]*

---

**Question [X] / [Total]**
> [Question text]

*[Source: URL or "Generated from JD requirements"]*

Type your answer — I'll wait for you to finish before evaluating.
```

Wait for the user's answer after each question. Do not show evaluation until the user has answered.

**For system design:** after the user gives their initial design, ask each follow-up question one at a time.

---

### Step 5 — Evaluate Each Answer
After each answer:

```
### Evaluation — Q[X]
**Score:** [X / 10]

**Strong points:**
- [What they got right]

**Gaps:**
- [Missing concept or angle — be specific, max 2 points]

**Model answer (concise):**
[3–5 bullets maximum — interview-ready, not tutorial-style]

**Source:** *[cite primer section if applicable: Primer §Section]*
```

For system design, evaluate the full design at the end with a rubric:
- Requirements clarification: [X/10]
- High-level design: [X/10]
- Deep dive quality: [X/10]
- Trade-off discussion: [X/10]
- Handling failures: [X/10]

---

### Step 6 — End-of-Round Scorecard
After all questions:

```
## Round Complete — [Company] [Round Type]
**Date:** [today]
**Overall Score:** [X] / [Total] ([Y]%)

### Question-by-Question
| Q | Topic | Score | Verdict |
|---|-------|-------|---------|
| 1 | [topic] | [X/10] | ✅ Strong / ⚠️ Shaky / ❌ Gap |
...

### Verdict: [PASS / BORDERLINE / FAIL]
*Pass threshold: ≥70% for this round type*

### Weak Spots — Action Items
| Topic | Gap | Fix |
|-------|-----|-----|
| [topic] | [what was missing] | `/deep-dive [topic]` |
...

### PYQ Sources Used
- [URL 1]
- [URL 2]
...
```

**Verdict thresholds by round:**
- Screening: ≥65% = Pass
- Technical: ≥70% = Pass
- System Design: ≥65% = Pass (design rounds are qualitative)
- Behavioral: ≥70% = Pass
- Bar Raiser / Managerial: ≥75% = Pass

---

### Step 7 — Save Results
Write the full session (questions + answers + evaluations + scorecard) to:
`companies/[company-slug]/interview-rounds/round-[N]-[type].md`

Update `companies/[company-slug]/interview-rounds/RESULTS-SUMMARY.md`:
- Add a row to the results table with today's date, score, verdict, and weak spots

Update `progress/TRACKER.md`:
- Under `## Company Prep Cross-Reference`, mark any topics tested in this round as "Tested via simulation" with the date

---

### Step 8 — Post-Round Prompt
After saving results:
```
Results saved to companies/[slug]/interview-rounds/round-[N]-[type].md

Next steps:
- Fix weak spots: `/deep-dive [topic]` for each gap flagged above
- Run next round: `/interview-round [slug] [next-round-type]`
- Lock in knowledge: `/revise` to update company revision notes
- View all round results: read `companies/[slug]/interview-rounds/RESULTS-SUMMARY.md`
```
