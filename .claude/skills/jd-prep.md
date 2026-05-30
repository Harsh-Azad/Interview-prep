# /jd-prep — Job Description Analyzer & Prep Setup

**Usage:** `/jd-prep`  
Paste a JD directly into the conversation (or attach a file), then invoke this skill.  
Creates a fully structured company prep folder and roadmap.

---

## Instructions

### Step 1 — Extract JD Intelligence
Parse the JD the user has provided in the conversation. Extract and structure:

**Company & Role**
- Company name (clean, no spaces — e.g., `Google`, `Stripe`, `MorganStanley`)
- Role title (slugified — e.g., `Senior-SWE`, `ML-Engineer`, `Backend-Engineer`)
- Seniority level: Junior / Mid / Senior / Staff / Principal / Lead / Manager
- Team/domain context (if mentioned)

**Technical Requirements** — categorize into:
- Languages & runtimes (Python, Go, Java, etc.)
- Frameworks & libraries
- Infrastructure & cloud (AWS, GCP, K8s, etc.)
- Data & ML tools (if any)
- Core CS concepts required (identify which of the 6 primers they map to)

**Managerial / Leadership Requirements** (extract if present):
- People management expectations
- Cross-functional collaboration
- Project ownership / delivery expectations
- Communication and stakeholder management

**Inferred Interview Format**
Use WebSearch to look up: `"[Company] [Role] interview process site:leetcode.com OR site:glassdoor.com OR site:levels.fyi OR site:teamblind.com`  
Extract typical rounds: Screening / Technical / System Design / Behavioral / Case Study / Leadership / Bar Raiser / etc.  
If no search results, infer format from role seniority:
- Junior/Mid SWE: Screening → 2× Technical → Behavioral
- Senior SWE: Screening → 2× Technical → System Design → Behavioral
- Staff/Principal: Screening → Technical → 2× System Design → Leadership/Behavioral
- Manager/Lead: Screening → Technical (light) → Behavioral/Leadership × 2 → Case Study

---

### Step 2 — Map to Global Tracker
Read `progress/TRACKER.md`. For each technical requirement in the JD, map it to specific topics in the exhaustive list. Produce:
- **Must-cover topics**: directly required by JD (map exact topic names from TRACKER.md)
- **Supporting topics**: not explicitly required but foundational
- **Stretch topics**: nice-to-have given the company/domain

---

### Step 3 — Create Company Folder
Create the following directory and files using exact paths below.  
Folder: `companies/[CompanyName]-[RoleSlug]/`

#### 3a — `JD.md`
```markdown
# [Company] — [Role Title]
*Extracted: [date]*

## Role Summary
[2–3 sentence summary of the role]

## Seniority
[Level]

## Technical Stack
| Category | Technologies |
|----------|-------------|
| Languages | ... |
| Frameworks | ... |
| Infrastructure | ... |
| Data/ML | ... |

## Core CS Requirements
[Bullet list mapped to primer topics]

## Managerial Requirements
[Bullet list or "Not applicable"]

## Inferred Interview Format
| Round | Type | Focus | Duration (est.) |
|-------|------|-------|-----------------|
| 1 | Screening | ... | 30 min |
| 2 | Technical | ... | 45–60 min |
...

## Sources
[Links to Glassdoor/Blind/LeetCode discussions found during research]
```

#### 3b — `ROADMAP.md`
```markdown
# Prep Roadmap — [Company] [Role]
*Generated: [date] | Target readiness: [suggest timeline based on coverage gaps]*

## Priority Tiers
### Tier 1 — Must Know (JD explicitly requires)
- [ ] [Topic] → [Primer §Section] — *why it matters for this role*
...

### Tier 2 — Strong Signal (common at this company / level)
- [ ] [Topic] → [Primer §Section]
...

### Tier 3 — Good to Have
- [ ] [Topic] → [Primer §Section]
...

## Managerial Prep (if applicable)
- [ ] STAR stories for: [specific scenarios from JD]
- [ ] Leadership principles: [company-specific if applicable, e.g., Amazon LPs]
- [ ] Case study prep: [if required]

## Week-by-Week Plan
### Week 1 — Foundations
- Day 1–2: [Topic] via /deep-dive [topic]
- Day 3–4: [Topic] via /deep-dive [topic]
- Day 5: /sim [primer] to test
- Weekend: /revise → update revision notes

### Week 2 — Core Technical
[Continue...]

### Week 3 — System Design + Behavioral
[Continue...]

### Final Week — Mock Rounds
- Run all interview rounds: /interview-round [company-slug] [round]
- Review weak spots from round results
- Final /revise pass

## Interview Round Schedule
| Round | Command | Focus |
|-------|---------|-------|
| Screening | `/interview-round [slug] screening` | Resume + 1–2 technical Qs |
| Technical 1 | `/interview-round [slug] technical-1` | [Topics] |
| Technical 2 | `/interview-round [slug] technical-2` | [Topics] |
| System Design | `/interview-round [slug] system-design` | [Scale/domain from JD] |
| Behavioral | `/interview-round [slug] behavioral` | [Company values/LPs] |
```

#### 3c — `TOPIC-COVERAGE.md`
```markdown
# Topic Coverage — [Company] [Role]
*Synced from global TRACKER.md | Last updated: [date]*

## Tier 1 — Must Cover
- [ ] [Topic from TRACKER] ← [source primer] | Global ID: [PRIMER-NNN]
...

## Tier 2 — Should Cover  
- [ ] [Topic] ← [source primer]
...

## Tier 3 — Optional
- [ ] [Topic] ← [source primer]
...

## Managerial / Behavioral Topics
- [ ] [Company]-specific leadership principles or values
- [ ] STAR story bank (add entries as you prep)
...

## Coverage Progress
- Tier 1: 0 / [N] (0%)
- Tier 2: 0 / [N] (0%)
- Overall: 0 / [Total] (0%)
```

#### 3d — `MINDMAP.md`
```markdown
# Mind Map — [Company] [Role]
*Role-specific mind map. Run /mind-map within this company context to expand nodes.*

## [Role Title] Skill Tree

### [Primer / Domain 1]
#### [Core Concept]
- Key fact
- Interview angle
#### [Core Concept]
...

### [Primer / Domain 2]
...

### Managerial / Leadership (if applicable)
#### People Management
...
#### Stakeholder Communication
...
```

#### 3e — `REVISION.md`
```markdown
# Revision Notes — [Company] [Role]
*Updated by /revise during prep sessions for this company.*
*Format: clean bullets, citations to primer sections.*

---

## [Primer / Domain]

<!-- Entries added here by /revise -->
```

#### 3f — `interview-rounds/` folder with stub files
Create one stub `.md` per inferred round:
```
interview-rounds/
├── round-1-screening.md
├── round-2-technical-1.md
├── round-3-technical-2.md   (if applicable)
├── round-4-system-design.md (if applicable)
├── round-5-behavioral.md
└── RESULTS-SUMMARY.md
```

Each stub format:
```markdown
# Round [N]: [Type] — [Company] [Role]
*Status: ⬜ Not Attempted*

## Focus Areas
[From ROADMAP.md]

## To run this round:
`/interview-round [company-slug] [round-type]`

## Results
(Populated after simulation)
```

`RESULTS-SUMMARY.md` stub:
```markdown
# Interview Rounds Summary — [Company] [Role]

| Round | Type | Date | Score | Verdict | Weak Spots |
|-------|------|------|-------|---------|------------|
| 1 | Screening | — | — | — | — |
...
```

---

### Step 4 — Update Global TRACKER.md
Append to the bottom of `progress/TRACKER.md` under the `## Company Prep Cross-Reference` section (create it if it doesn't exist):

```markdown
## Company Prep Cross-Reference
*Topics covered via company-specific prep are logged here. Check the box in the relevant primer section above when completed.*

| Topic | Primer | Company Prep | Folder | Date Covered |
|-------|--------|--------------|--------|--------------|
| [topic] | [Primer name] | [Company-Role] | `companies/[slug]/` | — |
...
```

---

### Step 5 — Output Confirmation
After all files are created, output:

```
## JD Prep Setup Complete ✓

**Company:** [Name]  
**Role:** [Title] ([Seniority])  
**Folder:** `companies/[CompanyName]-[RoleSlug]/`

### What was created:
- `JD.md` — extracted role intelligence + interview format research
- `ROADMAP.md` — [N]-week prep plan with [X] Tier 1 topics
- `TOPIC-COVERAGE.md` — [N] topics mapped from global tracker
- `MINDMAP.md` — role-specific skill tree
- `REVISION.md` — ready for session notes
- `interview-rounds/` — [N] round stubs ready

### Global TRACKER.md updated:
[N] topics cross-referenced under Company Prep section.

### Start prep:
1. `/prep [primer]` — begin studying Tier 1 topics
2. `/daily-brief` — get today's focus
3. `/interview-round [company-slug] screening` — when ready for first mock round
```
