# Sapiom Skills — Project Plan

## Final Repo Structure

```
sapiom/skills (forked)
│
├── skills/
│   │
│   ├── sapiom/                          ← EXISTS (don't touch)
│   │   └── SKILL.md
│   │
│   ├── sapiom-web-search/               ← NEW
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── search-api.md
│   │
│   ├── sapiom-verify/                   ← NEW
│   │   └── SKILL.md
│   │
│   ├── sapiom-ai-models/                ← NEW
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── models-api.md
│   │
│   ├── sapiom-images/                   ← NEW
│   │   └── SKILL.md
│   │
│   ├── sapiom-audio/                    ← NEW
│   │   └── SKILL.md
│   │
│   └── sapiom-browser/                  ← NEW
│       ├── SKILL.md
│       └── references/
│           └── browser-api.md
│
├── README.md                            ← UPDATE (add 6 new skills to table)
├── LICENSE                              ← unchanged
└── .gitignore                           ← unchanged
```

**Notes on structure decisions:**
- `sapiom-verify`, `sapiom-images`, and `sapiom-audio` are simple enough to
  live entirely in one `SKILL.md` — no references file needed.
- `sapiom-web-search`, `sapiom-ai-models`, and `sapiom-browser` get a
  `references/` file because they have meaningful parameter/model/endpoint
  complexity that would bloat the main skill file.
- Folder naming follows kebab-case to match the existing `sapiom` convention.

---

## Tickets

Each ticket maps to one unit of work in Cursor. Paste the listed docs pages
into this chat before starting the ticket so the skill content is grounded
in real API details.

---

### TICKET 0 — Research & Setup
**Type:** Setup  
**Estimated time:** 20 min  
**No docs paste needed**

**Tasks:**
- Fork `sapiom/skills` on GitHub
- Clone your fork locally
- Open in Cursor
- Read the existing `skills/sapiom/SKILL.md` to understand the current style
  and format before writing anything new

**Done when:** You have the repo open in Cursor and have read the existing skill.

---

### TICKET 1 — sapiom-web-search
**Type:** New skill  
**Estimated time:** 45 min

**Docs to paste into chat first:**
- `docs.sapiom.ai/quick-start/`
- `docs.sapiom.ai/capabilities/search/`
- `docs.sapiom.ai/reference/sdk/fetch/`

**Files to create:**
- `skills/sapiom-web-search/SKILL.md`
- `skills/sapiom-web-search/references/search-api.md`

**Skill should cover:**
- When to use Linkup vs You.com
- `standard` vs `deep` depth modes and cost implications
- SDK code example for a basic search
- Pattern for grounding an LLM response with search results
- Error handling (402 insufficient credits, 429 rate limit)

---

### TICKET 2 — sapiom-verify
**Type:** New skill  
**Estimated time:** 30 min

**Docs to paste into chat first:**
- `docs.sapiom.ai/capabilities/verify/`
- `docs.sapiom.ai/reference/sdk/fetch/` (if not already pasted)

**Files to create:**
- `skills/sapiom-verify/SKILL.md`

**Skill should cover:**
- Phone vs email verification flows
- The two-step pattern: `send` then `check`
- SDK code example for the full verify flow
- When to use this vs rolling your own Twilio/SendGrid setup
- Error handling (invalid number, code expired, max attempts)

---

### TICKET 3 — sapiom-ai-models
**Type:** New skill  
**Estimated time:** 50 min

**Docs to paste into chat first:**
- `docs.sapiom.ai/capabilities/ai-models/`
- `docs.sapiom.ai/reference/sdk/fetch/`

**Files to create:**
- `skills/sapiom-ai-models/SKILL.md`
- `skills/sapiom-ai-models/references/models-api.md`

**Skill should cover:**
- What OpenRouter provides (400+ models, one key)
- How to pick a model (cost vs capability tradeoffs)
- SDK example for a basic chat completion call
- How to switch models without changing integration code
- Cost awareness — this is the highest-spend capability
- The references file should have a curated table of notable models
  with their strengths (not all 400, just the ones worth knowing)

---

### TICKET 4 — sapiom-images
**Type:** New skill  
**Estimated time:** 30 min

**Docs to paste into chat first:**
- `docs.sapiom.ai/capabilities/images/`

**Files to create:**
- `skills/sapiom-images/SKILL.md`

**Skill should cover:**
- FLUX vs SDXL — when to use each
- Prompt engineering tips specific to each model
- SDK code example
- How to handle and store the returned image (base64 vs URL)
- Common failure modes (content policy, resolution limits)

---

### TICKET 5 — sapiom-audio
**Type:** New skill  
**Estimated time:** 30 min

**Docs to paste into chat first:**
- `docs.sapiom.ai/capabilities/audio/`

**Files to create:**
- `skills/sapiom-audio/SKILL.md`

**Skill should cover:**
- Three sub-capabilities: TTS, transcription, sound effects
- When to use each and what they're backed by
- SDK code example for each sub-capability
- Output formats and how to stream vs buffer audio
- Transcription: supported languages and file format limits

---

### TICKET 6 — sapiom-browser
**Type:** New skill  
**Estimated time:** 60 min

**Docs to paste into chat first:**
- `docs.sapiom.ai/capabilities/browser/`
- `docs.sapiom.ai/reference/sdk/fetch/`

**Files to create:**
- `skills/sapiom-browser/SKILL.md`
- `skills/sapiom-browser/references/browser-api.md`

**Skill should cover:**
- Three modes: scraping, screenshots, AI browser tasks
- When to use browser automation vs web search
- SDK code example for each mode
- How to handle dynamic/JS-heavy pages
- Cost — browser tasks are the most expensive capability
- The references file should cover the full task/action API in detail
  since this is the most complex capability

---

### TICKET 7 — Update README
**Type:** Housekeeping  
**Estimated time:** 15 min  
**No docs paste needed**

**Files to update:**
- `README.md`

**Tasks:**
- Add all 6 new skills to the Available Skills table with descriptions
- Match the voice and style of the existing entry
- Update the installation example to show skill-specific install

**Done when:** The README accurately reflects all 7 skills (existing + 6 new).

---

### TICKET 8 — PR
**Type:** Submission  
**Estimated time:** 20 min  
**No docs paste needed**

**Tasks:**
- Review all 6 skills for consistency (frontmatter format, code style,
  tone) against the existing `sapiom` skill
- Push branch to your fork
- Open PR against `sapiom/skills` main
- Write a clear PR description explaining what you added and why

**PR description should include:**
- What gap you noticed (no per-capability skills)
- What you built (6 skills covering every Sapiom capability)
- How to install and use them
- A note that you're open to feedback on structure/depth

---

## Docs Paste Reference

Quick lookup for which pages to paste per ticket:

| Ticket | Pages needed |
|--------|-------------|
| 1 — web-search | `quick-start/` · `capabilities/search/` · `reference/sdk/fetch/` |
| 2 — verify | `capabilities/verify/` · `reference/sdk/fetch/` |
| 3 — ai-models | `capabilities/ai-models/` · `reference/sdk/fetch/` |
| 4 — images | `capabilities/images/` |
| 5 — audio | `capabilities/audio/` |
| 6 — browser | `capabilities/browser/` · `reference/sdk/fetch/` |
| 7 — README | none |
| 8 — PR | none |

**Note:** `quick-start/` and `reference/sdk/fetch/` only need to be pasted
once — they inform all skills so paste them at the start of Ticket 1 and
they'll be in context for the rest of the session.