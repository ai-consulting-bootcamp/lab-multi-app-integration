# Telegram-to-Airtable Workflow Proof

## 1. Scenario

The support team receives client orders through a Telegram bot and needs a
shared record of each order. The workflow captures the message and sender
details, then an IF condition checks whether the message contains `/order`.
Matching messages are recorded in the team's Airtable orders table, while
non-matching messages receive a prompt to send an `/order` command. This gives
the team a searchable handoff and reduces the chance that an order is missed
in the chat history.

**Technical behavior:** When the Telegram Trigger receives a message, the
workflow normalizes its fields; if the message contains `/order`, it creates an
Airtable record and sends the client a confirmation with their name and order;
otherwise it sends a prompt to start an order.

## 2. Workflow export

Sanitized from original n8n JSON export:
[Download the sanitized workflow export](./workflow_export_sanitized.json).

Credential references, tokens, workflow/node/property IDs, webhook identifiers,
Telegram chat IDs, and Airtable base/table identifiers have been removed or
redacted.

## 3. One source payload

Sanitized Telegram Trigger output from the execution that created the Airtable
record in section 4. Personal names and Telegram user/chat IDs are redacted; the
message is retained to show the source-to-destination match.

```json
{
  "message": {
    "message_id": 56,
    "from": {
      "id": "[REDACTED_TELEGRAM_USER_ID]",
      "is_bot": false,
      "first_name": "[REDACTED]",
      "last_name": "[REDACTED]",
      "language_code": "de"
    },
    "chat": {
      "id": "[REDACTED_TELEGRAM_CHAT_ID]",
      "first_name": "[REDACTED]",
      "last_name": "[REDACTED]",
      "type": "private"
    },
    "date": 1793805059,
    "text": "/order 2 cakes",
    "entities": [
      {
        "offset": 0,
        "length": 6,
        "type": "bot_command"
      }
    ]
  }
}
```

## 4. One destination record

Sanitized Airtable **Create a record** output from the same execution. The
Airtable record ID and personal/chat identifiers are redacted. The message and
timestamp correspond to the Telegram payload above.

```json
[
  {
    "id": "[REDACTED_AIRTABLE_RECORD_ID]",
    "createdTime": "2026-11-04T15:11:00.000Z",
    "fields": {
      "Chat ID": "[REDACTED_TELEGRAM_CHAT_ID]",
      "Name": "[REDACTED]",
      "Timestamp": "2026-11-04T15:10:59.000Z",
      "Message": "/order 2 cakes"
    }
  }
]
```

## 5. Mapping table

These mappings match the sanitized workflow's `Edit Fields` and Airtable node
configuration.

| Source field (Telegram) | Normalized field | Destination field (Airtable) | Mapping / reason |
|---|---|---|---|
| `message.from.first_name` | `Name` | `Name` | Captures the sender's first name for request follow-up. |
| `message.from.username` | `Username` | `Username` | Captures the Telegram username; defaults to an empty string when absent. |
| `message.text` | `Message` | `Message` | Stores the message text, including the `/order` request. |
| `message.chat.id` | `Chat ID` | `Chat ID` | Converted to a string to fit the configured Airtable field. |
| `message.date` | `Timestamp` | `Timestamp` | Converted from Unix seconds to an ISO timestamp. |

## 6. Likely break point and detection

The `Message` field is most likely to need attention if the Telegram payload
changes, because the `Edit Fields` node reads it from `$json.message.text`.
The IF node checks whether that mapped `Message` contains `/order`: matching
items go to Airtable and then to the order confirmation, while non-matching
items go to the prompt asking the client to send an `/order` message. Detect a
mapping or routing problem by inspecting the Telegram Trigger output, the
`Edit Fields` value for `Message`, and which IF output branch the execution
takes. For a matching order, verify the Airtable record and confirmation use
the expected message and client name.
