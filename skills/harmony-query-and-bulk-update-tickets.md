---
name: Query and bulk-update tickets
description: >-
  Find Harmony tickets with filters and apply the same change to many at once. Grounded in the
  public Harmony Service Desk OpenAPI.
api: openapi/harmony-service-desk-openapi.json
operations:
- queryTickets
- bulkUpdateTickets
- getTicketCustomFields
- setTicketCustomFields
---

# Query and bulk-update tickets

Use this skill to locate a set of tickets and update them together — for example, reassigning or
closing all tickets matching a filter.

## Prerequisites
- AccessKey token sent as `Authorization: AccessKey <token>`.
- Base URL `https://<customer_subdomain>.harmony.io`.

## Steps
1. **Query** — `POST /api/v1/tickets/query` (`queryTickets`). Send a filter body: any of `status`,
   `priority`, `ticket_type`, `assignee`, `reporter`, `desk_id`, `subdesk_id`, `tag_ids`,
   `tag_names`, `source`, `metadata_filters`, plus `page` / `page_size` and `sort_by` /
   `sort_direction`. The response returns `items[]` and `total_count`.
2. **Collect ids** — gather the `id` of each ticket in `items` you intend to change.
3. **Bulk update** — `POST /api/v1/tickets/bulk-update` (`bulkUpdateTickets`) with the list of
   ticket ids and the shared field values. The update is applied as a single atomic transaction.
4. **Custom fields (optional)** — read with `GET /api/v1/tickets/{ticket_id}/custom-fields`
   (`getTicketCustomFields`) and write with `PUT /api/v1/tickets/{ticket_id}/custom-fields`
   (`setTicketCustomFields`) per ticket.

## Conventions & error handling
- Pagination is page-number based (`page`, `page_size`); iterate until you have `total_count` items.
- Bulk update is atomic — a `422` means the whole batch was rejected; inspect `detail[]`, fix, retry.
- See `conventions/harmony-conventions.yml` and `errors/harmony-problem-types.yml`.
