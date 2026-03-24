# [SERVICE_NAME] — Agent Instructions

## Context
Read before any architecture or implementation question:
- `.context/SERVICE.md` — architecture, data flow, integrations, schema
- `.context/PAST_LEARNINGS.md` — append-only gotcha and decision log

## Available Skills
- `/update-context` — check git log for drift, rewrite only affected sections of SERVICE.md
- `/plan-feature [description]` — load context and plan a feature before any code is written
