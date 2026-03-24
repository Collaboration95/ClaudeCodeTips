# Service Context: [SERVICE_NAME]

> **Last updated:** [DATE]
> **Last update reason:** [e.g. "New integration: Datadog added"]
> Auto-generated sections: §1–§7. Do not edit manually.
> Human-maintained sections: §8 Architectural Decisions, §9 Integration Checklist.

---

## §1 One-liner
What this service does in one sentence.

---

## §2 Trigger & Entry Point
- **Trigger type:** SQS / HTTP / Cron / Event
- **Triggered by:** [upstream service or event source]
- **Entry handler:** src/handlers/...

---

## §3 Data Flow

1. Receives [X] from [source]
2. Authenticates via [method]
3. Fetches [Y] from [service/API]
4. Transforms / processes
5. Writes to [S3 / DB / queue]
6. Sends completion notification to [SQS / SNS]

ASCII Diagram:
  [SQS: incident-queue]
          ↓
    [this-service]
     ├─ queries [DataSource A]
     ├─ queries [DataSource B]
     └─ writes → [S3: bucket/{incident_id}/]
          ↓
    [SQS: completion-queue]
     ├─ [consumer-service-A reads]
     └─ [consumer-service-B reads]

---

## §4 Integrations & External Dependencies

| System | Direction | Auth Method | Notes |
|--------|-----------|-------------|-------|
|        |           |             |       |

---

## §5 Environment Variables

  ENV_VAR_ONE=
  ENV_VAR_TWO=

---

## §6 Output Schema (S3 / Event payload)

  {
    "incident_id": "string",
    "source": "...",
    "fetched_at": "ISO8601",
    "raw_data": {},
    "metadata": {}
  }

---

## §7 Downstream Consumers

| Service | Reads From | Expects |
|---------|------------|---------|
|         |            |         |

---

## §8 Key Architectural Decisions
<!-- HUMAN-MAINTAINED — why things are the way they are.
     Prevents AI from suggesting changes that conflict with intentional design. -->

- **[Decision title]:** [Reason]

---

## §9 Adding a New Integration (Checklist)
<!-- HUMAN-MAINTAINED — the repeatable pattern for this specific repo -->

- [ ] Add handler in src/integrations/[name]/
- [ ] Register in src/registry.ts
- [ ] Add env vars to .env.example and §5 above
- [ ] Update §6 if output schema changes
- [ ] Update §7 if new downstream consumers are affected
- [ ] Add a PAST_LEARNINGS entry after completion
