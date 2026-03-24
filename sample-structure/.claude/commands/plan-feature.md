# /plan-feature [description]

You are a senior engineer helping plan a feature or integration for this service.
Before any planning, load context and surface anything that could affect the approach.

---

## Step 1 — Load context

Read in full:
1. .context/SERVICE.md
2. .context/PAST_LEARNINGS.md

Confirm with:
  "Loaded context for [SERVICE_NAME]. Last updated [DATE]."

---

## Step 2 — Scan PAST_LEARNINGS for relevance

Match the tags in PAST_LEARNINGS entries against the feature description:

| Feature type                  | Tags to match                    |
|-------------------------------|----------------------------------|
| New integration               | [integration] [auth] [schema]    |
| Queue / async work            | [queue] [performance]            |
| Auth / token handling         | [auth]                           |
| Output or data shape changes  | [schema] [storage]               |
| Infrastructure changes        | [infra]                          |

If relevant entries found, present a ⚠️ Watch Out block first, before any planning:

  ⚠️ RELEVANT PAST LEARNINGS

  1. [DATE] — [Title]
     → Why relevant: [reason tied to this feature]
     → Key takeaway: [one line]

If nothing relevant: state "No past learnings flagged for this feature."

---

## Step 3 — Architecture exploration (no code)

### Overview
What is the feature? What problem does it solve?

### Where it fits in the existing flow
Map it to the current data flow from SERVICE.md. Where does it slot in?

### Architecture additions needed
- New components or files (following the pattern in §9 of SERVICE.md)
- New env vars required
- Schema changes to §6 if applicable
- Downstream consumer impact per §7

### Data available
If this involves a new external tool or API, use web fetch to look up their docs:
- What data/events they expose
- Auth method
- Rate limits or constraints worth knowing

### Open questions / risks
Decisions that need to be made before implementation starts.

---

## Guardrails

- Do NOT write implementation code unless explicitly asked after this planning phase
- Do NOT suggest changing queue names, storage paths, or completion notification
  format without flagging downstream dependencies (§7 of SERVICE.md)
- Always check §8 (Architectural Decisions) before suggesting structural changes
- If your plan conflicts with an existing architectural decision, flag it explicitly
