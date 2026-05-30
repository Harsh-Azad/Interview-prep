# Interview Prep

AI-powered interview prep system built on Claude Code. Six technical primers, custom study skills, and per-company prep folders with roadmaps, mind maps, mock interview rounds, and revision notes.

## Primers
| File | Topic |
|------|-------|
| `AI-ML_Gen_AI_Advance_Primer.md` | AI, ML, Gen AI, LLMs, Agents |
| `System_Design_Primer.md` | System design, scalability, distributed systems |
| `DBMS_Primer.md` | Databases, SQL, NoSQL, vector DBs |
| `Operating_System_Primer.md` | OS, processes, memory, concurrency |
| `Computer_Networks_Primer.md` | Networking, HTTP, DNS, TLS |
| `Devops_Primer.md` | Docker, K8s, CI/CD, observability |

## Skills (type in Claude Code chat)

### Daily use
| Command | What it does |
|---------|-------------|
| `/daily-brief` | Today's focus: stale topics, suggested order, warm-up Qs |
| `/prep [primer]` | Start a study session; shows progress + TODO list |
| `/deep-dive <concept>` | In-depth explanation with analogies + cross-primer links |
| `/mind-map [topic]` | Heading-structured overview; expandable nodes |
| `/sim [primer] [easy\|medium\|hard]` | Mock quiz on covered topics |
| `/revise` | Append session learnings to `REVISION.md` in clean citation format |

### Company prep
| Command | What it does |
|---------|-------------|
| `/jd-new <Company> <Role>` | Scaffold a new company folder + empty `JD-raw.md` |
| `/jd-prep <company-slug>` | Process JD from file → roadmap, mindmap, topic tracker, round stubs |
| `/interview-round <slug> <round>` | PYQ-sourced simulation round; saves scored results to company folder |

## Workflow for a new job opening
```
/jd-new Stripe Backend-Engineer
# paste JD into companies/Stripe-Backend-Engineer/JD-raw.md
/jd-prep Stripe-Backend-Engineer
```

## Structure
```
├── progress/
│   ├── TRACKER.md      # 60-topic exhaustive checklist across all primers
│   └── MINDMAPS.md     # mind maps (auto-generated on first /mind-map call)
├── REVISION.md         # global revision notes (updated by /revise)
├── companies/
│   └── BMW-AI-Engineer/
│       ├── JD-raw.md, JD.md, ROADMAP.md
│       ├── TOPIC-COVERAGE.md, MINDMAP.md, REVISION.md
│       └── interview-rounds/   # round stubs + RESULTS-SUMMARY.md
└── .claude/skills/     # skill definitions
```
