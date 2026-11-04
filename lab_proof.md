# Lab Proof — Telegram to Airtable

**Student:** Roodabeh Seif  
**Date:** 4 November 2026

## 1. Business scenario

A small bakery receives customer orders through a Telegram bot. The workflow captures every text message and saves the customer's details, message, chat ID, and timestamp as a structured Airtable record so the team does not lose incoming requests.

## 2. Workflow

`Telegram Trigger` → `Edit Fields` → `Airtable: Create a record`

- **Telegram Trigger:** starts the workflow whenever the bot receives a message.
- **Edit Fields:** extracts and renames the five values needed by the business.
- **Airtable:** creates one new record in the `Messages` table of the `Order Tracker` base.

The sanitized workflow export is included as `telegram-airtable-workflow.json`. Credentials and tokens are not included.

## 3. Telegram payload evidence

The workflow received this real test message. Personal identifiers are redacted in this proof.

```json
{
  "message": {
    "from": {
      "first_name": "Rood",
      "username": "<redacted_username>"
    },
    "chat": {
      "id": "<redacted_chat_id>"
    },
    "date": 1793806978,
    "text": "I want to order a chocolate cake for Friday."
  }
}
```

## 4. Field mapping

| Airtable field | Telegram source / n8n expression | Purpose |
|---|---|---|
| `Name` | `{{ $json.message.from.first_name }}` | Customer's first name |
| `Username` | `{{ $json.message.from.username \|\| '' }}` | Telegram username, or an empty string when unavailable |
| `Message` | `{{ $json.message.text }}` | Customer's order message |
| `Chat ID` | `{{ String($json.message.chat.id) }}` | Telegram conversation identifier stored as text |
| `Timestamp` | `{{ DateTime.fromSeconds($json.message.date).toISO() }}` | Converts Telegram Unix time into an ISO date-time value |

## 5. Airtable record evidence

The test created a new record in `Order Tracker` → `Messages`:

| Field | Stored value |
|---|---|
| `Name` | `Rood` |
| `Username` | `<redacted_username>` |
| `Message` | `I want to order a chocolate cake for Friday.` |
| `Chat ID` | `<redacted_chat_id>` |
| `Timestamp` | `2026-11-04T16:42:58+01:00` |

A second end-to-end production test with the message `I need two birthday cakes for Saturday.` also created a new Airtable row automatically after the workflow was published.

## 6. Brittle field and failure risk

The most brittle field is `Message`, mapped from `message.text`. Telegram messages do not always contain text; a photo, sticker, voice note, or attachment may omit this property. In that case, the expression can return `undefined`, producing an empty Airtable value or a failed execution if the destination field becomes required.

A production version should either restrict the trigger to text messages or add a validation step before Airtable. For example, an `If` node can check whether `message.text` exists and route unsupported message types to a separate path.

The `Username` field is also optional in Telegram. The fallback expression `{{ $json.message.from.username || '' }}` prevents a missing username from breaking the workflow.

## 7. Validation result

- Telegram bot connection: successful
- Message received by n8n: successful
- Five fields extracted: successful
- Airtable record created: successful
- Published end-to-end test: successful
- Secrets committed to the export: none
