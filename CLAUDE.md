# Interview Prep — Session Bootstrap

## What this repo is
An AI-powered interview prep system with 6 technical primers, custom study skills,
and per-company prep folders. The user is preparing for software/AI engineering roles.

## Context hygiene rules (CRITICAL)
These rules exist to keep conversation context lean across long sessions:

1. **Never dump full file contents into conversation.** Read files silently; output only summaries.
2. **Never echo a JD back.** JDs live in `companies/[slug]/JD-raw.md` — read from there, never from chat.
3. **Never print TRACKER.md in full.** When progress is needed, extract only the relevant primer section or a summary row.
4. **Never print full primer content.** When studying a topic, read only the relevant section — not the whole file.
5. **Skills output summaries, not raw file content.** Each skill result should fit in ~20 lines.
6. **Cross-session state lives in files, not conversation.** At the start of every new conversation, the files are the source of truth.

## Session startup (auto-read these, silently)
On every new conversation in this repo, silently read:
- `progress/TRACKER.md` — just the per-primer Status/Sessions/Last-visited header lines (not the full topic lists)
- `companies/*/interview-rounds/RESULTS-SUMMARY.md` — only if a company-specific question is asked

Then greet with:
```
Interview Prep ready.
Primers: [list each with status emoji and coverage %]
Active company preps: [list folders in companies/ that have real content]

Run /daily-brief for today's focus, or /prep [primer] to start studying.
```

If this is the very first session (all primers show 0%), say:
```
Fresh start. 6 primers loaded, 0 topics covered.
Run /daily-brief or /jd-new [Company] [Role] to begin.
```

## File map (read on demand, not upfront)
| Purpose | File | Read when |
|---------|------|-----------|
| Full topic list | `progress/TRACKER.md` | `/prep`, `/daily-brief`, `/jd-prep` |
| Mind maps | `progress/MINDMAPS.md` | `/mind-map` |
| Global revision | `REVISION.md` | `/revise` |
| Raw JD | `companies/[slug]/JD-raw.md` | `/jd-prep [slug]` only |
| Company roadmap | `companies/[slug]/ROADMAP.md` | `/interview-round`, company-specific questions |
| Topic coverage | `companies/[slug]/TOPIC-COVERAGE.md` | `/interview-round`, `/jd-prep` |
| Round results | `companies/[slug]/interview-rounds/RESULTS-SUMMARY.md` | asked about round history |
| Primer content | `*_Primer.md` | `/deep-dive`, `/mind-map` — read only the relevant section |

## Skills available
| Command | File |
|---------|------|
| `/jd-new <Company> <Role>` | `.claude/skills/jd-new.md` |
| `/jd-prep <slug>` | `.claude/skills/jd-prep.md` |
| `/prep [primer]` | `.claude/skills/prep.md` |
| `/mind-map [topic]` | `.claude/skills/mind-map.md` |
| `/deep-dive <concept>` | `.claude/skills/deep-dive.md` |
| `/revise` | `.claude/skills/revise.md` |
| `/sim [primer] [difficulty]` | `.claude/skills/sim.md` |
| `/daily-brief` | `.claude/skills/daily-brief.md` |
| `/interview-round <slug> <round>` | `.claude/skills/interview-round.md` |

## Primers in this repo
| File | Short name | Slug |
|------|-----------|------|
| `AI-ML_Gen_AI_Advance_Primer.md` | AI-ML & Gen AI | `ai` |
| `System_Design_Primer.md` | System Design | `sd` |
| `DBMS_Primer.md` | DBMS | `dbms` |
| `Operating_System_Primer.md` | OS | `os` |
| `Computer_Networks_Primer.md` | Networks | `cn` |
| `Devops_Primer.md` | DevOps | `devops` |
