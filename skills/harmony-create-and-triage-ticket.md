---
name: Create and triage an IT ticket
description: >-
  Create a Harmony service-desk ticket via the API, then set its priority, status, and assignee to
  triage it. Grounded in the public Harmony Service Desk OpenAPI.
api: openapi/harmony-service-desk-openapi.json
operations:
- createTicket
- getTicketByID
- updateTicket
- getTicketActivity
---

# Create and triage an IT ticket

Use this skill to open a new IT ticket in Harmony and route it to the right team.

## Prerequisites
- An AccessKey API token (Harmony dashboard → Settings → API Keys → Generate New API Key; shown once).
- Your organization's API base URL: `https://<customer_subdomain>.harmony.io`.
- Send `Authorization: AccessKey <token>` on every request.

## Steps
1. **Create the ticket** — `POST /api/v1/tickets/` (`createTicket`). Provide `title`, `description`,
   `ticket_type` (`request` | `incident` | `question`), `priority` (`low` | `medium` | `high` |
   `urgent`), `reporter`, and optionally `desk_id`. The response returns the new ticket `id` and a
   `201` status. Validation problems come back as `422` with a `detail[]` list — fix the flagged
   fields and retry.
2. **Confirm it landed** — `GET /api/v1/tickets/{ticket_id}` (`getTicketByID`) to read back the
   created ticket and its assigned `status`, `desk_id`, and `thread_id`.
3. **Triage** — `PATCH /api/v1/tickets/{ticket_id}` (`updateTicket`) to set `assignee`, adjust
   `priority`, or move `status` (e.g. to `in_progress`).
4. **Audit** — `GET /api/v1/tickets/{ticket_id}/activity` (`getTicketActivity`) to read the
   activity-log events recording the changes.

## Conventions & error handling
- No idempotency key is supported — do not blindly retry `createTicket` on a network error without
  first checking whether the ticket exists (list/query by title).
- Errors use a `{"detail": ...}` envelope (FastAPI-style, not RFC 9457). See
  `errors/harmony-problem-types.yml`.
- Respect rate limits: 100 requests/minute and 1,000/hour; watch `X-RateLimit-Remaining`.
