# /revise — Revision Module Updater

**Usage:** `/revise [view|update|primer-name]`  
- `/revise` or `/revise update` — append this session's topics to REVISION.md
- `/revise view` — display current REVISION.md content
- `/revise [primer]` — view only that primer's section in REVISION.md

---

## Instructions

### On `/revise update` (default)

#### Step 1 — Collect Session Topics
Identify all concepts discussed in the current conversation since `/prep` was called (or since the conversation started). These are the topics to write revision notes for.

#### Step 2 — Read Existing REVISION.md
Read `REVISION.md`. Do not overwrite existing content — only append or update.

#### Step 3 — Write Revision Entries
For each concept discussed this session, add or update its entry under the correct primer heading. If an entry already exists, enhance it rather than duplicate.

**Entry format — strict:**
```markdown
### [Concept Name] `[PRIMER-CODE]-[NNN]`
> [One-sentence crux — what it is and why it matters]
- [Bullet: key fact or mechanism]
- [Bullet: key trade-off or failure mode]
- [Bullet: interview angle or gotcha]
- *Ref: [Primer filename] §[Section name]*
- *Also see: [[Related concept code]] in [Primer]*
```

**Rules:**
- Max 5 bullets per entry
- No paragraphs — bullets only
- Every entry must have a `*Ref:*` citation to the source primer section
- Keep language dense and interview-ready, not tutorial-style
- Primer codes: `AI` | `SD` | `DB` | `OS` | `CN` | `DO`
- IDs are sequential per primer (e.g., `SD-001`, `SD-002`)

#### Step 4 — Update TRACKER.md
Mark all concepts discussed this session as ✅ covered in `progress/TRACKER.md`. Update `Last visited` and `Sessions` count for the active primer.

#### Step 5 — Confirm What Was Added
After writing, output:
```
## Revision Updated ✓
**Primer:** [Name]  
**Added/updated [N] entries:**
- [Concept 1] → [PRIMER-CODE-NNN]
- [Concept 2] → [PRIMER-CODE-NNN]

**TRACKER.md:** [N] topics marked covered.
```

---

### On `/revise view`
Display the full `REVISION.md` content as-is, formatted cleanly.

---

## REVISION.md Structure
The file is organized as:
```markdown
# Interview Revision Notes
*Last updated: [date] | Total entries: [N]*

---

## AI-ML & Gen AI

### [Concept] `AI-001`
...

---

## System Design

### [Concept] `SD-001`
...

---
## DBMS
...
## Operating System
...
## Computer Networks
...
## DevOps
...
```
