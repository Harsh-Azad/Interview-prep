# /jd-new — Scaffold a New Company Prep Folder

**Usage:** `/jd-new <CompanyName> <Role-Slug>`  
**Example:** `/jd-new BMW AI-Engineer`

This creates the company folder and an empty `JD-raw.md` for you to paste the JD into manually.  
After filling JD-raw.md, run `/jd-prep <company-slug>` to process it — the JD never needs to enter the chat.

---

## Instructions

### Step 1 — Derive the slug
- Slug = `[CompanyName]-[RoleSlug]` with no spaces (e.g., `BMW-AI-Engineer`)
- Create folder path: `companies/[slug]/`
- Create subfolder: `companies/[slug]/interview-rounds/`

### Step 2 — Create JD-raw.md (empty template)
Write `companies/[slug]/JD-raw.md` with this content exactly:

```markdown
# JD Raw — [CompanyName] [Role]
*Paste the full job description below. Save the file, then run `/jd-prep [slug]`.*
*This file is read by the skill — do NOT paste the JD in chat.*

---

[PASTE JD HERE]
```

### Step 3 — Output (keep it short)
```
## Folder created: companies/[slug]/

Paste the JD into:
  companies/[slug]/JD-raw.md

Then run:
  /jd-prep [slug]
```

Do not output anything else. No summaries, no analysis — the JD hasn't been read yet.
