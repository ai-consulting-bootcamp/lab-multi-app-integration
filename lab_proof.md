# LAB | Telegram screams, Sheets remembers

**Week:** 5 · **Day:** 1  
**Workflow:** Telegram → Edit Fields (Set) → Google Sheets (Append Row)  
**Status:** Tested end to end on 3 November 2026.

## Scenario and purpose

A small support or operations team receives messages in Telegram but needs a searchable log outside the chat. This workflow records each received Telegram message in a Google Sheet with a timestamp, sender, chat identifier, message text, and initial status. The record can be reviewed without manually copying messages and gives the team a lightweight audit trail. A bot receives the event; the Edit Fields node normalizes the payload; the Google Sheets node appends a row.

**Technical behavior:** When the Telegram bot receives a message, n8n maps its message fields into a flat record and appends that record to a Google Sheet.

## Workflow and files

| Evidence file | Purpose |
| --- | --- |
| `telegram_to_sheets_workflow.json` | Importable three-node n8n workflow, sanitized for sharing. |
| `source_payload_redacted.json` | Representative Telegram `/start` trigger payload, with private identifiers replaced. |
| `execution_success.png` | n8n canvas with green success marks and one item through each of the three nodes. |
| `google_sheets_result_redacted.png` | Google Sheets screenshot proving appended `/start` records in the expected columns. Personal identifiers have been masked. |

### Trigger → transform → destination

1. **Telegram Trigger:** listens for the Telegram `message` update and emits the nested `message` object.
2. **Edit Fields (Set):** extracts exactly five columns from the message or execution context.
3. **Google Sheets — Append Row:** uses automatic field-to-column mapping and appends a record to worksheet ID `0`.

## Field mapping

| Destination column | n8n expression or fixed value | Why |
| --- | --- | --- |
| `Timestamp` | `{{ $now.toFormat('yyyy-MM-dd HH:mm:ss') }}` | Time the workflow runs, as a consistent text timestamp. |
| `Sender` | `{{ $json.message.from.first_name }}` | Human-readable sender name from Telegram payload. |
| `Chat ID` | `{{ String($json.message.chat.id) }}` | Preserves the chat identifier as a string. |
| `Message text` | `{{ $json.message.text }}` | Copies the message body. |
| `Status` | `New` | Marks each incoming request for later review. |

The Google Sheet has the headers `Timestamp`, `Sender`, `Chat ID`, `Message text`, and `Status`. The input field names match these headers for automatic mapping.

## Execution and destination proof

- A `/start` message was recorded as the initial source event; its shape is represented in `source_payload_redacted.json`.
- The supplied execution screenshot shows **one item** through Telegram Trigger, Edit Fields, and Append row in sheet, with a successful status for every node.
- The supplied Google Sheets screenshot shows **three appended `/start` rows** and the five destination headers. Names and chat IDs were obscured in the shareable image, while the message text and `New` status remain visible.
- After removing pinned trigger data, a separate live Telegram test message (`Week 5 Day 1 - Final automation test`) was **reported by the operator as successful**. This live test is not separately visible in the supplied destination screenshot, so its exact row is not claimed as screenshot-verified.

**Result:** The available trace and destination screenshot together establish that a Telegram event was transformed and written to a Google Sheet. The additional live test is operator-confirmed.

## Verification checks

1. Compare the Telegram event's `message.text` to the new Google Sheets `Message text` entry, and check that the sender and chat identifier map to the same message.
2. Inspect an execution with one item passing each node and verify a new row was appended to the destination sheet (rather than only being generated inside n8n).

## Operational risk and next improvement

**Risk — missing text or schema change:** Telegram can deliver messages without a `message.text` field (for example, a photo). In that situation, the current mapping may create a blank message cell. This can be detected by inspecting executions with empty `Message text` and by filtering for missing fields.

**Suggested improvement (not implemented):** Add an IF node before Edit Fields that allows only nonempty `message.text`; optionally use `update_id` as a deduplication key to reduce duplicate records during retries or repeated tests.

## Submission / portability notes

- The workflow JSON has been scrubbed of pinned Telegram events, credential references, instance identifiers, and the real spreadsheet URL. **After importing, reselect your Telegram and Google Sheets credentials and replace `REPLACE_WITH_YOUR_SPREADSHEET_ID` with the target spreadsheet URL or ID.** The live workflow in the author's n8n instance was not altered.
- No API tokens, OAuth secrets, private Telegram IDs, or personal Google spreadsheet identifiers are included in the submission files.
- Test execution is visible in screenshots; this export does not itself contain an execution log.
