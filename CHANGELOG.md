# Changelog

All notable changes to this Freshdesk connector are documented here. Release
versions track the `plugin.yaml` version.

## 0.2.1

### Changed

- Tool descriptions and errors no longer name the host app ("Requires human approval before execution."), and the User-Agent is `Freshdesk-Connector/0.2.1`.
- Reply and private-note approval cards show the first 250 characters of the text (was 100), still within the 450-character card cap.
- `freshdesk_download_attachment`: downloading the same bytes again returns the existing file instead of an error; different bytes under the same name still fail. Each download removes files older than a week from `cache/freshdesk` (regular files only; symlinks and directories are left alone).

## 0.2.0

### Added

- `freshdesk_reply_to_ticket`: public reply to the requester. The body is HTML-escaped, then verified through the conversation list.
- `freshdesk_add_time_entry`: a finished, non-timer time entry. Callers may pass `H:MM` through `HHHH:MM`. The client stores the `HH:MM` form Freshdesk keeps (`001:15` is stored as `01:15`). Zero is rejected, including padded zeros, before any HTTP call. Verification uses `GET /api/v2/tickets/{ticket_id}/time_entries`, not `GET /api/v2/time_entries/{id}`.
- `freshdesk_update_ticket`: update a ticket's `status` (2-5) and/or `priority` (1-4), then read it back.
- Search filters beyond `status` and `priority`: `tag`, `type`, `agent_id`, `group_id`, `created_after`, `created_before`, `updated_after`, and `updated_before`. Still no raw query string. Date bounds are exclusive (`created_at:>'date'`).

### Changed

- `plugin.yaml` version is `0.2.0`.
- Five write tools now pass the human-approval gate: private note, create, reply, time entry, and update.
- The approval Reason names the affected ticket and the proposed change it will write: note or reply text, create-ticket requester plus `status`/`priority` labels, a time-entry duration and billable flag, and an update from-to `status`/`priority` with labels. A newline in a Reason field is collapsed, so it cannot hide the change. `rule_key` is the tool name plus a hash of the approval-relevant arguments, so `Always` cannot blanket-approve every later write of a given tool.
- Ticket descriptions are HTML-escaped before they are sent.
- Install is the catalog command (`hermes plugins install freshdesk`), not a manual directory copy.

### Known limitation

- Freshdesk collapses a newline to a space in `description_text` and conversation `body_text`. A create or note whose body contains a newline can land, then fail readback with `state=write_may_have_succeeded` and `retry_safe=false`. Do not retry that write.
