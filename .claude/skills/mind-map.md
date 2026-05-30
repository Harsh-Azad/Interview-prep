# /mind-map — Concept Mind Map Navigator

**Usage:** `/mind-map [topic]`  
If no topic given, show the mind map for the active primer (set by `/prep`). If no active primer, ask which one.

---

## Instructions

### Step 1 — Determine Scope
- With argument: show mind map for that specific concept or section within the active primer
- Without argument: show the full top-level mind map for the active primer

### Step 2 — Read Existing Mind Maps
Read `progress/MINDMAPS.md`. If a map exists for the requested primer/topic, display it. If not, generate one from the corresponding primer `.md` file and append it to `MINDMAPS.md`.

### Step 3 — Display Format
Use strictly **heading-based** structure. Never use paragraphs in the map itself:
```
## [Primer Name] — Mind Map

### [Major Concept 1]
#### [Sub-concept A]
- Key point
- Key point
#### [Sub-concept B]
- Key point

### [Major Concept 2]
#### [Sub-concept A]
...
```

- H2 = Primer section / chapter
- H3 = Core concept
- H4 = Sub-concept or mechanism
- Bullets = atomic facts, only 1 line each
- Mark covered topics with ✅, uncovered with ⬜ (cross-reference TRACKER.md)

### Step 4 — Offer Navigation
After displaying the map, always offer:
```
**Navigate:**
- Type `/deep-dive [concept]` to expand any node
- Type `/mind-map [concept]` to zoom into a sub-tree
- Type a concept name to discuss it inline
```

### Step 5 — Cross-Primer Connections
After the map, add a short section:
```
### 🔗 Cross-Primer Links
- [Concept here] ↔ [Concept] in [Other Primer]
```
Only include links where the other primer's topic is already covered (check TRACKER.md).

---

## Updating MINDMAPS.md
When generating a new mind map, append it to `progress/MINDMAPS.md` under a `## [Primer Name]` heading. When deep-diving expands a node, update the relevant section in MINDMAPS.md to reflect the expanded understanding.
