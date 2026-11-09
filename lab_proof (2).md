# Lab Proof: Telegram screams, Sheets remembers

## 1. Scenario

A user interacts with a Telegram bot to submit messages that need to be
recorded and tracked. Each incoming message is automatically captured
and stored in a Google Sheet, including the sender's details, chat ID,
message content, and timestamp. A Google Sheet provides a structured,
centralized, and searchable record, making it easier to review and track
messages than relying on chat history or an email inbox.

## 2. Workflow

Telegram Trigger (On message) → Edit Fields (Set) → Google Sheets
(Append Row)

-   Export: `workflow.json`
-   Workflow canvas with all nodes green: `canvas.png`

## 3. Source payload

`trigger_output.png`

Fields used: - `message.date` - `message.from.username` (fallback:
`first_name`) - `message.chat.id` - `message.text`

User and chat IDs are personal and were redacted.

## 4. Mapping

  ----------------------------------------------------------------------------------------------------------
  Sheet column                        Expression
  ----------------------------------- ----------------------------------------------------------------------
  Timestamp                           `{{ new Date($json.message.date * 1000).toISOString() }}`

  Sender                              `{{ $json.message.from.username || $json.message.from.first_name }}`

  Chat                                `{{ String($json.message.chat.id) }}`

  Message Text                        `{{ $json.message.text }}`

  Status                              `New` (literal default)
  ----------------------------------------------------------------------------------------------------------

**Transformation evidence:** `set_node.png`

The timestamp is converted from Telegram's Unix timestamp in seconds to
an ISO 8601 date-time string. The sender mapping uses the username when
available and falls back to the first name. The chat ID is converted to
a string, and Status is assigned the literal value `New`.

## 5. Destination record

`sheet_row.png` shows the row containing the message "One more time",
with all five columns populated.

## 6. Verification

Execution evidence: - `canvas.png` - `set_node_io.png` -
`sheets_node_io.png`

Each node completed successfully and processed one item.

## 7. Risk

**Most likely to break:** The `message.text` field may be missing when
users send non-text content, such as photos, stickers, videos, or voice
messages, because Telegram uses different payload fields for these
message types.

**Detection:** An empty Message Text cell in the Google Sheet indicates
that the incoming Telegram message did not contain a text field and
requires additional handling.

**Duplicate protection:** Repeated test executions currently create
duplicate rows because the workflow does not yet implement duplicate
detection or idempotency controls.
